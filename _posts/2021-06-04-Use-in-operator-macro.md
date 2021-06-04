---
layout: post
title: Using the `in` operator in macro functions
thumbnail-img: /assets/img/sas.png
tags: [sas,snippet,in,operator, macro]
---

Here is a small snippet to explain how to use the Methods to enable the use of the `in` operator in macro functions.  

## Sample
```
    ...
    *-- First step: enable the IN operator in macros, using the comma as list delimiter --*;
    options minoperator mindelimiter=',';
    
    %let value=value2;
    
    *-- Second step: after 1st step, the `in` operator is available --*;
    %if &value. in (value1, value2, value3) %then %do;
        %put The value &value. is in the list!;
    %end;
    %else %do;
        %put The value &value. is not in the list.;
    end;
    ...
```
