---
layout: post
title: Snippet to create and empty dataset
thumbnail-img: /assets/img/sas.png
tags: [sas,snippet,empty,array]
---

Short post to share my method on how to create empty datasets (no record) without uninitialized veriables notes.

```
data struct (drop=i);
	attrib	a format=$10.
		  	b format=8.
		  	c format=date9.
  	;

	set _null_;

	*-- Init all variables --*;
	array my_chars[*] _character_;
	do i=1 to dim(my_chars);
		my_chars[i] = '';
	end;

	array my_nums[*] _numeric_;
	do i=1 to dim(my_nums);
		my_nums[i] = .;
	end;
run;
```
