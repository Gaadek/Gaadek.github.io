---
layout: post
title: Using a SAS dataset for macro loop
thumbnail-img: /assets/img/sas.png
tags: [sas,loop,macro,fetch,open,close,cursor]
---

This topic explains how to use a dataset to define a macro loop. OK, I need to clarify what this means. A macro loop is basically a set of instructions (macro function, macro code, SAS code...) to execute a certain number of times.  
If we know by advance how many time we have too loop, that's perfect, you don't need to go further. But if you know only at runtime the list of items to process, then it's a good idea to store 
these iterms in a dataset, then to loop over this dataset to call a macro. Typic example is: you build a list of subjects, then you generate one report per patient.

# Solution #1: cursor/fetch.
This method is the one I usualy implement because it's clear, easy to understand/review and **efficient**.  
Its drawbacks are it's a lot of code to perform a simple loop and, in case of crash in the middle of the loop, the handler opened to read the dataset remains open and you'll have to restart your SAS session.
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
