---
layout: post
title: Get current directory
thumbnail-img: /assets/img/sas.png
tags: [sas,macro,current,directory]
---

This macro function allows to get the directory (i.e. location) of the current SAS program.  
Note: the SAS program must have been saved at least once befre using this macro.  

## Sample
```
%%let cur_dir = %get_cur_dir;
```
The code above will retrieve the current SAS program directory and store it  in `cur_dir` macro variable.

## Code
```
*-- Macro to list the content of a directory --*;
%macro get_cur_dir;
    %let rc = %sysfunc(filename(filrf, .));
    %let curdir = %sysfunc(pathname(&filrf.));
    %let rc = %sysfunc(filename(filrf));
    
    &curdir.
%mend get_cur_dir;
```
