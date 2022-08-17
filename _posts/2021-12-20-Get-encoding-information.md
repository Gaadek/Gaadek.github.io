---
layout: post
title: Quick topic about encoding
thumbnail-img: /assets/img/sas.png
tags: [sas,encoding]
---

In this topic I do some proposal to cover encoding.  

1) How to identify the encoding of the current SAS session?  
Use the `sysencoding` macro variable:  
```
%put >>> System encoding: &sysencoding;
```

2) How to identify the encoding of a SAS dataset?  
Use the `encoding` value from dictionary.tables:  
```
%macro display_encoding(ds_in);
	*-- Set work as the default library --*;
	%let lib = work;

	%if %sysfunc(countw(&ds_in., .)) > 1 %then %do;
		%let lib = %scan(&ds_in., 1, .);
	%end;

	%let ds = %scan(&ds_in., -1, .);

	%let ds_encoding=;

	proc sql noprint;
		select		encoding
		into		:ds_encoding trimmed
		from		dictionary.tables
		where		libname = strip(upcase("&lib."))
			and		memname = strip(upcase("&ds."))
		;
	quit;

	%put >>> Dataset encoding of %upcase(&ds_in.): &ds_encoding;
%mend display_encoding;

%display_encoding(sashelp.cars);
```
