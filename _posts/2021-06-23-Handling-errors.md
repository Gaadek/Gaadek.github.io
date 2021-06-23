---
layout: post
title: How to control SAS execution flow?
thumbnail-img: /assets/img/sas.png
tags: [sas,handling,error,execution,flow,control,stop,abend,return,cancel]
---

This post is an attempt to determine how to manage SAS code execution flow. Indeed, a major issue of SAS (at least for me) is its lack of errors handling. Let's say you've got 
a script that extracts data, processes data then exports processed data. What if an error occurs at data extraction? Obviously, you would probably stop here and do not waste time in 
processing and output steps. Unfortunately, SAS does not take care at all of that, it will execute the full script whatever happen and by the way output millions or errors.  

## Scenario
To illustrate what we expect and how it works, let's write a small snippet:
```
%put >>> Data extract;
%put >>> Data processing;
%put >>> Data output;
```

It represents 3 steps to be run sequentially. The goal is to check, after each step, if something gone wrong and, if so, stop the execution of the following steps.

## Solution #1: embed your code in a macro.
The first proposal is to embed the code in a single, unique macro:
```
    %macro main;
        ...
        %put >>> Data extract;
        
        *-- Test if something gone wrong, e.g., check syscc --*;
        %if &something %then %do;
            %put Exiting macro;
           
            *-- %return exits the macro without error --*;
            %return;
            
            *-- %abort exits the macro AND output an error --*;
            %abort;
            
            *-- %abort cancel exits the macro AND stop the SAS script execution --*;
            %abort cancel;
        %end;
        
        %put >>> Data processing;
        ...
    %mend;
      
    %main;
      
    %put >>> Something outside the macro;
```

The main topic concerns how we exit the macro:
* **%return** exits the macro without warning/error. It can be useful when you don't want to output any error message and want to execute some steps outside the macro (e.g., to send an email notification)
* **%abort** exits the macro after raising an error, but the code after the macro is executed. It is similar to the **%return** statement, except it raises an erro which personally I appreciate because it highlights an abnormal termination of the SAS program.
* **%abort cancel** exits the macro after raising an error and stop SAS script execution. For me, the best solution if you don't have post processing steps (e.g., log parsing, email notification)


## Solution #2: define a macro to call each time you need to stop the SAS script.
The second proposal takes advantage of the possibility to use `%if %then...%end` code in open code. This proposal avoid the use of a big macro, thus is a little bit easier to implement while being less flexible than solution #1 (we cannot perform post error processing):
```
    %macro stop_sas;
        *-- It is mandatory to call %abort cancel, else this macro is useless --*;
        %abort cancel;
    %mend;
    
    ...
    %put >>> Data extract;

    *-- Test if something gone wrong, e.g., check syscc --*;
    %if &something %then %do;
        %stop_sas;
    %end;

    %put >>> Data processing;
    ...
```

## How to select the good option?
Solution #1 will be the preferred solution when:
1) You have to perform post processing actions, such as log parsing, notification or any other stuff. 
2) You don't want to output an error for aborting the SAS script execution. Then, there is only the `%return` statement that does the job

Solution #2 will be the preferred solution when:
1) You perform your post processing actions outside SAS (e.g., script to run your SAS script then log parsing/notificatio,)
2) You don't care about having an extra error in your log (for me it's a best practice to let this error)



