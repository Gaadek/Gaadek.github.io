---
layout: post
title: Extract folder, name and extension from file path
thumbnail-img: /assets/img/sas.png
tags: [sas,snippet,check,dataset,exist]
---

This snippet provides useful methods to extract from a file path:
1) Directory of the file
2) File name
3) File extension

## Sample
```
    %let file_path=/root/folder/subfolder/file.txt;
    
    %let _f_dir = %substr(&file_path, 1, %sysfunc(findc(&file_path, /, b)) - 1);
    * returns "/root/folder/subfolder"
    * can removes the '-1' to the subsrt function so the trailing slash will we returned
    
    %let _f_name = %scan(&file_path, -1, /);
    * returns "file.txt"
    
    %let _f_ext = %scan(&file_path, -1, .);
    * returns "txt"
    ...
```
