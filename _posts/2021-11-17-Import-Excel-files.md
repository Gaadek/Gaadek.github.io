---
layout: post
title: Full article to load Excel files with advanced control
thumbnail-img: /assets/img/sas.png
tags: [sas,load,import,excel,xls,xlsx]
---

In this topic I propose a complete solution to load Excel files into SAS datasets.  
Most of the people performs a simple `proc import` (or equivalent), but this procedure has some disagreements.  
For example, an Excel column containing a numeric value will be loaded as a numeric SAS variables. Later, if the Excel colunm contains character values, the SAS variable will be assigned to a char value, so the SAS output is not consistent.  
Even more problematic, SAS consider only few records of the input Excel file to determine the type (i.e. num or char). What if all looked up data rare numeric, and far beyond we have some char values?

This article tries to assess all thes impacts for providing accurate and consistent SAS outputs:
1) Check the Excel file structure (columns definition)
2) Assign consistent output variable name
3) Ensure consistent output variable types

The first part consist in specifying the expected SAS dataset structure. This is achieved by creating a dataset with the definition of the output variables:
```
*-------------------------------------------------*;
*-- Definition of the output SAS dataset        --*;
*-- Specifications of the input Excel file      --*;
*-------------------------------------------------*;

data definition;
	attrib var_name		format=$32.;	  * Name of the SAS variable to create;
	attrib var_format	format=$30.;	  * Format of the SAS variable to create;
	attrib var_label	format=$200.;	  * Label of the SAS variable to create;
	attrib excel_label	format=$200.;	* Label of the column in Excel. If not specified, use SAS label.;
	attrib auto_conv	format=$1.;		  * Allow automatic conversion from num to char, set to N for manual conversion;

  *-- Add as many line as needed, 1 per output variable --*;
	var_name="subject";						var_format="$10.";			var_label="Patient ID";					excel_label="";	auto_conv="";	output;
	...
run;
```

The **definition** dataset lists all output variable.
The **excel_label** allow to specify what is the column's header in the Excel file.
The autoc_onv is a flag used to handle special variables formats, e.g. dates.


The next step consists in creating an empty dataset with the structure of the expected output dataset. This is done automatically using the **definition** dataset:
```
*-------------------------------------------------*;
*-- Create a struct dataset from the definition	--*;
*-------------------------------------------------*;

*-- Build the "attrib" statement from the definition --*;
proc sql noprint;
	select		"attrib " || strip(var_name) || " format=" || strip(var_format) || " label=" || quote(strip(var_label))
	into		:attrib_stmt separated by ';'
	from		definition
	;
quit;

*-- Create a SAS dataset with the expected structure --*;
data struct (drop=i);
	&attrib_stmt.;

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

Now we can load the Excel file. It can be worth to define where is located the header row and where start the data:
```
option validvarname=v7;

proc import datafile="...path/to/excel_file" out=raw_data dbms=xlsx replace;
  getnames	= yes;
	namerow		= &header_row.;
	datarow		= &data_row.;
run;
```

Note here you can use either xlsx or xls for the dbms. Usually, I extract the dbms valud from the file name, but it's out of scope here.

At this stage, we are ready to perform our first check.  
I like to embed everything is a macro function so I can control my execution flow, report errors with accurate messages and so on, so let's first define a macro function:
And 
```
%macro check_file;
```

And now, perform the first check about structure of the Excel file vs structure of the definition:  
Does the Excel file contain the expected number of columns (aka variables)?
```
	*-------------------------------------------------*;
	*-- Check input file structure                  --*;
	*-------------------------------------------------*;

	*-- Check the number of columns --*;
	proc sql noprint;
		select		count(*)
		into		:rcol_cnt trimmed
		from		dictionary.columns
		where		libname = 'WORK'
			and		memname = 'RAW_DATA'
		;

		select		count(*)
		into		:ecol_cnt trimmed
		from		dictionary.columns
		where		libname = 'WORK'
			and		memname = 'STRUCT'
		;
	quit;

	%if &rcol_cnt. ne &ecol_cnt %then %do;
		%put ERROR: Input file contains &rcol_cnt. columns whereas &ecol_cnt are expected.;
		%let syscc = 99;
		%return;
	%end;
%mend check_file
```

As you can see, in case of error, we exit the macro and so we don't run useless validation steps.

Next check is:
Does the Excel file contain the expected headers?
```
	*-- Retrieve the headers of the Excel file --*;
	proc sql noprint;
		select		strip(lowcase(name)),
					varnum
		into		:xls_headers separated by ' ',
					:dummy
		from		dictionary.columns
		where		libname = 'WORK'
			and		memname = 'RAW_DATA'
		order by	varnum
		;
	quit;

	*-- Retrieve the expected headers --*;
	proc sql noprint;
		select		tranwrd(strip(lowcase(coalesce(excel_label, var_label)))," ", "_")
		into		:exp_headers separated by ' '
		from		definition
		;
	quit;

	*-- Compare Excel headers vs expected headers --*;
	%do i=1 %to &rcol_cnt.;
		%let exp_var = %scan(%quote(&exp_headers.), &i.);
		%let xls_var = %scan(%quote(&xls_headers.), &i.);

		*-- In case of mismatch, output an error, but do not leave yet, compare all columns first to identify all issues --*;
		%if &xls_var. ne &exp_var. %then %do;
			%put ERROR: Input file contains column &xls_var. at index &i. whereas column &exp_var. was expected.;
			%let syscc = 99;
		%end;
	%end;

	*-- Exist if column mismatch has been identified --*;
	%if &syscc. = 99 %then %return;
%mend check_file
```

At this stage, we performed all our structural checks, we can go ahead and map the variables from the Excel file to the SAS variables from the definition.
We will do 2 things:
1) Rename variables if needed
2) Convert variable to the expected output data type.

To avoid collision in case of transtyping, we rename all the input variables:
```
	*-------------------------------------------------*;
	*-- Rename variables in raw_data		--*;
	*-------------------------------------------------*;

	*-- Get the list of variables in raw_data --*;
	proc sql noprint;
		select		distinct varnum,
					name,
					type,
					count(*)
		into		:dummy,
					:var_list separated by ' ',
					:var_type separated by ' ',
					:var_count
		from		dictionary.columns
		where		libname = 'WORK'
			and		memname = 'RAW_DATA'
		order by	varnum
		;
	quit;

	*-- Build the rename statement --*;
	%let rename_stmt=;

	%do i=1 %to &var_count.;
		*-- Rename variables to v_1, v_2, v_3, v_4... --*;
		%let ivar = %scan(&var_list., &i.);
		%let ovar = v_&i.;

		*-- Do not rename variables whose name does not change --*;
		%if %upcase(&ivar.) ne %upcase(&ovar.) %then %do;
			%let rename_stmt = &rename_stmt. &ivar.=&ovar.;
		%end;
	%end;

	*-- Apply the rename statement --*;
	%if &rename_stmt ne %then %do;
		proc datasets library=work nolist;
			modify raw_data;
				rename &rename_stmt.;
		quit;
	%end;
```

Now, all input variables are `v_1`, `v_2`, ...

Next, we have to identify the variables to be converted and the SAS variable name to be assigned.  
To make things easy, let's start by collecting these information in a dataset:
```
	*-- Built a mapping table between input variables and output variables --*;
	proc sql noprint;
		create table conv as
			select		r.varnum,
						r.name as src_name,
						r.type as src_type,
						o.name as tar_name,
						o.type as tar_type,
						o.format as tar_fmt
			from		dictionary.columns r

			left join	dictionary.columns o
			on			o.libname = 'WORK'
				and		o.memname = 'STRUCT'
				and		o.varnum = r.varnum

			left join	definition d
			on			d.var_name = o.name

			where		r.libname = 'WORK'
				and		r.memname = 'RAW_DATA'
				and		lowcase(d.auto_conv) ne "n"

			order by	1
		;
	quit;
```

Next, use this mapping dataset to create SAS statements we'll be able to use to both convert and assign final SAS name as per **definition**:
```
  *-- Build the statement to convert and clean input variables to output variables --*;
	proc sql noprint;
		select		case
						when tar_type = 'char' and src_type = 'char' then cat(strip(tar_name), ' = strip(', strip(src_name), ')')
						when tar_type = 'num' and src_type = 'num' then cat(strip(tar_name), ' = ', strip(src_name))
						when tar_type = 'num' and src_type = 'char' then cat(strip(tar_name), ' = input(', strip(src_name), ', best.)')
						when tar_type = 'char' and src_type = 'num' then cat(strip(tar_name), ' = strip(put(', strip(src_name), ', best.))')
						else ''
					end,
					case
						when tar_type = 'char' then cat(strip(tar_name), ' = strip(compress(', strip(tar_name), ', , "kw"))')
						else ''
					end
		into		:conv_stmt separated by ';',
					:clean_stmt separated by ';'
		from		conv
		;
	quit;
```

Finally, create the output dataset:
```
	*-- Fill imported data into the expected structure --*;
	data imp_data (drop = v_:);
		set struct raw_data;

		*-- Convert rawdata to the expected format --*;
		&conv_stmt.;

		*-- Clean data by removing non-writable chars --*;
		&clean_stmt.;
	run;
```

OK, we're pretty close. if you noticed, during the creation of the conv dataset, I excluded some definition records (`lowcase(d.auto_conv) ne "n").  
Hanled manually convertion here, e.g.:
```
  *-- Fill imported data into the expected structure --*;
	data imp_data (drop = v_:);
		set struct raw_data;

		*-- Convert rawdata to the expected format --*;
		&conv_stmt.;

		*-- Clean data by removing non-writable chars --*;
		&clean_stmt.;
    
    *-- Manual conversions (i.e manage variable whose auto_conv definition is "N") --*;
		attrib v_tmp format=$50.;
		v_tmp = prxchange('s/[ ]UTC//', -1, v_6);
		output_date = input(v_tmp, datetime20.);  *output_date should be defined as  var_name in definition dataset;
	run;
```

And lastly, ends your macro function:
```
%mend check_file
```

And voila, we have a clean method to load an Excel file:

