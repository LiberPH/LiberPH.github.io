---
layout: post
title: Spark's big bug
subtitle: 20530 "Cannot evaluate expression" when filtering on parquet partition column 
tags: [spark,bugs,big data]
---

Pain! I've lost almost one week of work (ish, because I've already finished all the stuff I needed to do, the thing is I'm going from samples to work with the real data).
The thing is, tha partition column of the table that I'm working with is a date, dates usually have an extra problem in pySpark,
because python doesn't like the spark's date format and gets confused with the months of both. Nevertheless, if you want to compare 
the full date, everything seems to work fine. For example, in my case I want to use only the latest date from the table. So, if I get 
the latest date in the following way:
```python
dates = [i.data_date for i in ifrs9_df.select("date_column").distinct().collect()]
dates.sort()
date = dates[-1]
```
And then proceed to filter my df by that date like this:
```python
date_cond = c("date_column") == date
ifrs9_df_filtered = df.filter(date_cond)
```
Everything works and we are happy forever. Unless, you have more than one partition by month. In my case, I have two partitions for March, 
two for June and one for every other month. So if I try to filter my data like this:
```python
date_cond = (F.year(c("date_column")) == date.timetuple().tm_year) & (F.month(c("date_column")) == date.month)
ifrs9_df_filtered = df.filter(date_cond)
```
As soon as I proceed to apply an action like *count* or check if the DF *is empty* I get an error message. Thi seems to be happening due to an incompatibility between spark and pyhton's date formats. It seems like this, because, something pretty weird is that this only happens when comparing, if you group data by month and year like this: 
```python
count_date_df = df.groupBy(
            F.year(c("date_column")), F.month(c("date_column"))).count().orderBy(
        F.year(c("date_column")),F.month(c("date_column")))
```
There is not any problem at all O.o.

So, given Spark and python incompatibilities the "best" way to go is to use a UDF to parse both dates to be compared to strings. I'm going to omit my udf and only show how I declared it like an UDF:
```python
compare_date_udf = F.udf(lambda a,b: compare_date(a,b), IntegerType())
```
Now, if I define my latest date as previously but casting it to string, make a new column with my date casted as well and pass both to the UDF, everything seems fine until I call an RDD dependent action such as df.count():

```python
ifrs9_curr_df_a = df.select('*',
                                c("date_column").cast(StringType()).alias("date_str")
                                                          )
ifrs9_curr_df = ifrs9_curr_df_a.filter( compare_date_udf("date_str",F.lit(date)) == 1)
```

**The problem:** It is a pain in the #$$ to filter the column used for parquet partition

**Why?:** [There is a major bug in Spark] (https://issues.apache.org/jira/browse/SPARK-20530)

**How to solve it?**

