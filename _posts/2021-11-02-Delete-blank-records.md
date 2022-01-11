---
layout: post
title: Snippet to delete blank records from a dataset
thumbnail-img: /assets/img/sas.png
tags: [sas,snippet,delete,remove,blank,missing,record]
---

In this topic I propose a small macro function I'm using to delete blank records from a SAS datasets.   
I use it mainly for the processing of external data, where it's common to have blank records in the input file (CSV or Excel).

1) Solution #1: efficient but not for all cases
```
data ...;
	set ...;

	*-- Remove blank records --*;
	if missing(cats(of _all_)) then delete;
run;
```

The solution above works fine, except it converts missing numeric values into a dto char, thus it works only when the SAS dataset contains only char variables.

2) Generic method
```
*-- Create a SAS statement which test any missing variable, regardless of its type --*;
proc sql noprint;
    select      distinct 'missing(' || strip(name) || ')'
    into        :miss_stmt separated by ' and '
    from        dictionary.columns
    where       libname = 'YYY'
        and     memname = 'XXX'
    ;
quit;

*-- Delete blank records --*;
data yyy.xxx;
    set yyy.xxx;
    
    if &miss_stmt. then delete;
run;
```
