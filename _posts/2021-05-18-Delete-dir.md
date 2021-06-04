---
layout: post
title: Delete a directory
thumbnail-img: /assets/img/sas.png
tags: [sas,macro,delete,directory]
---

After the macro function which permits to [create directories](2021-05-18-Create_dir), this time I present the macro I use to delete directories. 
This macro is OS agnostic and uses only native SAS functions.  

## Sample
```
%delete_dir(home/user/new)
```
The code above delete the `test` directory from `/home/user`.

## Code
```
%macro delete_dir(dir);
    *-- Define path delimiter based on the OS type --*;
    %let _sep = %sysfunc(ifc("&sysscp."="WIN", %str(\), %str(/)));

    *-- Remove trailing path delimiter if any --*;
    %if (%index(%sysfunc(reverse(&dir.)), &_sep.) = 1) %then %do;
        %let dir = %substr(&dir., %length(&dir.) - 1);
    %end;

    *-- Directory to delete must exist --*;
    %if not %sysfunc(fileexist("&dir.")) %then %do;
        %put %str(NOTE: directory &dir. does not exist);
        %return;
    %end;

    *-- Delete directory --*;
    filename tmp "&dir.";
    %let rc = %sysfunc(fdelete(tmp));
    filename tmp clear;

    *-- Ensure everything is fine now--*;
    %if %sysfunc(fileexist("&dir.")) %then %do;
        %put %str(ERROR: not able to delete directory &dir.);
        %let syscc = 99;
        %return;
    %end;

    %put %str(NOTE: directory &dir. deleted successfully);
%mend;
```
