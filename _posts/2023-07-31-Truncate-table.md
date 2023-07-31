---
layout: post
title: Proc sql truncate table
thumbnail-img: /assets/img/sas.png
tags: [sas,sql,truncate]
---

There is no `truncate` statement in proc sql, but there is a workaround:  
```
proc sql noprint;
  create table test like test;
quit;
```

Note the reference to the same dataset after the `create table` and the `like` statements.
