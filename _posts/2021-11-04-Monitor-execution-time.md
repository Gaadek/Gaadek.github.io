---
layout: post
title: Snippet to monitor the execution duration of some SAS code
thumbnail-img: /assets/img/sas.png
tags: [sas,snippet,monitor,duration,time,timer,execution,run]
---

In this topic I propose a small piece of code to monitor the execution time of some SAS code. I use it from time to time to check if some code is more (or less) efficient than an existing solution.

The first part consist in collecting the start of our timer:
```
	*-- Start our custom timer --*;
	%let _timer_start = %sysfunc(datetime());
```

In this step, nothing really important, just get the current datetime

The second part consist in stopping the timer and calculating the execution duration of the code between the two actions:
```
	*-- Stop our custom timer --*;
	data _null_;
		duration = datetime() - &_timer_start;
		put 30*'-' / ' Execution duration:' duration time13.2 / 30*'-';
	run;
```

Easy and efficient!
