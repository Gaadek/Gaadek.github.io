---
layout: post
title: Snippet for converting a space delimited list to a comma delimited list
thumbnail-img: /assets/img/sas.png
tags: [sas,list,delimited,comma,space,quote]
---

This topic propose a small snippet to convert a macro variable, that contains a list of values separated by a space character, to a list of single quoted items spearated with a comma, so ready to use in a proc sql clause
```
*-- Transform a space delimited list to a quoted and comma delimited list --*;
%let out_list = %lowcase(%str(%')%qsysfunc(prxchange(s/\s+/%str(%', %')/, -1, &in_list.))%str(%'));
```

Example:
```
%let list = A B C D;

*-- Transform a space delimited list to a quoted and comma delimited list --*;
%let list = %lowcase(%str(%')%qsysfunc(prxchange(s/\s+/%str(%', %')/, -1, &list.))%str(%'));

%put >> &list.;
```

Returns:
```
>> 'a', 'b', 'c', 'd'
```
