---
layout: post
title: Snippet to delete blank records from a dataset
thumbnail-img: /assets/img/sas.png
tags: [sas,snippet,delete,remove,blank,missing,record]
---

In this topic I propose a small macro function I'm using to manage comment variables that report multiples comments. 
Let's say you've got a variables "issues" that must report all issues encountered during the processing of a file: file extension, file nampe pattern, file strucure, variables formats and so on.
It can be messy to do that manually, so instead I suggest to use a function that will automatically appends some content to an existing value, using a predefined delimiter.

```
  *-- Fill imported data into the expected structure --*;
	data ...;
		set ...;

    *-- Remove blank records --*;
		if missing(cats(of _all_)) then delete;
	run;
```
