---
layout: post
title: Dealing with data intervals
thumbnail-img: /assets/img/sas.png
tags: [sas,snippet,date,interval,intck,intnx]
---

The purpose of this post is to explain how to simply deal with dates intervals in SAS.  
To achieve this, SAS provides 2 fuseful functions:
* `intck` (interval check) which returns the number of *time units* between two dates.
* `intnx` (interval next) which returns a date which is a specified number of *time units* away from a date.

The *time unit* is just a keyword to simply things. You can decide to deal with second, hours, days or year, the time unit is there to specify this. 
SAS has by default a lot of time units that are references here: [
Intervals Used with Date and Time Functions](https://documentation.sas.com/doc/en/pgmsascdc/9.4_3.5/leforinforref/n0pxq4af0hx60nn1i1x3xn41mc3c.htm#n0zn1re74n6pfvn170g9us41coho)

## intck
intck returns the number of time units between two dates.  
It syntax is: *intck(time_unit, start_date, end_date, method)*
* time_unit can be any interval unit (e.g., day, month, year, qtr, hour)
* start_date and end_date are date or datetime
* method is C (continuous) or D (discrete)

```
    ...
    *-- returns 0 because the two dates are within the same month --*;
    interval = intck('month', '01jan2021'd, '31jan2021'd);
    
    *-- returns 12 --*;
    interval = intck('month', '01jan2020'd, '01jan2021'd);
    
    *-- returns -12 --*;
    interval = intck('month', '01jan2021'd, '01jan2020'd);
    
    *-- easy method to calculate age! --*;
    interval = intck('year', event_date, date_of_birth, 'c');
    ...
```

## intnx
intnx returns a date which is a specified number of time units away from a date.  
It syntax is: *intnx(time_unit, start_date, increment, alignement)*
* time_unit can be any interval unit (e.g., day, month, year, qtr, hour)
* start_date is date or datetime
* alignement is B (beginning) or M (middle) or E (end) or S (same)

```
    ...
    *-- Returns 01APR2021, the 1st day of the next month --*;
    test = intnx('month', '04mar2021'd, 1);
    
    *-- Returns 04APR2021, exactly 1 month after--*;
    test = intnx('month', '04mar2021'd, 1, 'same');
    
    *-- Returns 28FEB2021, the last day of the next month --*;
    test = intnx('month', '15jan2021'd, 1, 'end');
    ...
```
