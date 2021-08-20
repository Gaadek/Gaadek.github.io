---
layout: post
title: Retrieve the current date & time in a macro variable
thumbnail-img: /assets/img/sas.png
tags: [sas,snippet,date,time,macro,yyyymmdd]
---

Today, a snippet to retrieve the current date & time in a macro variable.  I've seen a lot of unnecessary complex codes where people use a data step with a call to `symput`, e.g.:
```
/*Get today's date*/
data _null_;
call symput('today_date', put(input("&sysdate", date9.), yymmdd10.));
run;
```

This works, but honestly, we can replace that by a more elegant code!

## Sample
```
    ...
    *-- version to extract the current date in YYYYMMDD format --*;
    %let yyyymmdd	= %sysfunc(date(), yymmddn8.);
    
    *-- version to extract the current date & time in YYYYMMDD_HHMI format --*;
    %let yyyymmdd_hhmi = %sysfunc(date(), yymmddn8.)_%sysfunc(compress(%sysfunc(time(), tod5.), , dk));
    
    *-- version to extract the current date & time in YYYYMMDD_HHMISS format --*;
    %let yyyymmdd_hhmiss = %sysfunc(date(), yymmddn8.)_%sysfunc(compress(%sysfunc(time(), tod8.), , dk));
    ...
```
