---
layout: post
title: Read, modify, then write a text file while retaining the formatting
thumbnail-img: /assets/img/sas.png
tags: [sas,file,read,write,modify,keep,format,formatting]
---

This post describes how I deal when I have to process a text file with SAS to apply some changes within. By text file, I mean any file whose content is text, it is usually a SAS program, a SQL file.  
It is common is such case to just read/modify and output the file, but this process removes leading spaces/tabulations, thus the output file is in general less comfortable to review.  
However, we can easily with little effort keep the original text formatting, and this is what I explain here.

## Reading the original file
I quickly describe the reading process which is very common stuff in SAS. Please note I always refer to external files as macro function, which makes the setup process much more easier by grouping all variable in the top section of my programs:
```
data tmp;
    infile "&input_file." delimiter='0A'x missover pad lrecl=2000 firstobs=1;

    attrib row_data format=$2000. informat=$char2000.;
    input row_data $;
```
The code above read the input file defined by the `input_file` macro variable. Nothing tricky there, I'm usign the correct delimiter so the code should work on both Windows & Unix systems.
I also assume the maximum length of a line will not exceed 2000 chars, this can be adjusted to your needs.

## Collect formatting information
As soon as I read input file data, I identify the position of the first non blank character (so it handles both space and tabulation):
```
    ...
    *-- Calculate row offset (ie. column position of 1st non null char) --*;
    attrib offset format=8.;
    offset = lengthn(row_data) - lengthn(left(row_data)) + 1;
    ...
```
After this step, we can perform modifications (renaming content, ...)

## Write output file with original formatting
Finally, once the modifications are complete, we can write the output file. Like the reading process, the writting process is straight forward:
```
filename outf "&output_file." lrecl=2000;

data _null_;
    file outf;

    set tmp;

    if not missing(row_data) then
        put @offset row_data;
    else
        put ;
run;
```
There, you can see when the data to be written are not empty, the we position the output cursor at the offset value collected at the reading process.

