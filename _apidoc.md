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
>when extracted via ['getf'](#getf) command with 'serbytea' field specifiecation, 
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
     specifies **int4**, the input can be a whole number of type **double**, such
     as 5.
* **pgParam** must be initialized before use; this can be done with a
     call `pgParam=com.allied.pgmex.pgmexec('paramCreate',dbConn)`. **pgParam** does not have to
     be empty; subsequent calls to **putf** will add more values to those
     already there.
</aside>


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
</aside>

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

