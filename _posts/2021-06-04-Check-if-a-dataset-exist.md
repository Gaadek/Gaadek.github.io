---
layout: post
title: Check if a dataset exist
thumbnail-img: /assets/img/sas.png
tags: [sas,snippet,check,dataset,exist]
---

This snippet provides a simple method to test if a dataset exists (or not).  

## Sample
```
    ...
    %if not %sysfunc(exist(dataset_name)) %then %do;
        %put The dataset does not exist!;
        %return;
    %end;
    ...
```
