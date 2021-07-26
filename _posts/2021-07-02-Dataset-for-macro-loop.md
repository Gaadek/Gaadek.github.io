---
layout: post
title: Using a SAS dataset for macro loop
thumbnail-img: /assets/img/sas.png
tags: [sas,loop,macro,fetch,open,close,cursor]
---

This topic explains how to use a SAS dataset as the source definition of a loop. This is very useful when you collect data at run time and then want to use these data to perform the same repeated task (e.g. build a list of items, then generate a report for each item)

# Solution #1: cursor/fetch.
This method is the one I usually implement because it's clear, easy to understand/review and **efficient**.  
Its drawbacks are it's a lot of code to perform a simple loop and, in case of code crash in the middle of the loop, the handler used to read the dataset remains open and you cannot close it. So if you run a second time the code, SAS will notify you that the SAS dataset is already in use. You'll have no other choice than restarting your SAS session.
```
    %macro loop;
        ...
        data input_dataset;
            *-- This is out input dataset. It's probably build outside the macro --*;
            attrib c_var format=$30.;
            attrib n_var format=8.;
            ...
        run;
        
        *-- Open the input dataset --*;
	%let hpcurvgl = %sysfunc(open(input_dataset));
        
        *-- Ensure the dataset has been successfully opened --*;
        %if &hpcurvgl. %then %do;
            *-- Fetch the input dataset (i.e. read 1 record at a time) --*;
            %do %while(not %sysfunc(fetch(&hpcurvgl.)));
                *-- Read input dataset variables --*;
                *-- Notice we use getvarC for char vars, getvarN for num var --*;
                %let char_var   = %qsysfunc(getvarc(&hpcurvgl., %qsysfunc(varnum(&hpcurvgl., c_var))));
                %let num_var    = %sysfunc(getvarn(&hpcurvgl., %sysfunc(varnum(&hpcurvgl., n_var))));
                
                *-- Do what you want here --*;
                *-- Call another macro --*;
                *-- Create a new dataset --*;
                *-- Or anything else you want ! --*;
            %end;

            *-- Finally, close the input dataset --*;
            %let hprcvgl=%sysfunc(close(&hpcurvgl));
        %end;
    %mend;
      
    %loop;
```

# Solution #2: macro variable loop.
This method is easier than method #1 because it does not requires to open the SAS dataset. However, it has some limitations, you have to use one variable only or create as many variables as needed.
```
    %macro loop;
        ...
        data input_dataset;
            *-- This is out input dataset. It's probably build outside the macro --*;
            attrib c_var format=$30.;
            attrib n_var format=8.;
            ...
        run;
        
		*-- Get the list of items in a macro variable --*;
		proc sql noprint;
			select		distinct c_var
          	into		:my_list separated by ' '
			from		input_dataset
			;
		quit;

        *-- Loop --*;
		%do i=1 %to %sysfunc(countw(&my_list));
			%let char_var = %scan(&my_list, &i);
			
			*-- Do what you want here --*;
			*-- Call another macro --*;
			*-- Create a new dataset --*;
			*-- Or anything else you want ! --*;
		%end;
    %mend;
      
    %loop;
```

