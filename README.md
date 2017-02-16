# PgMex
```
                                                                     
________        ______  ___            
___  __ \______ ___   |/  /________  __
__  /_/ /_  __ `/_  /|_/ /_  _ \_  |/_/
_  ____/_  /_/ /_  /  / / /  __/_>  <  
/_/     _\__, / /_/  /_/  \___//_/|_|  
        /____/                         
                 
			 
                                            
```

**PgMex** is a high-performance PostgreSQL client library for Matlab that enables a Matlab-based application to communicate with PostgreSQL database in the Matlab native way by passing data in a form of matrices, multi-dimensional arrays and structures. The library is written in pure C which gives a significant performance boost for both small and data-heavy database requests. Both Windows and Linux platforms are supported.

This project hosts the documentation available at http://pgmex.alliedtesting.com. 

The docs provides in-depth information on PgMex’s features and is meant to be a guide for Matlab developers integrating their apps with PostgreSQL.

### Key features

**PgMex** has the following key features:

 * Short learning curve.
 * Written 100% in C. 
 * Delivers a super-fast and reliable performance with a minimum overhead even for arrays - thanks to 100% binary data transfer between Matlab and PostgreSQL without any text parsing.
 * Linux and Windows binaries.
 * Much faster than JDBC, especially for a large data sets and a large number of calls.
 * 100% binary data transfer between Matlab and PostgreSQL, no text parsing .
 * Supports data exchange in both directions for most of PostgreSQL data types and arrays.
 * Full support for PostgreSQL NULLs for both scalars and arrays, a proper handling of Matlab +Inf/-Inf values.
 * Complex Matlab variables that cannot be mapped directly to PostgreSQL built-in type can be serialized and stored as PostgreSQL 'bytea' using PgMex built-in serialization; when extracted from PostgreSQL, such variables are automatically reconstructed (de-serialized) in Matlab.
 * Simple to use, yet powerful, comprehensive and entirely documented API.
 * We offer a fully functional evaluation version, with a few limitations to an amount of data transferred to DB, a number of extracted fields and a number of sequential calls.
 * Working and tested with any **Matlab** version starting with **2015b** up to the recently released **2016b** on both Linux and Windows platforms.
 * Linked against the latest **PostgreSQL 9.6 libpq** and is compatible with all prior versions of PostgreSQL.
 * With every purchase, we offer six months of free updates and priority e-mail and forum support.
 * We offer single user (for one developer) and site licenses (for all developers working in the same company). There are no application royalty fees.

 ```
We'll show you sample code on this pane!
```

## API Overview
The library contains of a single package **pgmex** with a single function **pgmexec** designed for execution of various commands like connecting to a database, 
executing SQL queries on PostgreSQL server etc. The first input to **pgmexec** is always a command name with a few command-specific input parameters to follow.

```matlab
import com.allied.pgmex.pgmexec;
%% connect to db
dbConn=com.allied.pgmex.pgmexec('connect','host=<myHos> user=<myUser> port=<myPort> password=<myPassword> dbname=<myDB>')
%% create schema and table
pgmexec('exec',dbConn,'CREATE SCHEMA IF NOT EXISTS demo');
pgmexec('exec',dbConn,'DROP TABLE IF EXISTS demo.demo_table');
pgResult=pgmexec('exec',dbConn,['CREATE TABLE IF NOT EXISTS demo.demo_table ('...
    't_date date,'...
    'exp_name varchar(40),'...
    'exp_conf xml,'...
    'exp_t_vec timestamp[],'...
    'exp_col_name_vec name[],'...
    'exp_res_mat float8[])'])% pgResult=uint64(3740775088)
%% get status of execution result (all is successful)
pgmexec('resultStatus',pgResult)% ans = 1
pgmexec('cmdStatus',pgResult)% ans= 'CREATE TABLE'
%% clear result
pgmexec('clear',pgResult)
%% insert the first tuple into the table and get it as text
%% create parameters structure
pgParam=pgmexec('paramCreate',dbConn)% pgParam =uint64(3843492816)
%% put values of parameters
pgmexec('putf',pgParam,...
    '%date %varchar %xml %timestamp[]@ %name[]@ %float8[]',...
    datenum('1-Dec-2016'),'Experiment #1',...
    ['<model type="struct" ><param1 type="double" >1</param1>'...
    '<param2 type="boolean" >false</param2></model>'],...
    datenum({'0:00:00';'0:00:10';'0:01:30'}),{'res1','res2'},...
    struct('valueMat',{[NaN 2.4;-2 Inf;0 NaN]},...
    'isNullMat',{[true false;false false;true false]},...
    'isValueNull',false));
% insert the correspoding tuple into the table
pgmexec('paramExec',dbConn,pgParam,...
    'INSERT INTO demo.demo_table values ($1, $2, $3, $4, $5, $6)');
% reset parameters once again
pgmexec('paramreset',pgParam)
% put new values of parameters
pgmexec('putf',pgParam,...
    '%date %varchar %xml %timestamp[]@ %name[]@ %float8[]',...
    datenum('3-Dec-2016'),'Experiment #2',[...
	'<model type="struct" ><param1 type="double" >NaN</param1>'...
    '   <param2 type="boolean" >false</param2></model>'],...
    datenum({'0:00:40';'0:02:30'}),{'res1','res2','res3'},...
    struct('valueMat',{[]},'isNullMat',{logical([])},...
    'isValueNull',{true}));
% insert one more tupole into the table
pgmexec('paramExec',dbConn,pgParam,...
    'INSERT INTO demo.demo_table values ($1, $2, $3, $4, $5, $6)');
% clear the parameters as they are not necesssary
pgmexec('paramClear',pgParam)
%% get results as binary
pgResult=pgmexec('exec',dbConn,'SELECT * FROM  demo.demo_table')
% pgResult=uint64(3849610544)
%
%% check pgResult
pgmexec('binaryTuples',pgResult)% ans=true - binary result
pgmexec('resultStatus',pgResult) %ans=2 (TUPLES_OK)
pgmexec('cmdStatus',pgResult)% ans='SELECT 2'
pgmexec('cmdTuples',pgResult)% ans='2'
pgmexec('nFields',pgResult)% ans=6
pgmexec('nTuples',pgResult)% ans=2
%% extract results
% get vector of experiment dates
STDate=pgmexec('getf',pgResult,'#date','t_date')
% STDate = struct with fields:
%          valueVec: [2?1 double]
%         isNullVec: [2?1 logical]
%    isValueNullVec: [2?1 logical]
%
tDateVec=STDate.valueVec
%
% tDateVec =[736666;736667]
%
% get names of experiments
SExpName=pgmexec('getf',pgResult,'%varchar',1)
% SExpName = struct with fields:
%           valueVec: {2?1 cell}
%          isNullVec: [2?1 logical]
%     isValueNullVec: [2?1 logical]
%
expNameCVec=SExpName.valueVec
% expNameCVec ={'Experiment #1','Experiment #2'}
%
% get results of experiments
[SExpTVec, SExpCol, SExpRes]=pgmexec('getf',pgResult,...
    '#timestamp[]@ #name[]@ #float8[]',...
    'exp_t_vec','exp_col_name_vec','exp_res_mat')
%
% SExpTVec, SExpCol, SExpRes are structures with fields:
%           valueVec: {2?1 cell}
%          isNullVec: {2?1 cell}
%     isValueNullVec: [2?1 logical]
%
% SExpTVec.valueVec={...
%    [736330;736330.000115741;736330.001041667];
%    [736330.000462963;736330.001736111]};
%
% SExpCol.valueVec={{'res1';'res2'};{'res1';'res2';'res3'}}
% SExpCol.isNullVec={[false;false];[false;false;false]}
% SExpCol.isValueNullVec=[false;false]
%
% SExpRes.valueVec={[0 2.4;-2 Inf;0 NaN];[]}
% SExpRes.isNullVec={[true false;false false;true false];logical.empty(0,0)}
% SExpRes.isValueNullVec=[false;true]
%
%% clear results
pgmexec('clear',pgResult)
```


### Pointer arguments:

Some input and output arguments, are pointers to C structures, such as
**PGconnect**. Matlab type for these arguments depends on the architecture of the
implementation: **uint32** for 32-bit or **uint64** for 64-bit. These
pointers often provide a handle for dynamically-allocated resources that
should be freed when they are no longer needed. Free resources with
special commands, such as **clear** and **finish**.

### Field specification strings:

Commands [**putf**](#putf) and [**getf**](#getf) use field specification strings that essentially represent codecs 
for converting betwen PostgreSQL data Matlab data for specific pairs of types.
The following table covers all Postgres-Matlab type pairs supported by **PgMex**.

|PGMEX        |        Postgres        |       Matlab |
|-------------|------------------------|--------------|
|int2         |       smallint         |   int16      | 
|int4         |       integer          |   int32      | 
|int8         |       bigint           |   int64      | 
|float4       |       real             |   single     |  
|float8       |       double precision |   double     |  
|numeric      |       numeric          |   char [1,N] |     
|bool         |       boolean          |   logical    |
|varchar      |       varchar          |   char [1,N] |
|text         |       text             |   char [1,N] |
|bpchar       |       char             |   char [1,N] |
|name         |       name             |   char [1,N] |
|xml          |       xml              |   char [1,N] |
|json         |       json             |   char [1,N] |
|bytea        |       bytea            |   uint8 [1,N]|
|**serbytea**     |       bytea            |  **any type&ast;** |
|**serbytea_wn**  |       bytea            | **any type&ast;** |
|date         |       date             |   datenum    |
|time         |       time             |   datenum    |    
|timetz       |       timetz           |   datenum    |    
|timestamp    |       timestamp        |   datenum    |    
|timestamptz  |       timestamptz      |   datenum    |

If **'@'** is added at the end of a type spec, then if the original value is
not of a variable-length type, the value will be transformed to a column
vector [N,1] \(**getf**\) or a 1D array \(**putf**\), regardless of the dimensions of
the original value.

An empty array must have the dimensions [0,0] when it is given as an
argument to **putf**. When an empty (zero-dimensional) array is read from the
database, the resulting Matlab array also has the dimensions [0,0].

Variable-length inputs and outputs are string types, such as varchar, and
byte array types, such as bytea. Scalar values of variable-length types
are stored in row vectors, and arrays of variable-length types are stored
in cell arrays. For instance, a [2,2] array of varchar may be given as
`{'foo','bar';'baz','qux'}`.

**serbytea** and **serbytea_wn** type specs serve to store any Matlab variable as
a byte array. The serialization and deserialization are performed in **putf**
and **getf** respectively. The corresponding Postgres type is **bytea** for
scalars and **bytea[]** for arrays. As with other variable-length types,
arrays of variables should be packaged in cell arrays. When **serbytea_wn**
type spec is used, **putf** always expects a special structure with NULL
specifiers on input (see documentation for **putf** below). **serbytea** is used
to put "naked" values, i.e. any structure will be interpreted as a Matlab
variable to be serialized as is. For **getf** the specs **serbytea** and
**serbytea_wn** are equivalent.

Date and time inputs and outputs are Matlab datenums (doubles). Since datenums do not store time zone
information, time zone for **timetz** and **timestamptz** values is assumed to be
UTC. The schema part of the type spec for **PgMex** types is **pgmex**, whereas
the schema for standard types is **pg_catalog**. However, it is not
necessary to specify the `pgmex` schema in the field spec string.



