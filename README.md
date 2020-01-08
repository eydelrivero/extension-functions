# extension-functions

## Changes
Added new window function `medianw` which can be used as a window function. 

## Build

#### macOs
```bash
gcc -g -fPIC -dynamiclib extension-functions.c -o extension-functions.dylib
cp extension-functions.dylib /usr/local/lib
```

#### Linux
```bash
gcc -g -fPIC -shared extension-functions.c -o extension-functions.so
cp extension-functions.so /usr/lib  # you might need root permissions for this
```

## Test

To tests, run this script:
```sql
-- setup test table
CREATE TABLE t3(x, y);
INSERT INTO t3 VALUES('a', 4),
                     ('b', 5),
                     ('c', 3),
                     ('d', 8),
                     ('e', 1);
                     
-- load the extension                     
SELECT load_extension('extension-functions');

-- try to use original median function as a window function
SELECT x,
       median(y) OVER (
           ORDER BY x ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING
           ) AS median_y
FROM t3
ORDER BY x;
-- OUTPUT: Error: median() may not be used as a window function

-- repeat the test with the new medianw function
SELECT x,
       medianw(y) OVER (
           ORDER BY x ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING
           ) AS median_y
FROM t3
ORDER BY x;
-- OUTPUT:
--   x | median_y
--------------
--   a | 4.5
--   b | 4
--   c | 5
--   d | 3
--   e | 4.5

```

Original `median` function is still available but can be easily replaced by the new version by deleting/commenting line:  
```c
{ "median",           1, 0, 0, modeStep,     medianFinalize  },
```
 and then registering the new version as `median` in this line  
```c
sqlite3_create_window_function(db, "median", 1, SQLITE_UTF8, 0,
```

## Original description by Liam Healy
Provide mathematical and string extension functions for SQL queries in SQLite using the loadable extensions mechanism. Math: acos, asin, atan, atn2, atan2, acosh, asinh, atanh, difference, degrees, radians, cos, sin, tan, cot, cosh, sinh, tanh, coth, exp, log, log10, power, sign, sqrt, square, ceil, floor, pi. String: replicate, charindex, leftstr, rightstr, ltrim, rtrim, trim, replace, reverse, proper, padl, padr, padc, strfilter. Aggregate: stdev, variance, mode, median, lower_quartile, upper_quartile.
