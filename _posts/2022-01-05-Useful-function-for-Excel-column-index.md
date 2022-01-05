---
layout: post
title: Useful functions to work with Excel column index
thumbnail-img: /assets/img/sas.png
tags: [sas,excel,index,column]
---

This topic propose some functions to work with Excel data loaded into SAS. One issue I regularly encounter is: how do I change a numeric index (1,2,3...) to an Excel index (A,B,C...) and vice versa.
On first review, it may appear difficult to handle, but in fact, it isn't. Let's fix that once for all

1) How to convert a numeric index to an Excel column index?  
    The purpose is to convert a numeric value (1, 2, 3...) into a valid Excel column index (A, B, C, ..., AA, AB...)
    
    The basis is very simple: to convert a numeric value to a character value, we "just" have to convert the numerci to a byte. 
    Easy read like this, isn't it? However, we have to convert to meaningul characters (A, B, C, ...) so we can check in the ASCII table which vyte value corresponds to "A" (it's byte(65)) to define a first conversion formulae:
    `char = byte(num + 64)`
    
    However, beyond index 26 (aka "Z"), we cannot simply convert the numeric (in the ASCII table, we have only unique chars).  
    
    
    The solution Here, it's simple: if the index is more than 26, decompose the value as 2 characters, the first one being , so we must build a string (e.g. AA) decompose the numeric index in multiple of 26, wich is basicaly an euclidean division.
 
    ```
    *-- Option to avoid warnings in case of re-run of proc fcmp --*;
    options cmplib = _null_;

    *-- Build a user package to store functions --*;
    proc fcmp outlib=work.funcs.tmp;
	    *-- Function to convert an index (1, 2, 3) to an Excel index (A, B, C) --*;
	    function xls_col_conv(int_index) $;
    		length res $10;
    		res = '';

	    	q = int((int_index - 1) / 26);
    
		    if q > 0 then do;
			    do while (q > 26);
				    q = int((q - 1) / 26);
				    res = strip(res) || byte(q + 64);
			    end;

			    res = strip(res) || byte(q + 64);
		    end;

		    m = mod(int_index - 1, 26) + 1;
		    res = strip(res) || byte(m + 64);

		    return(res);
	    endsub;
    run;

    options cmplib=work.funcs;
    ```

    ```
    *-- Option to avoid warnings in case of re-run of proc fcmp --*;
    options cmplib = _null_;

    *-- Build a user package to store functions --*;
    proc fcmp outlib=work.funcs.tmp;
        *-- Function to convert an Excel column index (A,B,C...) to an integer index (1,2,3...) --*;
        function xls_idx_to_num(excel_index $);
            length res 8;
            res = .;
            put "Excel index: " excel_index;

            do _t = 1 to length(excel_index);
                coef = length(excel_index) - _t;
                cur_val = rank(char(upcase(excel_index), _t)) - 64;

                if coef then	res = sum(res, cur_val * 26 * coef);
                else			res = sum(res, cur_val);
            end;

            return(res);
        endsub;
    run;

    options cmplib=work.funcs;
    ```
