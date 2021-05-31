---
layout: post
title: Create a directory
thumbnail-img: /assets/img/sas.png
tags: [sas,create,directory]
---

This macro function allows to create a directory from SAS. I use it very often, mainly to create output folder with the current date, to classify my outputs.

## Description
Macro function for creating a directory.  
It checks if the directory to create already exists, then ensures the parent directory is valid and eventuelly create the new directory.  
  
This macro is OS agnostic and uses only native SAS functions.

## Sample
```
%create_dir(/home/user/test);
```

The code above creates a directory `test` inside `/home/user`.

## Code
```
%macro create_dir(dir);
    *-- Define path delimiter based on the OS type --*;
    %let _sep = %sysfunc(ifc("&sysscp."="WIN", %str(\), %str(/)));

    *-- Remove trailing path delimiter if any --*;
    %if (%index(%sysfunc(reverse(&dir.)), &_sep.) = 1) %then %do;
        %let dir = %substr(&dir., %length(&dir.) - 1);
    %end;

    *-- New directory must no exist --*;
    %if %sysfunc(fileexist("&dir.")) %then %do;
        %put %str(NOTE: directory &dir. already exists);
        %return;
    %end;
    
    *-- Extract the parent directory and the new folder to create within it --*;
    %let _root = %sysfunc(reverse(%substr(%sysfunc(reverse(&dir.)), %index(%sysfunc(reverse(&dir.)), &_sep.))));
    %let _fold =  %sysfunc(reverse(%substr(%sysfunc(reverse(&dir.)), 1, %index(%sysfunc(reverse(&dir.)), &_sep.) - 1)));

    *-- Parent directory must exist --*;
    %if not %sysfunc(fileexist("&_root.")) %then %do;
        %put %str(ERROR: &_root. directory is not valid);
        %let syscc = 99;
        %return;
    %end;

    *-- Create new directory --*;
    %let rc = %sysfunc(dcreate(&_fold., &_root.));

    *-- Ensure everything is fine now--*;
    %if not %sysfunc(fileexist("&dir.")) %then %do;
        %put %str(ERROR: not able to create new directory &_fold. within &_root.);
        %let syscc = 99;
        %return;
    %end;

    %put %str(NOTE: directory &dir. created successfully);
%mend;
```
