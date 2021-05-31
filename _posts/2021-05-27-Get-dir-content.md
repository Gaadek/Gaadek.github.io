
---
layout: post
title: Create a directory
thumbnail-img: /assets/img/sas.png
tags: [sas],[directory]
---

## Purpose
Get (or list) the content of a directory.  
It checks if the directory to create already exists, then ensures the parent directory is valid and eventuelly create the new directory.

## Sample
```
%get_dir_content(new, /home/user/new)
```
The code above list the content of "/home/user/new" directory (either file or directory) and report as a dataset "new".

## Code
```
*-- Macro to list the content of a directory --*;
%macro get_dir_content(ds, dir);
    data &ds. (keep = memname memtype);
        attrib memname format=$300.;
        attrib memtype format=$10.;
        
        *-- Associate 'filrf' file reference with the directory to process --*;
        %if %sysfunc(filename(filrf, &dir.)) = 0 %then %do;
            *-- Open the file reference as directory --*;
            %let did = %sysfunc(dopen(&filrf));
            
            %if &did. ne 0 %then %do;
                *-- Define path delimiter based on the OS type --*;
                %let _sep = %sysfunc(ifc("&sysscp."="WIN", %str(\), %str(/)));
    
                *-- Loop through the directory members --*;
                %do i=1 %to %sysfunc(dnum(&did));
                    %let memname = %qsysfunc(dread(&did., &i.));
                    
                    *-- Determine if current processed directory member is a file or a directory --*;
                    %if %sysfunc(filename(mrf, &dir.&_sep.&memname.)) = 0 %then %do;
                        *-- Open the member reference as directory --*;
                        %let mid = %sysfunc(dopen(&mrf));
                        
                        %if &mid. ne 0 %then %do;
                            %let memtype = directory;
                            %let rc = %sysfunc(dclose(&mid.));
                        %end;
                        %else %let memtype = file;
                    %end;
                    
                    *-- Output data in the dataset --*;
                    memname = strip("&memname.");
                    memtype = strip("&memtype.");
                    output;
                %end;
                
                *-- Force the stop. In case of empty directory, allows a 0 observation output dataset --*;
                stop;
                
                *-- Close the directory file reference --*;
                %let rc = %sysfunc(dclose(&did.));
            %end;
            %else %do;
                %put ERROR: &dir cannot be opened.;
                
                *-- Force the stop to get a 0 observation output dataset --*;
                stop;
            %end;
        %end;
        
    run;
%mend get_dir_content;
```
