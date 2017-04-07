Functions defined in q.k are loaded as part of the "bootstrap" of q. (They are of course written in k.) Some are exposed in the root namespace as the q language. Others are documented here as utility functions in the .Q namespace. 

!!! warning "Undocumented functions in the .Q namespace"
    Consider _undocumented_ functions in the .Q namespace as a private API – and do not use them. 


## General 

### `.Q.addmonths` 

Syntax: `.Q.addmonths[x;y]`

Where `x` is a date and `y` is an int, returns `x` plus `y` months. (Since V2.4.)
```q
q).Q.addmonths[2007.10.16;6 7]
2008.04.16 2008.05.16
```
If the date `x` is near the end of the month and (`x.month + y`)’s month has fewer days than `x.month`, the result may spill over to the following month.
```q
q).Q.addmonths[2006.10.29;4]
2007.03.01
```


### `.Q.addr` (IP address)

Syntax: `.Q.host x`

Where `x` is a hostname or IP address as a symbol atom, returns the IP address as an integer (same format as [`.z.a`](dotz/#za-ip-address))
```q
q).Q.addr`localhost
2130706433i
q).Q.host .Q.addr`localhost
`localhost
```
<i class="fa fa-hand-o-right"></i> [`.Q.host`](#qhost-hostname)


### `.Q.chk` (fill HDB)

Syntax: `.Q.chk x`

Where `x` is a HDB as a filepath, fills missing tables.
```q
q).Q.chk[`:hdb]
```
Note that q must have write permission for the HDB area so that it can create missing tables. If it signals an error similar to
```q
'./2010.01.05/tablename/.d: No such file or directory
```
then check that the process has write permissions for that filesystem.


### `.Q.cn` (count partitioned table)

Syntax: `.Q.cn x`

Where `x` is a partitioned table, passed by value, returns its count. Populates `.Q.pn` cache.



### `.Q.dd` (join symbols)

Syntax: `.Q.dd[x;y]`

Shorthand for `` ` sv x,`$string y``. Useful for creating filepaths, suffixed stock symbols, etc.
```q
q).Q.dd[`:dir]`file
`:dir/file
q){x .Q.dd'key x}`:dir
`:dir/file1`:dir/file2
q).Q.dd[`AAPL]"O"
`AAPL.O
q)update sym:esym .Q.dd'ex from([]esym:`AAPL`IBM;ex:"ON")
esym ex sym
--------------
AAPL O  AAPL.O
IBM  N  IBM.N
```


### `.Q.def`

Syntax: `.Q.def[x;y]`

==FIXME==


### `.Q.dpft` (save table)

Syntax: ``.Q.dpft[directory;partition;`p#field;tablename]``

Saves a simple table splayed to a specific `partition` of a database sorted (`` `p#``) on a specified `field`. 

!!! warning 
    The table cannot be keyed. This will signal an `'unmappable` error if there are columns which are not vectors or simple nested columns (e.g. char vectors for each row).

It also rearranges the columns of the table so that the column specified by `field` is second in the table (the first column in the table will be the virtual column determined by the partitioning e.g. date).

Returns the table name if successful.
```q
q)trade:([]sym:10?`a`b`c;time:.z.T+10*til 10;price:50f+10?50f;size:100*1+10?10)
q).Q.dpft[`:db;2007.07.23;`sym;`trade]
`trade
q)delete trade from `.
`.
q)trade
'trade
q)\l db
q)trade
date       sym time         price    size
-----------------------------------------
2007.07.23 a   11:36:27.972 76.37383 1000
2007.07.23 a   11:36:27.982 77.17908 200
2007.07.23 a   11:36:28.022 75.33075 700
2007.07.23 a   11:36:28.042 58.64531 200
2007.07.23 b   11:36:28.002 87.46781 800
2007.07.23 b   11:36:28.012 85.55088 400
2007.07.23 c   11:36:27.952 78.63043 200
2007.07.23 c   11:36:27.962 90.50059 400
2007.07.23 c   11:36:27.992 73.05742 600
2007.07.23 c   11:36:28.032 90.12859 600
```
If you are getting an `'unmappable` error, you can identify the offending columns and tables:
```q
/ create 2 example tables
q)t:([]a:til 2;b:2#enlist (til 1;10))  / bad table, b is unmappable
q)t1:([]a:til 2;b:2#til 1)  / good table, b is mappable
q)helper:{$[(type x)or not count x;1;t:type first x;all t=type each x;0]};
q)select from (raze {([]table:enlist x;columns:enlist where not helper each flip .Q.en[`:.]`. x)} each tables[]) where 0<count each columns
table columns
-------------
t     b
```


### `.Q.dsftg` (load process save)

Syntax: `.Q.dsftg[d;s;f;t;g]`

Where 

- `d` is `(dst;part;table)`
- `s` is `(src;offset;length)`
- `f` is fields as a symbol vector
- `t` is `(types;widths)
- `g` is a unary post-processing function

loops M&1000000 rows at a time. 
For example, loading TAQ DVD:
```q
q)d:(`:/dst/taq;2000.10.02;`trade)
q)s:(`:/src/taq;19;0)  / nonpositive length from end
q)f:`time`price`size`stop`corr`cond`ex
q)t:("iiihhc c";4 4 4 2 2 1 1 1)
q)g:{x[`stop]=:240h;@[x;`price;%;1e4]}
q).Q.dsftg[d;s;f;t;g]
```


### `.Q.en` (enumerate varchar cols)

Syntax: `.Q.en[x;y]`

<i class="fa fa-hand-o-right"></i> [Enumerating varchar columns in a table](http://code.kx.com/wiki/Cookbook/SplayedTables#Enumerating_varchar_columns_in_a_table)


### `.Q.f` (format)

Syntax: `.Q.f[x;y]`

Where `x` is an int atom and `y` is a numeric atom, returns `y` as a string formatted as a float to `x` decimal places.

Because of the limits of precision in a double, for `y` above `1e13` or the limit set by `\P`, formats in scientific notation.
```q
q)\P 0
q).Q.f[2;]each 9.996 34.3445 7817047037.90 781704703567.90 -.02 9.996 -0.0001
"10.00"
"34.34"
"7817047037.90"
"781704703567.90"
"-0.02"
"10.00"
"-0.00"
```
The `1e13` limit is dependent on `x`. The maximum then becomes `y*10 xexp x` and that value must be less than `1e17` – otherwise you'll see sci notation or overflow.
```
q)10 xlog 0Wj-1
18.964889726830812
```

### `.Q.fc` (parallel on cut)

Syntax: `.Q.fc[x;y]`

Where `x` is is a unary atomic function and `y` is a list, returns the result of evaluating `f vec` – using multiple threads if possible. (Since V2.6)
```q
q -s 8
q)f:{2 xexp x}
q)vec:til 100000
q)\t f vec
12
q)\t .Q.fc[f]vec
6
```
In this case the overhead of creating 100,000 threads in `peach` significantly outweighs the computational benefit of parallel execution.
```q
q)\t f peach vec
45
```


### `.Q.ff` (append columns)

Syntax: `.Q.ff[x;y]`

Where `x` is table to modify, and `y` is a table of columns to add to `x` and set to null, returns `x`, with all new columns in `y`, with values in new columns set to null of the appropriate type.

If there is a common column in `x` and `y`, the column from `x` is kept (i.e. it will not null any columns that exist in `x`).
```q
q)src:0N!flip`sym`time`price`size!10?'(`3;.z.t;1000f;10000)
 sym time         price    size
 ------------------------------
 mil 10:30:32.148 470.7883 6360
 igf 00:28:17.727 634.6716 7885
 kao 06:52:34.397 967.2398 4503
 baf 10:07:47.382 230.6385 4204
 kfh 00:45:40.134 949.975  6210
 jec 05:12:49.761 439.081  8740
 kfm 16:31:50.104 575.9051 8732
 lkk 04:54:11.685 591.9004 4756
 kfi 13:01:04.698 848.1567 3998
 fgl 05:18:45.828 389.056  9342
 
q).Q.ff[src] enlist `sym`ratioA`ratioB!3#1
 sym time         price    size ratioA ratioB
 --------------------------------------------
 mil 10:30:32.148 470.7883 6360
 igf 00:28:17.727 634.6716 7885
 kao 06:52:34.397 967.2398 4503
 baf 10:07:47.382 230.6385 4204
 kfh 00:45:40.134 949.975  6210
 jec 05:12:49.761 439.081  8740
 kfm 16:31:50.104 575.9051 8732
 lkk 04:54:11.685 591.9004 4756
 kfi 13:01:04.698 848.1567 3998
 fgl 05:18:45.828 389.056  9342
```



### `.Q.fk` (foreign key)

Syntax: `.Q.fk x`

Where `x` is a table column, returns `` ` `` if the column is not a foreign key or `` `tab`` if the column is a foreign key into `tab`.(Since V2.4t)


### `.Q.fmt` (format)

Syntax: `.q.fmt[x;y;z]`

Where `x` and `y` are integer atoms and `z` is a numeric atom, returns `z` as a string of length `x`, formatted to `y` decimal places. (Since V2.4)
```q
q).Q.fmt[6;2]each 1 234
"  1.00"
"234.00"
```
Q) Is it possible to format the decimal data in a column to 2 decimal places?  
A) Yes, through changing it to string
```
q)fix:{.Q.fmt'[x+1+count each string floor y;x;y]}
q)fix[2]1.2 123 1.23445 -1234578.5522
"1.20"
"123.00"
"1.23"
"-1234578.55"
```
also handy for columns (view in a fixed-width font for proper effect):
```q
q)align:{neg[max count each x]$x}
q)align fix[2]1.2 123 1.23445 -1234578.5522
"       1.20"
"     123.00"
"       1.23"
"-1234578.55"
```
Q) I have a table with float values. Those values have to be persisted to a file as character strings of length 9, e.g. 34.3 --> `"     34.3"`
I would also like to keep as much precision as possible, i.e. 343434.3576 should be persisted as `"343434.36"`
What is the best way of doing that?  
A)
```
q)fmt:{.Q.fmt[x;(count 2_string y-i)&x-1+count string i:"i"$y]y}
q)fmt[9] each 34.4 343434.358

"     34.4"
"343434.36"
```


### `.Q.fps` (streaming algorithm)

Syntax: `.Q.fps[x;y]`

`.Q.fs` for pipes. (Since V3.4)
Reads conveniently sized lumps of complete `"\n"` delimited records from a pipe and applies a function to each record. This enables you to implement a streaming algorithm to convert a large CSV file into an on-disk kdb+ database without holding the data in memory all at once.

<i class="fa fa-hand-o-right"></i> [Cookbook/Named Pipes](http://code.kx.com/wiki/Cookbook/NamedPipes)


### `.Q.fs` (streaming algorithm)

Syntax: `.Q.fs[x;y]`

Where `x` is a unary function and `y` is a filepath, loops through `y` (grabbing conveniently sized lumps of complete `"\n"`-delimited records) and applies function `x` to each record. This enables you to implement a streaming algorithm to load a large CSV file into an on-disk database without holding the data in memory all at once.

For example, assume that the file potamus.csv contains the following:
```csv
Take, a,   hippo, to,   lunch, today,        -1, 1941-12-07
A,    man, a,     plan, a,     hippopotamus, 42, 1952-02-23
```
If you call `.Q.fs` on this file with the function `0N!`, you get the following list of rows:
```q
q).Q.fs[0N!]`:potamus.csv
("Take, a,   hippo, to,   lunch, today,        -1, 1941-12-07";"A,    man, a,..
120
```
`.Q.fs` can also be used to read the contents of the file into a list of columns.
```q
q).Q.fs[{0N!("SSSSSSID";",")0:x}]`:potamus.csv
(`Take`A;`a`man;`hippo`a;`to`plan;`lunch`a;`today`hippopotamus;-1 42i;1941.12..
120
```
<i class="fa fa-hand-o-right"></i> [Loading large CSV files](http://code.kx.com/wiki/Cookbook/LoadingFromLargeFiles)


### `.Q.fsn` (streaming algorithm)

Syntax: `.Q.fsn[x;y;z]`

Loops over a file and grabs `z`-sized lumps of complete `"\n"` delimited records and applies a function to each record. This enables you to implement a streaming algorithm to convert a large CSV file into an on-disk database without holding the data in memory all at once. 

`.Q.fsn` is almost identical to `.Q.fs` but it takes an extra argument: `z` is the size in bytes that chunks will be read in. This is particularly useful for balancing load speed and ram usage. `.Q.fs` is in fact a projection of `.Q.fsn` with the size set to 131000 bytes by default.


### `.Q.ft` (apply simple)

Syntax: `.Q.ft[x;y]`

Where 

- `y` is a keyed table
- `x` is a unary function `x[t]` in which `t` is a simple table, and the 
result is a table with at least as many key columns as `t` 

As an example, note that you can index into a simple table with row indices, but not into a keyed table - for that you should use a select statement. However, to illustrate the method, we show an indexing function being applied to a keyed table.
```q
q)\l sp.q

q)sp 2 3           / index simple table with integer list argument
s  p  qty
---------
s1 p3 400
s1 p4 200

q)s 2 3            / index keyed table fails
'length
``
Now create an indexing function, and wrap it in `.Q.ft`. This works on both types of table:
```q
q).Q.ft[{x 2 3};s]
s | name  status city
--| -------------------
s3| blake 30     paris
s4| clark 20     london
```
Equivalent select statement:
```q
q)select from s where i within 2 3
s | name  status city
--| -------------------
s3| blake 30     paris
s4| clark 20     london
```


### `.Q.fu` (apply unique)

Syntax: `.Q.fu[x;y]`

Where `x` is a unary atomic function and `y` is a list, returns `x[y]` after evaluating `x` only on distinct items of `y`.
```q
q)n: 100000; vec: n ? 30 / long vectors with few different values
q)f:{[x] exp (x*x)} / e raised to x*x
q)\t y: f each vec / returns 270 (milliseconds)
q)\t y2: .Q.fu[f] vec / returns 10 (milliseconds)
q)y ~ y2 / returns 1b, the outputs are equal
```


### `.Q.gc` (garbage collect)

Syntax: `.Q.gc[]`

Returns the amount of memory that was returned to the OS. 
(Since V2.7 2010.08.05, enhanced with coalesce in V2.7 2011.09.15, and executes in slave threads since V2.7 2011.09.21)

!!! note "How it works"
    Kdb+ uses reference counting and <i class="fa fa-external-link-square"></i> <a target="_blank" href="http://en.wikipedia.org/wiki/Buddy_memory_allocation">buddy memory allocation</a>. The chosen buddy algorithm dices bucket sizes according to powers of 2, and the heap expands in powers of 64MB.
    
    Reference counting means there is never any garbage (so `.Q.gc` is not accurately named) and memory is returned to the heap as soon as it is no longer referenced; if that memory is a vector using >=64MB it may be returned immediately to the OS depending on the command-line option `-g`.
    `.Q.gc` attempts to coalesce diced blocks into their original 64MB block, and then returns blocks >=64MB to the OS.
    
    Coalescing is always deferred, i.e. can only be triggered by a call to `.Q.gc`.
    When slave threads are used, `.Q.gc` in the main thread also executes `.Q.gc` in the slave threads.
    `.Q.gc` can take several seconds to execute on large memory systems that have a fragmented heap, and hence is not recommended for frequent use in a time-critical path of code. Consider running with the command-line option `-g 1`, which will return larger blocks of memory to the OS without trying to coalesce the smaller blocks.
```q
q)a:til 10000000
q).Q.w[]
used| 67233056
heap| 134217728
peak| 134217728
wmax| 0
mmap| 0
syms| 534
symw| 23926
q).Q.gc[]
0j
q)delete a from `.
`.
q).Q.gc[]
67108864j
q).Q.w[]
used| 128768
heap| 67108864
peak| 134217728
wmax| 0
mmap| 0
syms| 535
symw| 23956
```
Note that memory can become fragmented and therefore difficult to release back to the OS. 
```q
q)v:{(10#"a";10000#"b")}each til 10000000;
q).Q.w[]
used| 164614358256
heap| 164752261120
peak| 164752261120
wmax| 0
mmap| 0
mphy| 270538350592
syms| 569
symw| 24934
q).Q.gc[]
134217728
q).Q.w[]
used| 164614358256
heap| 164618043392
peak| 164752261120
wmax| 0
mmap| 0
mphy| 270538350592
syms| 570
symw| 24964
q)v:v[;0] / just retain refs to the small char vectors of "aaaaaaaa"
q)/the vectors of "bbb.."s will come from the same memory chunks
q)/so can't be freed
q).Q.gc[]
134217728
q).Q.w[]
used| 454358256
heap| 164618043392
peak| 164752261120
wmax| 0
mmap| 0
mphy| 270538350592
syms| 570
symw| 24964
q)v:-8!v;0N!.Q.gc[];v:-9!v;.Q.w[] / serialize, release, deserialize
164483825664 / amount freed by gc
used| 454358848
heap| 738197504
peak| 164886478848
wmax| 0
mmap| 0
mphy| 270538350592
syms| 570
symw| 24964
```
So if you have many nested data, e.g. columns of char vectors, or an awful lot of grouping you may be fragmenting memory quite heavily.


### `.Q.hdpf` (save tables)

Syntax: ``.Q.hdpf[historicalport;directory;partition;`p#field]``

Saves all tables by calling `.Q.dpft`, clears tables, and sends reload message to HDB.


### `.Q.hg` (HTTP get)

Syntax: `.Q.hg x`

Where `x` is a URL as a symbol atom, returns as a list of strings the result of an HTTP[S] GET query.
(Since V3.4)
```q
q).Q.hg`:http://www.google.com
q)count a:.Q.hg`:http:///www.google.com
212
q)show a
"<!DOCTYPE HTML PUBLIC \"-//IETF//DTD HTML 2.0//EN\">\n<html><head>\n<title>4..
```
If you have configured SSL/TLS, HTTPS can also be used.
```q
q).Q.hg`:https://www.google.com
```
It will utilize proxy settings from the environment, lower-case versions taking precedence:

`HTTP_PROXY`, `http_proxy`
: The URL of the HTTP proxy to use

`NO_PROXY`, `no_proxy`
: Comma-separated list of domains for which to disable use of proxy

N.B. HTTPS is not supported across proxies which require `CONNECT`.


### `.Q.host` (hostname)

Syntax: `.Q.host x`

Where `x` is an IP address as an int atom, returns its hostname as a symbol atom.
```q
q).Q.host .Q.addr`localhost
`localhost
q).Q.addr`localhost
2130706433i
```


### `.Q.hp` (HTTP post)

Syntax: ``.Q.hp[x;y]"payload"``

Where `x` is a URL as a symbol atom, `y` is a MIME type as a string, and `z` is the POST query as a string, returns as a list of strings the result of an HTTP[S] POST query.
(Since V3.4)


### `.Q.id` (purge)

Syntax: `.Q.id x`

Where `x` is

- a symbol atom, returns `x` purged of non-alphanumeric characters
```q
q).Q.id each (`$"a/b";`in)
`ab`in
```
- a table, returns `x` with columns renamed by removing characters that interfere with `select/exec/update` and adding `"1"` to column names which clash with commands in .q namespace. (Updated in V3.2 to include `.Q.res` for checking collisions.) 
```q
q).Q.id flip (5#.Q.res)!(5#())
in1 within1 like1 bin1 binr1
----------------------------
q).Q.id flip(`$("a";"a/b"))!2#()
a ab
----
```


### `.Q.ind` (partitioned index)

Syntax: `.Q.ind[x;y]`

Where `x` is a partitioned table, and `y` is a **long** int vector of row indexes into `x`, returns rows `y` from `x`. 

When picking individual records from an in-memory table you can simply use the special virtual field `i`:
```q
select from table where i<100
```
But you can't do that directly for a partitioned table.
`.Q.ind` comes to the rescue here, it takes a table and (long!) indexes into the table - and returns the appropriate rows.
```q
.Q.ind[trade;2 3j]
```
A more elaborate example that selects all the rows from a date:
```q
q)t:select count i by date from trade
q)count .Q.ind[trade;(exec first sum x from t where date<2010.01.07)+til first exec x from t where date=2010.01.07]
28160313
/ show that this matches the full select for that date
q)(select from trade where date=2010.01.07)~.Q.ind[trade;(exec first sum x from t where date<2010.01.07)+til first exec x from t where date=2010.01.07]
1b
```

!!! tip "Continuous row intervals"
    If you are selecting a continuous row interval, for example if iterating over all rows in a partition, instead of using `.Q.ind` you might as well use
    ```q
    q)select from trade where date=2010.01.07,i within(start;start+chunkSize)
    ```


### `.Q.j10` (encode binhex)
### `.Q.x10` (decode binhex)
### `.Q.j12` (encode base64)
### `.Q.x12` (decode base64)

Syntax: `.Q.j10 s`  
Syntax: `.Q.x10 s`  
Syntax: `.Q.j12 s`  
Syntax: `.Q.x12 s`

Where `s` is a string, these functions return `s` encoded (`j10`, `j12`) or decoded (`x10`, `x12`) against restricted alphabets:

- `j10` encodes against the alphabet `.Q.b6`, this is a base-64 encoding - see <i class="fa fa-external-link-square"></i> <a target="_blank" href="https://en.wikipedia.org/wiki/BinHex">BinHex</a> and <i class="fa fa-external-link-square"></i><a target="_blank" href="https://en.wikipedia.org/wiki/Base64">Base64</a> for more details than you ever want to know about which characters are where in the encoding. To keep the resulting number an integer the maximum length of `s` is 10.
- `j12` encodes against `.Q.nA`, a base-36 encoding. As the alphabet is smaller `s` can be longer – maximum length 12.

The main use of these functions is to encode long alphanumeric identifiers (CUSIP, ORDERID..) so they can be quickly searched – but without filling up the symbol table with vast numbers of single-use values.
```q
q).Q.x10 12345j
"AAAAAAADA5"
q).Q.j10 .Q.x10 12345j
12345j
q).Q.j10 each .Q.x10 each 12345j+1 2 3
12346 12347 12348j
q).Q.x12 12345j
"0000000009IX"
q).Q.j12 .Q.x12 12345j
12345j
```

!!! tip
    If you don't need the default alphabets it can be very convenient to change them to have a blank as the first character, allowing the identity `0` <-> `" "`.
    
    If the values are not going to be searched (or will be searched with `like`) then keeping them as nested character is probably going to be simpler.



### `.Q.k` (version)

Syntax: `.Q.k`

Returns the interpreter version number for which q.k has been written:
checked against [`.z.K`](dotz/#zk-version) at startup. 


### `.Q.l` (load)

Syntax: ==FIXME==

Implements [`\l`](syscmds/#l-load-file-or-directory). 


### `.Q.M` (long infinity)

Syntax: `.Q.M`

Returns long integer infinity.
```q
q)0Wj~.Q.M
1b
```


### `.Q.MAP` (maps partitions)

Syntax: `.Q.MAP[]`

Added in V3.1, keeps partitions mapped to avoid the overhead of repeated file system calls during a `select`.

For use with partitioned HDBS, used in tandem with `\l dir`
```q
q)\l .
q).Q.MAP[]
```
When using `.Q.MAP[]` you can't access the date column outside of the usual: 
```q
select … [by date,…] from … where [date …]
```
NOT recommended for use with compressed files, as the decompressed maps will be retained, using physical memory|swap.

!!! note "File handles and maps" 
    You may need to increase the number of available file handles, and also the number of available file maps. For Linux see vm.max_map_count.


### `.Q.opt` (command parameters)

Syntax: `.Q.opt .z.x`

Returns a dictionary, so you can easily see if a key was defined (flag set or not) or, if a value is passed, to refer to it by its key.

<i class="fa fa-hand-o-right"></i> [`.z.x`](dotz/#zx-argv)


### `.Q.par` (locate partition)

Syntax: `.Q.par[dir;part;table]`

Where `dir` is a directory filepath, `part` is a date, returns the location of `table`. (Sensitive to `par.txt`.)
```q
q).Q.par[`:.;2010.02.02;`quote]
`:/data/taq/2010.02.02/quote
```
Can assist in checking `` `p`` attribute is present on all partitions of a table in an HDB
```
q)all{`p=attr .Q.par[`:.;x;`quote]`sym}each  date
1b
```


### `.Q.qp` (is partitioned)

Syntax: `.Q.qp x`

Where `x` 

- is a partitioned table, returns `1b`
- a splayed table, returns `0b` 
- anything else, returns 0

```q
q)\
  B
+`time`sym`price`size!`B
  C
+`sym`name!`:C/
  \
q).Q.qp B
1b
q).Q.qp select from B
0
q).Q.qp C
0b
```


### `.Q.qt` (is table)

Syntax: `.Q.qt x`

Where `x` is a table, returns `1b`, else `0b`.


### `.Q.res` (k words)

Syntax: `.Q.res`

Returns the k control words and functions as a symbol vector. ``key `.q`` returns the functions defined to extend k to the q language. Hence to get the full list of reserved words for the current version:
```q
q).Q.res,key`.q
`abs`acos`asin`atan`avg`bin`binr`cor`cos`cov`delete`dev`div`do`enlist`exec`ex..
```
<i class="fa fa-hand-o-right"></i> `.Q.id`


### `.Q.s` (plain text)

Syntax: `.Q.s x`

Returns `x` formatted to plain text, as used by the console. Obeys console width and height set by [`\c`](syscmds/#c-console-size).
```q
q).Q.s ([h:1 2 3] m: 4 5 6)
"h| m\n-| -\n1| 4\n2| 5\n3| 6\n"
```
Occasionally useful for undoing "Studio for kdb+" tabular formatting.


### `.Q.ty` (type)

Syntax: `.Q.ty x`

Returns the type of `x` as a character code.
```q
q).Q.ty 1 2
"i"
q).Q.ty 1 2.
"f"
```


### `.Q.v` (value)

Syntax: `.Q.v x`

Where `x` is

- a filepath, returns the splayed table stored at `x`
- any other symbol, returns the global named `x`
- anything else, returns `x`


### `.Q.V` (table to dict)

Syntax: `.Q.V x`

Where `x` is 

- a table, returns a dictionary of its column values. 
- a partitioned table, returns only the last partition (N.B. the partition field values themselves are not restricted to the last partition but include the whole range).

<i class="fa fa-hand-o-right"></i> [`meta`](metadata/#meta)


### `.Q.view` (subview)

Syntax: `.Q.view x`

Set a subview
```q
.Q.view 2#date
```


### `.Q.w` (memory stats)

Syntax: `.Q.w[]`

Returns the memory stats from [`\w`](syscmds/#w-workspace) into a more readable dictionary.
```q
q).Q.w[]
used| 168304
heap| 67108864
peak| 67108864
wmax| 0
mmap| 0
mphy| 8589934592
syms| 577
symw| 25436
```


### `.Q.x` (non-command parameters)

Syntax: `.Q.x`

Set by `.Q.opt`: a list of _non-command_ parameters from the command line, where _command parameters_ are prefixed by `-`.
```bash
~$ q taq.k path/to/source path/to/destn
```
```q
q)cla:.Q.opt .z.X /command-line arguments
q).Q.x
"/Users/me/q/m64/q"
"path/to/source"
"path/to/destn"
```
<i class="fa fa-hand-o-right"></i> [`.z.x`](dotz/#zx-argv), [`.z.X`](dotz/#zx-raw-command-line)


### `.Q.Xf` (create file)

Syntax: `.QXf[x;y]`

Where `x` is a mapped nested datatype as either an upper-case char atom, or as a short symbol (e.g. `` `char``) and `y` is a filepath, creates an empty nested-vector file at `y`.
```q
q).Q.Xf["C";`:emptyNestedCharVector];
q)type get`:emptyNestedCharVector
87h 
```


## Partitioned database state


### `.Q.bv` (build vp)

Syntax: `.Q.bv[]`  
Syntax: ``.Q.bv[`]``

In partitioned DBs, construct the dictionary `.Q.vp` of table schemas for tables with missing partitions. Optionally allow tables to be missing from partitions, by scanning partitions for missing tables and taking the tables’ prototypes from the last partition. After loading/re-loading from the filesystem, invoke `.Q.bv[]` to (re)populate `.Q.vt`/`.Q.vp`, which are used inside `.Q.p1` during the partitioned select `.Q.ps`.
(Since V2.8 2012.01.20, modified  V3.0 2012.01.26)

If your table exists at least in the latest partition (so there is a prototype for the schema), you could use `.Q.bv[]` to create empty tables on the fly at run-time without having to create those empties on disk. ``.Q.bv[`]`` (with argument) will use prototype from first partition instead of last. (Since V3.2 2014.08.22.)

Some admins prefer to see errors instead of auto-manufactured empties for missing data, which is why `.Q.bv` is not the default behaviour.
```q
q)n:100
q)t:([]time:.z.T+til n;sym:n?`2;num:n)
q).Q.dpft[`:.;;`sym;`t]each 2010.01.01+til 5
`t`t`t`t`t
q)tt:t
q).Q.dpft[`:.;;`sym;`tt]last 2010.01.01+til 5
`tt
q)\l .
q)tt
+`sym`time`num!`tt
q)@[get;"select from tt";-2@]; / error
./2010.01.01/tt/sym: No such file or directory
q).Q.bv[]
q).Q.vp
tt| +`date`sym`time`num!(`date$();`sym$();`time$();`long$())
q)@[get;"select from tt";-2@]; / no error
```


### `.Q.PD` (partition locations)

Syntax: `.Q.PD`

In partitioned DBs, returns a list of partition locations – conformant to `.Q.PV` – which represents the partition location for each partition.
(In non-segmented DBs, this will be simply `count[.Q.PV]#`:.`.)
`.Q.PV!.Q.PD` can be used to create a dictionary of partition-to-location information.
```q
q).Q.PV
2010.05.26 2010.05.27 2010.05.28 2010.05.29 2010.05.30 2010.05.30 2010.05.31
q).Q.PD
`:../segments/1`:../segments/2`:../segments/3`:../segments/4`:../segments/3`:../segments/4`:../segments/1
q).Q.PV!.Q.PD
2010.05.26| :../segments/1
2010.05.27| :../segments/2
2010.05.28| :../segments/3
2010.05.29| :../segments/4
2010.05.30| :../segments/3
2010.05.30| :../segments/4
2010.05.31| :../segments/1
```


### `.Q.pd` (modified partition locations)

Syntax: `.Q.pd`

In partitioned DBs, `.Q.PD` as modified by `.Q.view`.


### `.Q.pf` (partition type)

Syntax: ==FIXME==

In partitioned DBs, returns the partition type.
Possible values are `` `date`month`year`int``.


### `.Q.pn` (partition counts)

Syntax: `.Q.pn`

In partitioned DBs, returns a dictionary of cached partition counts – conformant to `.Q.pt`, each conformant to `.Q.pv` – as populated by `.Q.cn`.
Cleared by `.Q.view`.

`.Q.pv!flip .Q.pn` can be used to create a crosstab of table-to-partition-counts once `.Q.pn` is fully populated.
```q
q)n:100
q)t:([]time:.z.T+til n;sym:n?`2;num:n)
q).Q.dpft[`:.;;`sym;`t]each 2010.01.01+til 5
`t`t`t`t`t
q)\l .
q).Q.pn
t|
q).Q.cn t
100 100 100 100 100
q).Q.pn
t| 100 100 100 100 100
q).Q.pv!flip .Q.pn
          | t
----------| ---
2010.01.01| 100
2010.01.02| 100
2010.01.03| 100
2010.01.04| 100
2010.01.05| 100
q).Q.view 2#date
q).Q.pn
t|
q).Q.cn t
100 100
q).Q.pn
t| 100 100
q).Q.pv!flip .Q.pn
          | t
----------| ---
2010.01.01| 100
2010.01.02| 100
```


### `.Q.pt` (partitioned tables)

Syntax: `.Q.pt`

Returns a list of partitioned tables.


### `.Q.PV` (partition values)

Syntax: `.Q.PV`

In partitioned DBs, returns a list of partition values – conformant to `.Q.PD` – which represents the partition value for each partition.
(In a date-partitioned DB, unless the date has been modified by `.Q.view`, this will be simply date.)
```q
q).Q.PD
`:../segments/1`:../segments/2`:../segments/3`:../segments/4`:../segments/3`:../segments/4`:../segments/1
q).Q.PV
2010.05.26 2010.05.27 2010.05.28 2010.05.29 2010.05.30 2010.05.30 2010.05.31
q)date
2010.05.26 2010.05.27 2010.05.28 2010.05.29 2010.05.30 2010.05.30 2010.05.31
q).Q.view 2010.05.28 2010.05.29 2010.05.30
q)date
2010.05.28 2010.05.29 2010.05.30 2010.05.30
q).Q.PV
2010.05.26 2010.05.27 2010.05.28 2010.05.29 2010.05.30 2010.05.30 2010.05.31
```


### `.Q.pv` (modified partition values)

Syntax: `.Q.pv`

In partitioned DBs, `.Q.PV` as modified by `.Q.view`.


### `.Q.vp` (missing partitions)

Syntax: `.Q.vp`

In partitioned DBs, returns a dictionary of table schemas for tables with missing partitions, as populated by `.Q.bv`.
(Since V3.0 2012.01.26.)
```q
q)n:100
q)t:([]time:.z.T+til n;sym:n?`2;num:n)
q).Q.dpft[`:.;;`sym;`t]each 2010.01.01+til 5
`t`t`t`t`t
q)tt:t
q).Q.dpft[`:.;;`sym;`tt]last 2010.01.01+til 5
`tt
q)\l .
q)tt
+`sym`time`num!`tt
q)@[get;"select from tt";-2@]; / error
./2010.01.01/tt/sym: No such file or directory
q).Q.bv[]
q).Q.vp
tt| +`date`sym`time`num!(`date$();`sym$();`time$();`long$())
q)@[get;"select from tt";-2@]; / no error
```


## Segmented database state


### `.Q.D` (partitions)

Syntax: `.Q.D`

In segmented DBs, contains a list of the partitions – conformant to `.Q.P` – that are present in each segment.

`.Q.P!.Q.D` can be used to create a dictionary of partition-to-segment information.
```q
q).Q.P
`:../segments/1`:../segments/2`:../segments/3`:../segments/4
q).Q.D
2010.05.26 2010.05.31
,2010.05.27
2010.05.28 2010.05.30
2010.05.29 2010.05.30
q).Q.P!.Q.D
:../segments/1| 2010.05.26 2010.05.31
:../segments/2| ,2010.05.27
:../segments/3| 2010.05.28 2010.05.30
:../segments/4| 2010.05.29 2010.05.30
```


### `.Q.P` (segments)

Syntax: `Q.P`

In segmented DBs, returns a list of the segments (i.e. the contents of `par.txt`).
```q
q).Q.P
`:../segments/1`:../segments/2`:../segments/3`:../segments/4
```


### `.Q.u` (date based)

Syntax: `.Q.u`

- In segmented DBs, returns `1b` if each partition is uniquely found in one segment. (E.g., true if segmenting is date-based, false if name-based.)
- In partitioned DBs, returns `1b`.

