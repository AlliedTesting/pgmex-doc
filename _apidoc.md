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

### Key features

**PgMex** has the following key features:

 * Short learning curve.
 * Written 100% in C. 
 * Delivers a super-fast and reliable performance with a minimum overhead even for arrays - thanks to 100% binary data transfer between Matlab and PostgreSQL without any text parsing.
 * Linux and Windows binaries.
 * Much faster than JDBC, especially for a large data sets and a large number of calls.
 * 100% binary data transfer between Matlab and PostgreSQL, no text parsing.
 * Supports data exchange in both directions for most of PostgreSQL data types and arrays.
 * Full support for PostgreSQL NULLs for both scalars and arrays, a proper handling of Matlab +Inf/-Inf values.
 * Complex Matlab variables that cannot be mapped directly to PostgreSQL built-in type can be serialized and stored as PostgreSQL 'bytea' using PgMex built-in serialization; when extracted from PostgreSQL, such variables are automatically reconstructed (de-serialized) in Matlab.
 * Simple to use, yet powerful, comprehensive and entirely documented API.
 * We offer a fully functional evaluation version, with a few limitations to an amount of data transferred to DB, a number of extracted fields and a number of sequential calls.
 * Working and tested with any **Matlab** version starting with **2015b** up to the recently released **2018b** on both Linux and Windows platforms.
 * Compatible with the latest **PostgreSQL 10** as well as with all prior versions of PostgreSQL.
 * With every purchase, we offer six months of free updates and priority e-mail and forum support.
 * We offer single user (for one developer) and site licenses (for all developers working in the same company). There are no application royalty fees.

 ```
We'll show you sample code on this pane!
```

## API Overview

<aside >
The following documentation is also available at <a href="https://github.com/AlliedTesting/pgmex-doc">GitHub</a>, please feel free to contribute fixes to any problems or misprints you find.
</aside>

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

Exceptions: The first (component) part of **PgMex** exception identifiers is
**PgMex**, followed by the function name and error mnemonic.

# Connection Control

## connect
**connect** opens a new database connection

### INPUT:
* **connStr**: char [1,N] - connection string specified either as a keyword/value string 
(`host=localhost port=5432 dbname=mydb connect_timeout=10` for instance) or as a connection URI (see
 [LIBPQ-CONNSTRING](http://www.postgresql.org/docs/9.6/static/libpq-connect.html#LIBPQ-CONNSTRING) for more information)
         Default values are used for missing parameters.
		 
### OUTPUT:
* **dbConn**: uint32 or uint64 [1,1] - C pointer to a **PGconn** structure

```matlab
 %% Example:
 dbConn=com.allied.pgmex.pgmexec('connect','dbname=test user=postgres');
``` 
## finish
**finish** closes the connection to the server. Also frees memory used by the
 **PGconn** object.

```matlab
com.allied.pgmex.pgmexec('finish',dbConn)
```

### INPUT:
#### regular:
* **dbConn**: uint32 or uint64 [1,1] - C pointer to a **PGconn** structure

<aside class="warning">
 The **PGconn** pointer must not be used again after **PQfinish** has been called.
</aside>

## reset 
**reset** resets the communication channel to the server.

 ```matlab
 com.allied.pgmex.pgmexec('reset',dbConn)
 ```

### INPUT:
#### regular:
* **dbConn**: uint32 or uint64 [1,1] - C pointer to a **PGconn** structure

### DESCRIPTION:

 This function will close the connection to the server and attempt to
 re-establish a new connection to the same server, using all the same
 parameters previously used. This might be useful for error recovery if a
 working connection is lost.
 
# Connection Status

## db
**db** returns the database name of the connection.

 ```matlab
 dbName=com.allied.pgmex.pgmexec('db',dbConn)
 ```

### INPUT:
#### regular:
* **dbConn**: uint32 or uint64 [1,1] - C pointer to a **PGconn** structure

### OUTPUT:
* **dbName**: char [1,N] - database name


 
## user
**user** returns the user name of the connection.

 ```matlab
 userName=com.allied.pgmex.pgmexec('user',dbConn)
 ```

### INPUT:
#### regular:
* **dbConn**: uint32 or uint64 [1,1] - C pointer to a **PGconn** structure

### OUTPUT:
* **userName**: char [1,N] - user name


## pass
**pass** returns the password of the connection.

 ```matlab
 connPass=com.allied.pgmex.pgmexec('pass',dbConn)
 ```

### INPUT:
#### regular:
* **dbConn**: uint32 or uint64 [1,1] - C pointer to a **PGconn** structure

### OUTPUT:
* **password**: char [1,N] - connection password

## host
**host** returns the server host name of the connection.

 ```matlab
 hostName=com.allied.pgmex.pgmexec('host',dbConn)
 ```

### INPUT:
#### regular:
* **dbConn**: uint32 or uint64 [1,1] - C pointer to a **PGconn** structure

### OUTPUT:
* **hostName**: char [1,N] - host name


## port
**port** returns the port of the connection.

 ```matlab
 connPort=com.allied.pgmex.pgmexec('port',dbConn)
 ```

### INPUT:
#### regular:
* **dbConn**: uint32 or uint64 [1,1] - C pointer to a **PGconn** structure

### OUTPUT:
* **port**: char [1,N] - connection port


## options
**options** Returns the command-line options passed in the connection request.

 ```matlab
 connOpt=com.allied.pgmex.pgmexec('options',dbConn)
 ```

### INPUT:
#### regular:
* **dbConn**: uint32 or uint64 [1,1] - C pointer to a **PGconn** structure

### OUTPUT:
* **options**: char [1,N] - connection options


## status
**status** returns the status of the connection.

 ```matlab
 connStatus=com.allied.pgmex.pgmexec('status',dbConn)
 ```

### INPUT:
#### regular:
* **dbConn**: uint32 or uint64 [1,1] - C pointer to a **PGconn** structure

### OUTPUT:
* **connStatus**: double [1,1] - connection status code (see below)

### DESCRIPTION:

 The status can be one of a number of values. However, only two of these
 are seen outside of an asynchronous connection procedure: **CONNECTION_OK
 (0)** and **CONNECTION_BAD (1)**. A good connection to the database has the
 status **CONNECTION_OK**. A failed connection attempt is signaled by status
 **CONNECTION_BAD**. Ordinarily, an OK status will remain so until **finish** command,
 but a communications failure might result in the status changing to
 CONNECTION_BAD prematurely. In that case the application could try to
 recover by calling **reset**.


## errorMessage
**errorMessage** returns the error message most recently generated by an
 operation on the connection.

 ```matlab
 errMsg=com.allied.pgmex.pgmexec('errorMessage',dbConn)
 ```

### INPUT:
#### regular:
* **dbConn**: uint32 or uint64 [1,1] - C pointer to a **PGconn** structure

### OUTPUT:
* **errMsg**: char [1,N] - latest error message

 Most failing **PgMex** commands throw exceptions. This command is left for
 backward compatibility.
 
# Working With Parameters

## paramCreate
**paramCreate** creates a new **PGparam** object for storing command parameters.

```matlab
pgParam=com.allied.pgmex.pgmexec('paramCreate',dbConn)
```

### INPUT:
#### regular:
* **dbConn**: uint32 or uint64 [1,1] - C pointer to a **PGconn** structure

### OUTPUT:
* **pgParam**: uint32 or uint64 [1,1] - C pointer to a **PGparam** structure


## paramCount
**paramCount** returns the number of parameters in a **PGparam** object.

```matlab
nParams=com.allied.pgmex.pgmexec('paramCount',pgParam)
```

### INPUT:
#### regular:
* **pgParam**: uint32 or uint64 [1,1] - C pointer to a **PGparam** structure

### OUTPUT:
* **nParams**: int32 [1,1] - the number of parameters in **pgParam**


## paramReset
**paramReset** clears out any previously put parameters.

```matlab
com.allied.pgmex.pgmexec('paramReset',pgParam)
```

### INPUT:
#### regular:
* **pgParam**: uint32 or uint64 [1,1] - C pointer to a **PGparam** structure

### DESCRIPTION:
 This function only resets the parameter counter, but does not free any
 memory. It is useful for reusing the **PGparam** object. You must still call
 **paramclear** in the end.


## paramClear
**paramclear** releases all resources being used by a **PGparam** object.

```matlab
pgmexec('paramClear',pgParam)
```

### INPUT:
#### regular:
* **pgParam**: uint32 or uint64 [1,1] - C pointer to a **PGparam** structure 
<aside class="warning">The object should not be used after a clear.</aside>


## putf
**putf** packs a set of parameters in a PGparam structure for use with a
 parameterized query.

>Put a date, a string, a double vector and a double matrix:

```matlab
pgmexec('putf',pgParam ,'%date %varchar %float8[]@ %float8[]',...
	datenum('2015-03-26'),'description',[1;2;3;4],[0.1, 0.2; 0.3, 0.4]);
```

>Put an integer matrix with one NULL cell, a NULL value of a boolean
>  array type (null::bool[]) and a NULL value of a varchar type
>  (null::varchar):

```matlab
SArray=struct('valueMat',[1, 2; 0, 4],'isNullMat',logical([0, 0; 1 0]));
SNull=struct('valueMat',[],'isValueNull',true);
pgmexec('putf',pgParam ,'%int4[] %bool[] %varchar',SArray,SNull,SNull);
```

>Put a string array, a serialized Matlab object (bytea), and an array
>  of serialized Matlab objects (bytea[]):

```matlab
pgmexec('putf',pgParam ,'%text[] %serbytea %serbytea[]',...
    {'foo','bar';'baz','qux'},struct('arr',[1 2 3]),...
    {[10 20 30],'abcd',@sin});
%
pgmexec('paramClear',pgParam );
```
>when extracted via ['getf'](#getf) command with 'serbytea' field specification, 
>struct('arr',[1 2 3]) is automatically de-serialized into Matlab structure
>the same goes for cell elements provided as values for '%serbytea[]' field specification



### INPUT:
#### regular:
* **pgParam**: uint32 or uint64 [1,1] - C pointer to a **PGparam** structure
* **fieldSpecStr**: char - specification for types of fields
* **SFeldVal1**: matrix of some type or struct[1,1] - values of first
         field to set
* ...
* **SFieldValN**: matrix of some type or struct[1,1] - values of N-th field
         to set


### OUTPUT:
none.

### DESCRIPTION


**SFieldVal1**,...,**SFieldValN** inputs are expected to be formed according to the following rules		 

* If some of **SFieldVal1**,...,**SFieldValN** are structures that it is
     assumed that they have the following fields:
     *    valueMat: matrix of some type;
     *    isNullMat: logical matrix.
     
	 Both **valueMat** and **isNullMat** are of the same size (excluding the case
     when **valueMat** is a plain vector, in this case **isNullMat** is scalar logical),
     **valueMat** gives values for the corresponding field while **isNullMat**
     determines what values are NULL. If some **SFieldValK**, K=1,...,N, is not
     a structure, then it is supposed that all elements of a matrix that
     is value for K-th field are not-NULL and fieldValK gives immediately
     this value.
* If SFieldValK is a structure, it may have a field
     *    **isValueNull**: logical [1,1]
     When **isValueNull** is true, the values of the other fields in the
     structure are ignored. The entire field value, be it a scalar or an
     array, is treated as NULL.
* When the format specifies a numeric type other than **float8 (double)**,
     the input can still be of type **double**, as long as it can be cast
     directly into the specified type. For instance, if the format
     specifies **int4**, the input can be an integer of type **double**, such
     as 5.
* **pgParam** must be initialized before use; this can be done with a
     call `pgParam=com.allied.pgmex.pgmexec('paramCreate',dbConn)`. **pgParam** does not have to
     be empty; subsequent calls to **putf** will add more values to those
     already there.


# Command Execution
 
## exec
**exec** submits a command to the server and waits for the result.

```matlab
pgResult=com.allied.pgmex.pgmexec('exec',dbConn,command)
```

### INPUT:
#### regular:
* **dbConn**: uint32 or uint64 [1,1] - C pointer to a **PGconn** structure
     command: char [1,N] - SQL command to be executed

### OUTPUT:
* **pgResult**: uint32 or uint64 [1,1] - C pointer to a **PGresult** structure

### DESCRIPTION:
* The command can contain only a single SQL statement (nested
     function calls or complex SELECT statements are admissible).
* Server errors (e.g. due to malformed SQL) will trigger exceptions.
* If no output is requested, the result will be disposed of
     automatically. Otherwise, the result needs to be cleared using the
     **clear** command to avoid memory leaks (even if no tuples were
     returned.)

## paramExec
**paramExec** submits a parameterized command to the server and waits for the
 result.

```matlab
pgResult=com.allied.pgmex.pgmexec('paramExec',dbConn,pgParam,commandStr)
```

### INPUT:
#### regular:
* **dbConn**: uint32 or uint64 [1,1] - database connection, a C pointer
         to a **PGconn** structure
* **pgParam**: uint32 or uint64 [1,1] - C pointer to a PGparam structure
         that holds query parameters.
* **commandStr**: char [1,N] - SQL command to be executed

### OUTPUT:
* **pgResult**: uint32 or uint64 [1,1] - query result, a C pointer to a
         PGresult structure

### DESCRIPTION:

* Parameters can be packed into **pgParam** with the **putf** function.
     If the query does not reference any parameters, a NULL pointer
     uint32(0) or uint64(0) can be used in place of **pgParam**; this is
     equivalent to the **exec** command.
* The command can contain only a single SQL statement (nested
     function calls or complex SELECT statements are admissible).
* Server errors (e.g. due to malformed SQL) will trigger exceptions.
* If no output is requested, the result will be disposed of
     automatically. Otherwise, the result needs to be cleared using the
     **clear** command to avoid memory leaks (even if no tuples were
     returned.)

```matlab
import com.allied.pgmex.pgmexec;
pgParam=pgmexec('paramCreate',dbConn);
pgmexec('putf',pgParam,'%int4 %name',10,'foo');
res=pgmexec('paramExec',dbConn,pgParam,...
    'select * from my_table where ind > $1 and code = $2');
pgmexec('paramClear',pgParam);
% do something with result...
pgmexec('clear',res);
```


## batchParamExec
**batchParamExec** creates a prepared statement for a given command and executes this statement
within batch mode on the server for combinations of parameters values.

>Example 1: Insert 100000 tuples, each with an integer, a boolean, a string and an array 1x2 of dates

```matlab
import com.allied.pgmex.pgmexec;
SData=struct();
SData.mytupleno=transpose(int32(1:100000));
SData.myticker=strcat('foo',cellfun(@num2str,num2cell(transpose(1:100000)),...
    'UniformOutput',false);
startDateVec=datenum('1-Jan-2000')+transpose(0:99999);
SData.mydate_span=[startDateVec startDateVec+30];
SData.ismytuple=randi([0 1],100000,1); % double, but convertible to bool
pgmexec('batchParamExec',dbConn,'insert into my_table1 values ($1,$2,$3,$4)',...
    '%int4 %name %date[] %bool',SData);
```

>Example 2: Insert 100000 tuples, each with a column array of strings, a serialized function_handle and a double array, prefix for prepared statement is defined

```matlab
SData=struct();
SData.mystr_arr=repmat({{'a','b','c'};{'d'};{'e','f'};{}},25000,1);
SData.myfunc_list=repmat({@sin;@cos},50000,1);
SData.mymeas_mat=repmat({[1 2];[];[1 3;2 4];0.5;[NaN;Inf]},20000,1);
% insert mystr_arr as column array due to '@' postfix
pgmexec('batchParamExec',dbConn,'insert into my_table2 values ($1,$2,$3)',...
    '%varchar[]@ %serbytea %float8[]',SData,[],[],'my_bpe_prefix');
```

>Example 3: Insert 100000 tuples, each with an integer and a string, some elements of values are NULL

```matlab
SData=struct();
SData.myind=randi([1 10],100000,1); % convertible to int4
tickerCVec={'aaa';'bb';'cccc'};
SData.myticker=tickerCVec(randi([1 3],100000,1));
SIsNull=struct();
SIsNull.myind=repmat([true;false;false;true],25000,1);
SIsNull.myticker=logical(randi([0 1],100000,1));
pgmexec('batchParamExec',dbConn,'insert into my_table3 values ($1,$2)',...
    '%int4 %name',SData,SIsNull);
```

>Example 4: Insert 100000 tuples, each with an integer, a string of length 3, a numeric matrix of arbitrary size and a numeric array 1x2x2, some values are NULL as a whole

```matlab
SData=struct();
SData.mytupleno=int64(transpose(1:100000));
currMat=['USD';'EUR';'RUB'];
SData.mycurr=currMat(randi([1 3],100000,1),:);
SData.mymeas_mat=repmat({[1 2];[];[1 3;2 4];0.5;[NaN;Inf]},20000,1);
numMat=cat(1,reshape([1 NaN 0.5 -1],1,2,2),reshape([-Inf 0.25 3 Inf],1,2,2),reshape([NaN 0 3 2.5],1,2,2));
SData.mynum_mat=numMat(randi([1 3],100000,1),:,:);
SIsValueNull=struct();
SIsValueNull.mytupleno=false(100000,1);
SIsValueNull.mycurr=repmat([true;false;true;true;false],20000,1);
SIsValueNull.mymeas_mat=logical(randi([0 1],100000,1));
SIsValueNull.mynum_mat=repmat([true;false],50000,1);
pgmexec('batchParamExec',dbConn,'insert into my_table4 values ($1,$2,$3,$4)',...
    '%int8 %varchar %float4[] %float8[]',SData,[],SIsValueNull);
```

### INPUT:
#### regular:
* **dbConn**: uint32 or uint64 [1,1] - database connection, a C pointer
         to a **PGconn** structure
* **commandStr**: char [1,N] - SQL command to be executed
* **fieldSpecStr**: char - specification for types of fields
* **SData**: struct [1,1] - structure that contains data for all fields
     of types given in **fieldSpecStr** for all combinations of these fields values.
     It is assumed that the order of fields in **SData** corresponds to their order
     in **fieldSpecStr** (for other assumptions see 
     [RULES](#rules-for-data-and-info-on-nulls) below)
<aside class="warning">
The names of fields in **SData** are not important. For example, if INSERT is performed,
the names of fields in **SData** may be different from those of the affected table.
</aside>

#### optional:
* **SIsNull**: empty or struct [1,1] - if non-empty, then structure that allows
     to determine for each value of each field which elements of this value are NULL
     (for detailed interpretation of this input as well as for other assumptions see
     [RULES](#rules-for-data-and-info-on-nulls) below)
* **SIsValueNull**: empty or struct [1,1] - if non-empty, then structure that allows
     to determine for each value of each field whether this value is NULL or not
     (considering the value as a whole, for detailed interpretation of this input and
     for other assumptions see [RULES](#rules-for-data-and-info-on-nulls) below)
* **stmtPrefixName**: char [1,] - if given, then non-empty string determining prefix
     for the name of prepared statement, if not given, assumed to be equal to
     'batchparamexec'

### OUTPUT:
none.

### DESCRIPTION

#### WHAT DOES THIS COMMAND DO AND HOW

In essence, **batchParamExec** does the same as several sequential calls of **putf** and **paramExec**
do (but without creation of **pgParam** through **paramCreate**). The differences are that
**batchParamExec** 

* is an optimized version of this procedure executed in the so-called batch mode (queries are sent
to the server in the asynchronous mode not waiting for the response from the server);
* does not return results.

This may be useful for such commands as INSERT, UPDATE or DELETE, or if some stored procedure
not returning results are executed on the server for some sufficiently
large number of parameters values combinations.

#### GENERAL REMARKS

* The command can contain only a single SQL statement (nested function calls or complex SELECT statements are admissible).
<aside class="warning">
Server errors (e.g. due to malformed SQL) will trigger exceptions.
</aside>

* **fieldSpecStr** is formed by the same way it is done for [**putf**](#putf).

* The name of prepared statement is obtained as the value of **stmtPrefixName** concatenated
     by means of '\_' with the value of a special counter that changes after each execution of
     **batchParamExec**. For example, if **stmtPrefixName** is not given (default value is used),
     the name of prepared statement is **batchparamexec\_xxxxxx** with the current value
     of the counter instead of **xxxxxx**.

          At the end of its execution **batchParamExec** automatically deallocates the created prepared
     statement. This naming is done this way both to exclude conflicts
     between calls of **batchParamExec** and to avoid accidental coincidence with the names
     of prepared statements created by user. Thus, it makes sense to pass **stmtPrefixName**
     only in the case the latter situation is be excluded. Look at Example 2 to see how this input
     is to be passed.

#### RULES FOR DATA AND INFO ON NULLS

*  All the fields in **SData** should have the same size along the first dimension. The latter size
     determines the number of iterations the created prepared statement executes on the server. And
     each of these executions is parameterized by their own combination of fields values. These values
     are the corresponding slices of all fields in **SData** for each fixed value of the first index.

* If for some field its type in **fieldSpecStr** is not an array and is of fixed size (i.e. is not represented
     in Matlab as a string and is not of **bytea**, **serbytea** or **serbytea_wn** types), the values of this
     field in **SData** should be given as a column vector either of numeric or of logical type (the latter
     if the type is **bool**). In this case the values of the corresponding parameters for each execution
     of the command are scalar (each element of the corresponding column vector). 
     <aside class="notice">
     The fields **mytupleno** and **ismytuple** of Example 1 above fit this rule.
     </aside>

>Example 5: Prepare data for tuples, each with a char and a string of length 3

```matlab
callPutVec=['C';'P'];
SData.mycall_put=callPutVec(randi([1 2],100000,1)); % use %bpchar in fieldSpecStr
currMat=['USD';'EUR';'RUB'];
SData.mycurr=currMat(randi([1 3],100000,1),:); % use %varchar in fieldSpecStr
```

>Example 6: Prepare data for tuples, each with a numeric array 1x2x2

```matlab
measMat=cat(1,reshape([1 NaN 0.5 -1],1,2,2),...
    reshape([-Inf 0.25 3 Inf],1,2,2),...
    reshape([NaN 0 3 2.5],1,2,2));
SData.mymeas_mat=measMat(...
    randi([1 3],100000,1),:,:); % use %float4[] or %float8[] in fieldSpecStr
```

* If the type of the field specified in **fieldSpecStr** is either an array or is of non-fixed size,
     three situations are possible. 

     - In the first simple case the data may be represented in **SData**
     by cell array in the form of column vector such that each cell contains a value of the parameter
     for the respective execution iteration.
     <aside class="warning">
     The way just described above is obligatory in the case of **serbytea** and **serbytea_wn** as well as arrays of these types.
     </aside>
     <aside class="notice">
     The field **myticker** of Example 1 and all the fields of Example 2 above illustrate this rule.
     </aside>

     - The next situation occurs when the values are represented in **SData** as a two-dimensional matrix (of numeric
     or char type), but these values are supposed to be scalars of non-fixed size
     (**bytea** or represented by a string, for instance, **varchar**, **text**, **xml**, etc.).
     In this case the rows of this matrix represent each of the mentioned values for the parameter. For example,
     if the type is **varchar**, this way to pass the values of the parameter is possible if these values are strings of
     equal length, so that these strings can be stacked up one above the other.
     <aside class="notice">
     Example 5 above illustrates how **SData** is to be filled in this case.
     </aside>

     - The third case occurs when the values are given in **SData** as a multidimensional Matlab array (of numeric,
     logical or char type) in the case the parameter values are of fixed size and of array type. The slices of the field in **SData**
     corresponding to each fixed value of the first index are these values for each particular execution iteration.
     And again it is possible only in some special case. Namely, if the values of the parameter for each execution iteration
     are arrays of the same size along all dimensions, at that their size along first dimension equals 1. Then
     these arrays are stacked up one above the other to form the value of the corresponding field of **SData** uniting
     the values of the parameter for all the iterations of statement execution.
     <aside class="notice">
     Example 6 above shows how **SData** is to be formed to fit this rule.
     </aside>


* When the format specifies a numeric type other than **float8 (double)**,
     the values of the corresponding field in **SData** (or the values of each cell if
     this field in **SData** is a cell array, see the previous item for details when this may be the case)
     can still be of type **double**, as long as this can be cast directly into the specified type. For instance, if
     the format specifies **int4**, the corresponding value can be an integer of type **double**, such
     as 5.
     <aside class="notice">
    The field **ismytuple** of Example 1 and the field **myind** of Example 3 above illustrate this rule.
    </aside>

* If either **SIsNull** or **SIsValueNull** is non-empty, then this input is assumed to be a scalar structure with
     exactly the same fields (and having the same order within the corresponding structure) as in **SData**.
     Besides, all the fields should have the same size long first dimension as those of **SData**.

* If **SIsNull** or **SIsValueNull** are given as scalar structures, all the fields in **SIsNull** are expected to be
     either logical matrices or cell vector of logical matrices, while all the fields of **SIsValueNull** are expected
     to be plain logical column vectors.

>Example 7: Prepare data and info on NULLs for tuples, each with a string array and an array of numeric matrices

```matlab
SData=struct();
SData.myticker_vec=repmat(...
    {{'aaa','aa'};{'bbbb','bbbbb',''}},...
    50000,1); % use %varchar[] in fieldSpecStr
SData.mymeas_mat=repmat(...
    {[1 2];[1 3;2 4];0.5;[NaN;Inf]},...
    25000,1); % use %float4[] or %float8[] in fieldSpecStr
SIsNull=struct();
SIsNull.myticker_vec=repmat({[false true];false;[false true false]},50000,1);
SIsNull.mymeas_mat=repmat({[true false];false(2,2);true;[false;true]},25000,1);
``` 

>Example 8: Prepare data and info on NULLs for tuples, each with a numeric array 1x2x2

```matlab
SData=struct();
measMat=cat(1,reshape([1 NaN 0.5 -1],1,2,2),...
    reshape([-Inf 0.25 3 Inf],1,2,2),...
    reshape([NaN 0 3 2.5],1,2,2));
SData.mymeas_mat=measMat(...
    randi([1 3],100000,1),:,:); % use %float4[] or %float8[] in fieldSpecStr
SIsNull=struct();
SIsNull.mymeas_mat=logical(randi([0 1],100000,2,2));
``` 

* If **SIsNull** is non-empty, the values of the fields in **SIsNull** should correspond to those in **SData**
     so that the values of each field in **SIsNull** determine what values of the same field in **SData** are NULL.

     - In particular, if some parameter is supposed to have scalar value (i.e. not an array) for each particular
     iteration of statement execution, the respective field in **SIsNull** should be logical column vector of the length
     equal to all the sizes of the fields in **SData** along first dimension. Then if some element of this logical vector is true,
     the corresponding value of the field in **SData** with the same first index is NULL.
     <aside class="notice">
     Example 3 above illustrates how **SIsNull** should be formed for the case of scalar values.
     </aside>

     - Further below in this subitem and the next one we consider only the case when the values of the parameter are assumed
     to be arrays. Here two situations are possible (mirroring the first and the last situations for fields of **SData**, see above for details).
     If some field of **SData** is a column cell array (with each cell containing an array with the values of the parameter
     for the corresponding execution iteration). We stress that namely array types are here under consideration,
     the case of scalars of non-fixed sizes (such as strings, that also may be represented by cell arrays in **SData**)
     is described above and does not relate to given situation. So, in this case the field of **SIsNull**
     should be also a column cell array of the same length as that in **SData**. And the contents of each cell
     for the field in **SIsNull** should be a logical matrix of the same size as the contents of the respecive cell
     of the field in **SData**. Thus, elements of each cell for the former cell array point which values of each cell for
     the latter cell array are NULL.
     <aside class="notice">
     Example 7 above shows all the fields of **SIsNull** in this situation.
     </aside>

     - At last, if the field of **SData** is a multidimensional Matlab array (of numeric, logical or char type), then
     this field in **SIsNull** should be a logical multidimensional Matlab array of the same size as of the
     field in **SData**. Then if some element of the logical array in **SIsNull** is true,
     the corresponding value of the field in **SData** at the same position is NULL.
     <aside class="notice">
     Example 8 above illustrates this case for **SIsNull**.
     </aside>

* If **SIsValueNull** is non-empty, the values of the fields in **SIsValueNull** allow to set NULL for the whole values
     of the respective parameter for some execution iterations. Namely, for all the indices for which some
     field in **SIsValueNull** is true, the corresponding parameter of the executed command equals to NULL as
     a whole, be it a scalar or an array (in contrast to the values of the fields in **SIsNull**, the latter ones
     in the case of array values can determine which separate elements of each array under consideration are NULL,
     see above for details). When some element of some field in **SIsValueNull** is true, the values of this field
     in **SData** and **SIsNull** (if this is non-empty) with the same value of the first index are ignored.
     <aside class="notice">
     Example 4 above illustrates how **SIsValueNull** is to be formed.
     </aside>

<aside class="warning">
If some parameter is a scalar or an array of **serbytea** type, it is forbidden to set the values of
the corrsponding field in **SIsNull** or **SIsValueNull** to true. If it is necessary to determine NULL values
for serialized data, the types connected to **serbytea_wn** should be used instead.
</aside>


## pqExec
**pqExec** submits a command to the server and waits for the result (**libpq**
 version).

```matlab
pgResult=com.allied.pgmex.pgmexec('pqExec',dbConn,command)
```

### INPUT:
#### regular:
* **dbConn**: uint32 or uint64 [1,1] - C pointer to a **PGconn** structure
* **command**: char [1,N] - SQL command to be executed

### OUTPUT:
* **pgResult**: uint32 or uint64 [1,1] - C pointer to a PGresult structure


### DESCRIPTION:
* This function returns text results, which makes it unsuitable for
     some **getf** requests. It is left for backwards compatibility.
* Unlike **exec** and **paramexec**, the command can contain multiple SQL
     statements separated by semicolons.


## resultStatus
**resultStatus** returns the result status of the command.

```matlab
execStatus=com.allied.pgmex.pgmexec('resultStatus',pgResult)
```

### INPUT:
#### regular:
* **pgResult**: uint32 or uint64 [1,1] - C pointer to a **PGresult** structure

### OUTPUT:
* **execStatus**: double [1,1] - result status code in a range from 0 to 9 (see table below)
		 
|Code value|Code name          | Code description                                                                                              | 
|----------|-------------------|---------------------------------------------------------------------------------------------------------------|
|0         |EMPTY_QUERY        | empty query string was executed                                                                               |
|1         |COMMAND_OK         | a query command that doesn't return anything was executed properly by the backend                             |  
|2         |TUPLES_OK          | a query command that returns tuples was executed properly by the backend, PGresult contains the result tuples |
|3         |COPY_OUT           | Copy Out data transfer in progress																			   |
|4         |COPY_IN            | Copy In data transfer in progress 																			   |
|5         |BAD_RESPONSE       | an unexpected response was recv'd from the backend 														   |
|6         |NONFATAL_ERROR     | notice or warning message 																					   |
|7         |FATAL_ERROR        | query failed 																								   |
|8         |COPY_BOTH          | Copy In/Out data transfer in progress																	       |
|9         |SINGLE_TUPLE       |single tuple from larger resultset 																    		   |
	 


## resultErrorMessage 
**resultErrorMessage** returns the error message associated with the command or an empty string if there was no error.

```matlab
errMsg=pgmexec('resultErrorMessage',pgResult)
```

### INPUT:
#### regular:
* **pgResult**: uint32 or uint64 [1,1] - C pointer to a **PGresult** structure

### OUTPUT:
* **errMsg**: char [1,N] - error message

 Normally, execution commands such as **exec** throw an exception on errors.
 This command is left for backward compatibility.


## clear
**clear** frees the storage associated with a **PGresult**.

```matlab
com.allied.pgmex.pgmexec('clear',pgResult)
```

### INPUT:
#### regular:
* **pgResult**: uint32 or uint64 [1,1] - C pointer to a **PGresult** structure

 Every command result should be freed via **clear** when it is no longer
 needed.
 
# Retrieving Query Result

## nTuples
**nTuples** returns the number of rows (tuples) in the query result.

```matlab
nTuples=com.allied.pgmex.pgmexec('nTuples',pgResult)
```

### INPUT:
#### regular:
* **pgResult**: uint32 or uint64 [1,1] - C pointer to a **PGresult** structure


### OUTPUT:
* **nTuples**: double [1,1] - the number of tuples

<aside class="warning">
 Because the underlying libpq function returns a 32-bit integer result,
 large result sets might overflow the return value.
</aside>

## nFields
**nFields** returns the number of columns (fields) in each row of the query
 result.

```matlab
nFields=com.allied.pgmex.pgmexec('nFields',pgResult)
```
### INPUT:
#### regular:
* **pgResult**: uint32 or uint64 [1,1] - C pointer to a **PGresult** structure

### OUTPUT:
* **nFields**: double [1,1] - the number of fields


## fName 
**fName** returns the column name associated with the given column number.

```matlab
fName=com.allied.pgmex.pgmexec('fName',pgResult,colInd)
```
### INPUT:
#### regular:
* **pgResult**: uint32 or uint64 [1,1] - C pointer to a **PGresult** structure
     colInd: double [1,1] - column number (numbers start at 0)

### OUTPUT:
* **fName**: char [1,N] - column name. An empty string is returned if the
         column number is out of range.


## fNumber
**fNumber** returns the column number associated with the given column name.

```matlab
fnum=com.allied.pgmex.pgmexec('fNumber',pgResult,colName)
```
### INPUT:
#### regular:
* **pgResult**: uint32 or uint64 [1,1] - C pointer to a **PGresult** structure
* **colName**: char [1,N] - column name
<aside class="notice">
 The given name is treated like an identifier in an SQL command, that is,
 it is downcased unless double-quoted.
</aside> 

### OUTPUT:
* **fnum**: double [1,1] - column number. -1 is returned if the given name
         does not match any column.


```matlab
pgResult=com.allied.pgmex.pgmexec('exec',dbConn,'SELECT 1 AS FOO, 2 AS "BAR"');
%
name=com.allied.pgmex.pgmexec('fName',pgResult,0)             % foo
name=com.allied.pgmex.pgmexec('fName',pgResult,1)             % BAR
ind=com.allied.pgmex.pgmexec('fNumber',pgResult,'FOO')        % 0
ind=com.allied.pgmex.pgmexec('fNumber',pgResult,'foo')        % 0
ind=com.allied.pgmex.pgmexec('fNumber',pgResult,'BAR')        % -1
ind=com.allied.pgmex.pgmexec('fNumber',pgResult,'"BAR"')      % 1
```

## fSize 
**fSize** returns the size in bytes of the column associated with the given
 column number.

```matlab
fSize=com.allied.pgmex.pgmexec('fSize',pgResult,colInd)
```

### INPUT:
#### regular:
* **pgResult**: uint32 or uint64 [1,1] - C pointer to a **PGresult** structure
* **colInd**: double [1,1] - column number (numbers start at 0)

### OUTPUT:
* **fSize**: double [1,1] - the space allocated for this column in a
         database row. A negative value indicates the data type is
         variable-length.

### DESCRIPTION:
 This function returns the size of the server's internal representation of
 the data type. Accordingly, it is not really very useful to end-users.

## binaryTuples 
**binaryTuples** returns true if the **PGresult** contains binary data and false if it
 contains text data.

```matlab
isBinary=com.allied.pgmex.pgmexec('binaryTuples',pgResult)
```
### INPUT:
#### regular:
* **pgResult**: uint32 or uint64 [1,1] - C pointer to a **PGresult** structure

### OUTPUT:
* **isBinary**: logical[1,1] - true only if all columns of the result are
         binary, otherwise false.


## getValue
**getValue** returns a single field value of one row of a **PGresult**.
<aside class="warning">
 This function is deprecated: use **getf** instead.
</aside>

```matlab
valueStr=com.allied.pgmex.pgmexec('getValue',pgResult,rowInd,colInd)
```

### INPUT:
#### regular:
* **pgResult**: uint32 or uint64 [1,1] - C pointer to a **PGresult** structure
* **rowInd**: double [1,1] - row number (starts with 0)
* **colInd**: double [1,1] - column number (starts with 0)

### OUTPUT:
* **value**: double or char [M,N] - a numeric or a string representation of
         the field value.

### DESCRIPTION:		 
* Only text results can be handled.
* Numeric values are converted to double and returned as sclars or arrays
 of matching dimensions (up to two dimensions). The following numeric
 types are supported: smallint, integer, bigint, real, and double
 precision.

* A 1D array of *text* is returned as a NULL-padded 2D string array.

* All other results are returned in their string representation, as
 returned by PQgetValue
 [LIBPQ-PQGETVALUE](http://www.postgresql.org/docs/9.6/static/libpq-exec.html#LIBPQ-PQGETVALUE)


## getIsNull
**getIsNull** tests a field for a NULL value.

```matlab
isNull=com.allied.pgmex.pgmexec('getIsNull',pgResult,rowInd,colInd)
```

### INPUT:
#### regular:
* **pgResult**: uint32 or uint64 [1,1] - C pointer to a **PGresult** structure
* **rowInd**: double [1,1] - row number (starts with 0)
* **colInd**: double [1,1] - column number (starts with 0)

### OUTPUT:
* **isBinary**: logical [1,1] - true if the field is NULL and false if it contains a non-NULL value.

## getLength
**getLength** returns the actual length of a field value in bytes.

```matlab
length=com.allied.pgmex.pgmexec('getLength',pgResult,rowInd,colInd)
```

### INPUT:
#### regular:
* **pgResult**: uint32 or uint64 [1,1] - C pointer to a **PGresult** structure
* **rowInd**: double [1,1] - row number (starts with 0)
* **colInd**: double [1,1] - column number (starts with 0)

### OUTPUT:
* **length**: double [1,1] - the actual data length for the particular data
         value.
		 
### DESCRIPTION:
 This function returns the length of the data pointed by **PQgetValue** (but
 not necessarily by the MEX functions **getValue** or **getf**). It is not likely
 to be useful to end users and is left for backward-compatibility.


## getf
**getf** gets one or more field values from **PGresult** using a *scanf*-style
 interface.

> Select all data (i.e. select *) from a table with 2 tuples and the following columns:
>
>`( event_id bigint, t_date date, metric_name character(1), metric_value double precision[])`
>

```matlab
 import com.allied.pgmex.pgmexec;
 [SId,SDate,SName,SValue]=pgmexec('getf',pgResult,'%int8 #date %varchar %float8[]',...
     0,'t_date',1,2);
% SId.valueVec={int64(5645334);int64(645647)}
% SId.isNullVec=[false;false]
% SId.isValueNullVec=[false;false]
%
% SDate = struct with fields:
%          valueVec: [2x1 double]
%         isNullVec: [2x1 logical]
%    isValueNullVec: [2x1 logical]
%
dateVec=SDate.valueVec
%
% dateVec =[736666;736667]
%
% SName = struct with fields:
%           valueVec: {2x1 cell}
%          isNullVec: [2x1 logical]
%     isValueNullVec: [2x1 logical]
%
nameCVec=SName.valueVec
% nameCVec ={'Experiment #1','Experiment #2'}
%
% empty value for the second extracted metric:
% SValue.valueVec={[0 2.4;-2 Inf;0 NaN];[]}
% some elements in the value matrix for the first metric are null:
% SValue.isNullVec={[true false;false false;true false];logical.empty(0,0)}
% the value extracted for the second metrics is empty because it is NULL:
% SValue.isValueNullVec=[false;true]

```
>Note that in this example in the first two output structures the 'valueVec' field is a
>column vector (int64 and double respectively), while in the last two
>structures 'valueVec' is a cell array. 'isNullVec' is a logical column
>vector in the first three structures and a cell array of logical arrays
>in the last structure.


### INPUT:
#### regular:
* **pgResult**: uint32 or uint64 [1,1] - query result, a C pointer to a
 **PGresult** structure

* **fieldSpecStr**: char - specification for types of fields
* **fieldNum1**: double [1,1] or char - field number or field name of first
            field to get
<aside class="notice">Field numeration starts with zero (0)!</aside>

* ...
* **fieldNumN**: double [1,1] or char - field number or field name of N-th
            field to get

#### properties:
* **isUniformOutput**: logical [1,1] - if this parameter is true (default),
         then this function returns values of scalar fields concatenated
         into an array instead of cell array wherever possible.


### OUTPUT:
* **SFieldValue1**: struct [1,1] - values of first field to get (for the
         format of this structure see notes below)
* ...
* **SFieldValueN**: struct [1,1] - values of N-th field to get (for the
         format of this structure see notes below)


### DESCRIPTION:

The structures SFieldValue1,...,SFieldValueN have the following fields:
 
* **valueVec**: any[nTuples,1] - vector of some type with values extracted from db;
* **isNullVec**: any[nTuples,1] - vector of some type, serves as an indicator of value elements in **vectorValue** being NULL;
* **isValueNullVec**: logical[nTuples,1] - indicates if values for some tuples are NULL.


If **isUniformOutput** is false, then in any case both **valueVec** and
     **isNullVec** are cell[nTuples,1], such that elements of **valueVec**
     contain values of corresponding fields while cells of **isNullVec** are
     logical matrices determinining whether these values are NULL or not.
     If **isUniformOutput** is true, then the format described above for both
     **valueVec** and **isNullVec** is valid only for fields with non-scalar
     values, for fields with scalar values **valueVec** is plain (not cell)
     array of size [nTuples,1] of these values while **isNullVec** is logical[nTuples,1]. 
	 Thus, if **isUniformOutput** is true, then values of
     **valueVec** and **isNullVec** obtained in general case are concatenated in
     simple column vectors. **isValueNullVec** indicates, for each tuple,
     whether the entire value, be it a scalar or an array, is NULL.


# Retrieving Other Result Info

## cmdStatus 
**cmdStatus** returns the command status tag from the SQL command that
 generated the **PGresult**.

```matlab
statusStr=com.allied.pgmex.pgmexec('cmdStatus',pgResult)
```

### INPUT:
#### regular:
* **pgResult**: uint32 or uint64 [1,1] - C pointer to a **PGresult** structure

### OUTPUT:
* **statusStr**: char [1,N] - command status tag. Commonly this is just the name of the command, but it might include
 additional data such as the number of rows processed.


## cmdTuples
**cmdTuples** returns the number of rows affected by the SQL command.

```matlab
tuplesStr=com.allied.pgmex.pgmexec('cmdTuples',pgResult)
```

### INPUT:
#### regular:
* **pgResult**: uint32 or uint64 [1,1] - C pointer to a **PGresult** structure

### OUTPUT:
* **tuplesStr**: char [1,N] - a string containing the number of rows
         affected by the SQL statement that generated the PGresult.

<aside class="warning">
 This function can only be used following the execution of a **SELECT**,
 **CREATE TABLE AS**, **INSERT**, **UPDATE**, **DELETE**, **MOVE**, **FETCH**, or **COPY** statement,
 or an **EXECUTE** of a prepared query that contains an INSERT, UPDATE, or
 **DELETE** statement. If the command that generated the **PGresult** was anything
 else, **PQcmdTuples** returns an empty string.
</aside>

