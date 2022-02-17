### ***SAS Viya in the SAS Platform***

> SAS Viya is cloud enabled, allowing scalable, web-based access for your data processing needs with in-memory engine and parallel processing capabilities.SAS Viya provides integration with open-source tools. 

***SAS Viya can access a variety of data sources:***
*  SAS tables
* data that is on-premises or in the cloud
* relational databases or it can be unstructured
* XML, JSON, CSV, or XLSX.

***SAS Viya services and applications:***
* *SAS Drive* - organize and access all your SAS content
* *SAS Visual Analytics* - for web-based reporting and dashboards
* *SAS Visual Data Mining* and *Machine Learning* - to solve complex analytical problems with a visual interface that handles all tasks in the analytics life cycle
* *SAS Studio* - to write and submit code

***SAS Viya Programming Interfaces***
* *SAS Studio* is the default interface:
    * web-based application
    * writing and running code
    * access data and programs
    * stored code for common actions
    * use tasks to generate code
* *open-source applications*- Jupyter Notebook or R Studio
* *SAS 9 applications* - SAS Studio 3, SAS Enterprise Guide, or the SAS windowing environment.

***SAS Viya Servers and Processing Environments***

* *SAS Compute Server* - the SAS 9 workspace server

When this standard code is submitted in Viya it will automatically execute on the SAS Compute Server:
```
libname pvbase "&path/data";

data profit;
    set pvbase.orders;
    ...
run;

proc means data=profit;
    ...
run;
```

* *SAS Cloud Analytic Services* or *CAS* - high-performance server that performs parallel processing on in-memory data.

Use CAS for your big data and complex analytics.

Often, only very minor code modifications are required in order for programs to run in CAS.

---

***SAS Studio interface:***

1. The Start Page tab is open by default with:
    * Start a new program (in a new tab)
    * Build a flow
    * Import data 
    * Query data
    * Learn (tutorials, videos)
    * Stay connected(community)

2. The navigation pane is on the left side:
    * `Open Files` - the list of open files
    * `Explorer` -files and folders in your SAS environment (NOT on your local machine)
    * `Steps`
    * `Tasks` - procedures and generate SAS code and formatted results for you
    * `Snippets` - lines of commonly used code or text that you can save and reuse
    * `Libraries` - your SAS libraries
    * `Git Repositories` - the basic Git features 

---

***Parallel Processing in CAS***
`Client <-> Controller -- worker -- worker -- worker `

1) After the code is submitted first goes to the CAS controller. 
2) The controller distributes the data to separate worker nodes and sends each node a copy of the code. 
3) The nodes perform coordinated parallel processing on their portion of the data  executing the same action at the same time.
4) As worker nodes complete processing, they return results back to the controller in the order in which they finish. 

*In this topology, the nodes perform the work much faster than executing the code on a single processor.*

*CAS can be configured to run on a single machine, taking advantage of multiple CPUs or threads to speed up the processing. Or to run on multiple machines, with one controller node and several worker nodes.*

***Server Strengths***

All the visual interfaces in Viya use CAS behind the scenes, including:
* SAS Visual Analytics
* SAS Visual Statistics
* SAS Visual Data Mining and Machine Learning. 

> CAS procedures work directly with in-memory tables, ensuring full use of parallel processing.

`exemple:`

*Foundation PROCs*:
* FREQ
* LOGISTIC
* MIXED

*CAS PROCs*:
* FREQTAB
* LOGSELECT
* LMIXED

> Open-source languages like Java, Lua, Python, R, or REST APIs can call CAS actions to process data in CAS 

***Steps that should be modified to run in CAS:***
* steps using data sources larger than 50 G
* steps that run for 30 minutes or longer, 
* PROCs that are computationally demanding
* DATA steps with many computations, functions or conditional logic.

***Steps to execute SAS code in CAS:***

* Create a CAS session
* Create caslibs to access data
* Load in-memory tables
* Leverage CAS to accelerate processing
* Save and drop in-memory tables
* Terminate the CAS session

***SAS Libraries***

> *SAS library* - a folder or directory on your local machine that contains various file types, including SAS tables, CSV files, Microsoft Excel files, and more. In order to access the SAS tables, you associate a SAS library with the folder.

`Syntax exemple:`

```
libname library_name engine   
    "path/to/data/forlder";
```
> *Caslib* is  a container that has two main areas: 
* one for in-memory tables
* one for data source files (*ex*: SAS, CSV, Hadoop, Microsoft Excel, and databases)

***Caslib Attributes***
* *Library*
* *Source Type*
* *Path*
* *Session local* - (scope of the caslib) :
   * yes - **session scope**:
    * vissible to the CAS session where it was created
    * deleted when the cas session is terminated
   * no - **global scope**:
    * visible across CAS sessions
    * persists when the CAS session is terminated
* *Active* - 
    * **yes**
     * (current - if not specified) 
    * **no**
     * caslib available
     * calib must be specified to reference tables of files
* *Personal* - 
    * **yes**
     * global scope
     * only your user ID can access the data
     * **Casuser** - typically the active or default caslib 
    * **no**
     * all other caslibs

A different active caslib can be specified in the CAS statement with the `SESSOPTS=` and `CASLIB=` options.

To view the caslib attributes use `CASLIB caslib-name LIST;`
```
caslib casuser list;

NOTE: Session = MYSESSION Name = CASUSER(student)
    Type = PATH
    Description = Personal File System Caslib
    Path = /opt/sas/viya/config/data/cas/
           default/casuserlibraries/student/
    Definition = 
    Subdirs = Yes
    Local = No
    Active = Yes
    Personal = Yes
NOTE: Action to LIST caslib CASUSER completed 
      for session MYSESSION.
```

***Assigning a Library Reference to a Caslib***

>When a CAS session begins, Casuser doen't appear in the Libraries list `caslib ne libref`

> A libref must be assigned to a caslib using to a caslib using the CAS engine:
`LIBNAME libref CAS <CASLIB=caslib-name>;`

***NOTE:*** The libref can be the same as caslib name, or it ca be different name

1) Manually asssign:
```
libname mycas cas caslib=casuser;
    or 
libname casuser cas;
```
2)Automatically assign a matching library reference to all existing caslibs

```
caslib _all_ assign;
SAS Log
NOTE: A SAS Library associated with a caslib can only reference library 
      member names that conform to SAS Library naming conventions.
NOTE: CASLIB CASUSER(student) for session MYSESSION will be mapped to 
      SAS Library CASUSER.
NOTE: CASLIB EXEMPLE for session MYSESSION will be mapped to SAS Library EXEMPLE.
NOTE: CASLIB Public for session MYSESSION will be mapped to SAS Library PUBLIC.
```
***NOTE:*** After librefs are assigned to caslibs, the librefs are listed in the Libraries section, identified by a cloud icon. Here we see the orders table in the Casuser caslib.

***Predefined Caslibs***:
* predefined by an administrator
* global scope (Local=No)
* available to users with permission
* used for shared data sources (Personal=no)

***PROC CASUTIL Procedure***

> View the data source files that are available in a caslib:
```
PROC CASUTIL <INCASLIB="caslib-name">;
          LIST FILES | TABLES <option(s)>;
QUIT;
```

* *CASUTIL* is used to manage caslibs, data source files, and in-memory tables
* *INCASLIB=* option specifies the caslib for the procedure. If this option is omitted, the active caslib is used by default. 
* *LIST* statement lists either the data source files or in-memory tables.

***Manually Added Caslibs***
`CASLIB caslib-name PATH='/file-path/' LIBREF=libref<options>;`
* *PATH=* option specifies the location of the caslib (in other words, the folder that contains the source files)
* *LIBREF=* option maps the caslib to a library reference, using the CAS engine.
    * recommended that the libref and caslib name match
> When a new caslib is defined, it automatically becomes the new active caslib.

```
caslib XMPLE path="&path/data" libref=XMPLE;

NOTE: 'XMPLE' is now the active caslib.
NOTE: Cloud Analytic Services added the caslib 'XMPLE'.
NOTE: CASLIB XMPLE for session MYSESSION will be 
      mapped to SAS Library XMPLE.
NOTE: Action to ADD caslib XMPLE 
      completed for session MYSESSION.
```

***NOTE:*** By default, manually added caslibs are session scope, but they can be created with global scope if you are authorized to do so.

```
proc casutil;
    list files;
quit;
```
***NOTE:*** Although the files are listed, we can't access them until we load them into memory. That will be our next step.

***Setting the Default Caslib***

> When a new caslib is defined, it automatically becomes the new active caslib.

BUT you can use the CASLIB= session option in the CAS statement  to set the active caslib to the new caslib:

`cas mySession sesopts=(caslib=xmple);`

***Note***: If you don't have an active CAS session, first submit the New CAS Session snippet.

***Load in-memory tables:***

There are two sources of data files, that can be loaded to in-memory tables in a caslib:
* **server-side files** - data files that are already included in a defined caslib
* **client-side files** - files of any kind that are not mapped to the caslib that you can access through the SAS Compute Server: *database tables*, *SAS tables*, *text files* 

**NOTE:** Sas can load client-side or server-side files directly to in-memory tables.

> ***In memory tables*** - temporary copies of data files that can be processed in CAS:
    * by default *Promote=No*, session scope

> The ***LOAD*** statement in **PROC CASUTIL** is the best way to load files to in memory tables.

```
PROC CASUTIL;
    LOAD DATA=sas-data-set
    <CASOUT="target-table-name">
    <OUTCASLIB="caslib">
    <PROMOTE | REPLACE> <option(s)>;
QUIT;
```
`DATA` - used for Base SAS tables

`CASOUT` - names the in-memory table

`OUTCASLIB` - names the caslib in which the table will be loaded

`PROMOTE` - creates a global scope table

`REPLACE` - overwrites data if the in-memory table already exists

***Loading other Client-side Data Source Files***

* *Microsoft Excel od comma-separated-values(CSV) files*
```
PROC CASUTIL;
        LOAD FILE="filename.ext"
        <CASOUT="target-table-name">
        <OUTCASLIB="caslib">
        <PROMOTE | REPLACE> <option(s)>;
QUIT;
```

***Loading Server-Side Data Source Files***

```
PROC CASUTIL;
        LOAD CASDATA="source-table-name"
            <INCASLIB="caslib">
            <OUTCASLIB="caslib">
            <CASOUT="target-table-name">
            <PROMOTE | REPLACE> <option(s)>;
QUIT;
```

You can also use *DATA* step to customize data and load it to memory in the same step and direct the output to a caslib:
*exemple*

```
data casuser.sales_import;
    infile "&path/data/copy-to-casuser/sales.csv" dsd firstobs=2;
    input Employee_ID  FirstName:  $12. 
          LastName : $15. Salary 
          JobTitle : $20. Country : $2. 
          BirthDate : date9. HireDate : mmddyy10.;
    YearsEmployed=yrdif(HireDate,today());
    format BirthDate HireDate mmddyy10.;
run;
```
`PROC IMPORT`- can read a specific worksheet from a xlsx file:

```
exemple:
proc import datafile="&path/data/sales.xlsx" 
            dbms=xlsx    
            out=casuser.sales_AU replace;
    sheet=Australia;
run;
```

**NOTE:** *In both the DATA step example and the PROC IMPORT example, the code is processed on the Compute Server, but the output data is loaded to an in-memory table.*

***Accessing Database Tables:***

There are two ways: 
1) Use *SAS/ACCESS library* to read database files
    * you define a client-side conection
    * data will be treated as *a clilent-side file*
    * when data is loaded it first passes through the Compute Server 
    * then second is loaded into memory
 **NOTE:** This method is not the most efficient way to brong data into CAS

2) Use a *caslib* that uses technology called ***data connectors***
    * *data conectors* are suplied by SAS to connect to databases
    * data is treated as *a server-side file*
    * data can be loaded directly into memory
    * some connectors do parallel loads, resulting in faster load time

```
    CASLIB caslib-name DATASOURCE=
          (SRCTYPE="oracle",
           PATH="oracleServiceName",
           SCHEMA="oracleSchemaName",
           UID="oracleUserID",
           PWD="oraclePassword" <,...>);
```

    * `CASLIB` - specifies the caslib name
    * `DATASOURCE` - specifies the source type, path, schema, userID and password:

    ```
    exemple establish connection to Oracle schema:
        caslib oracas datasource=
        (srctype="oracle",
        path="orcl",
        schema="sales",
        uid="student",
        pwd=XXXXXXXXXXX);
    ```

***Saving In-Memory tables:***
`SASHDAT engine`: 
* Stores a permanent copy of in-memory data
* Data is optimized for reloading CAS
* Supports rapid parallel loading

`.sashdat file` - is a permanent copy of an  in-memory table tat can be quickly loaded into memory in the future.
* when you whant to remove the table from memory it allows to save the data 

**Syntax:**
```
    PROC CASUTIL;
        SAVE CASDATA="filename"
                     <INCASLIB="caslib"> < OUTCASLIB="caslib">
                     <CASOUT="target-table-name">
                     <EXPORTOPTIONS={export-options}>;
        QUIT;      
```

`PROC CASUTIL SAVE statement` - can create various file types, including SASHDAT files

`EXPORTOPTIONS` -specify the file type
* If you don't provide a file extension, a .sashdat file is created by default.

**Exemple:**
```
proc casutil;
    save casdata="employees" incaslib="casuser" outcaslib="casuser"
	      casout="emps_hd";
    list files;
quit
```

`LIST FILES statement` -  view the data files in the Casuser caslib


*It's a good practice to drop tables from memory if you're not actively using them:
*

**Syntax:**

```
PROC CASUTIL;
    DROPTABLE CASDATA="table-name"
        <INCASLIB="caslib"> <QUIET>;
QUIT;
```


`PROC CASUTIL SAVE statement` - remove tables from memory

`QUIET option` - suppresses error messages 


--- 


***DATA Step Processing in CAS:***

* A **DATA** step can run on either:
    * the SAS Compute Server - 
     * **single-threaded** - on either servers reads data sequentially, one row at a time
    * CAS - 
     * ***single-threaded*** or 
     * ***multi-threaded*** - enables data to be processed simustaneously on multiple **threads**:
            * receive different amounts of data
            * complete processing and return results to the server controller at different times 
            * the default option
    * ***NOTE:*** To run data step in a single-thread you can use `single=yes` option in *data* step
```
DATA caslib.cas-table-name / <SINGLE=yes>;
    SET caslib.cas-table-name;
    ...
RUN;
```

***Using BY statement***
* In *SAS9* or *SAS Compute Server*, you must sort the data before using these DATA step features
* In *CAS*, when a BY statement is used, data is distributed to threads based on the values of the by variable(or variables). So -> *No sorting is required!*
    * The rows are returned to the controller in the order in which the threads finish processing

    ```
    DATA caslib.cas-table-name;
       SET caslib.cas-table-name;
       BY BY-variable;
        …
    RUN;
    ```

***Restrictions:***
* first BY variable controls data distribution
* each thread sorts its row based on additional BY variables
* SAS Viya 3.5 and later support the DESCENDING option for *all but the first BY variable*
* results are not returned in a sorted sequence


***Several filtering options are supported in CAS:***
* **WHERE=** data set option on the *input* table
    * The **WHERE=** data set option is NOT supported on the *output* table
* **WHERE** statement
* **subseting IF** statement

**Other exceptions:**
* **infile**, **input**, **datalines** statements will run on the Compute Server and the output is transferred to an in-memory table in CAS
* SAS Viya doesn't allow updating rows of an in-memory table with: **modify**, **replace**, **remove**, respectively they are not supported in CAS
    * Use **modify**, **replace**, **remove** statements to update the SAS dataset on the Compute Server and then load to memory
* Not all functions are supported in CAS: [Formats in CAS documentation](https://documentation.sas.com/doc/en/pgmsascdc/9.4_3.2/leforinforref/n0p2fmevfgj470n17h4k9f27qjag.html)
```
SAS Formats
Data Value	Compute Server Format	Formatted Value	CAS     Alternative
22036	    WORDDATE.	            May 1, 2020	                NLDATE.
2.5	        WORDS30.	            two and fifty hundredths	none
2.5	        WORDF20.	            two and 50/100          	none
```

---

> **NOTE** To control the level of detail in log messages use `OPTIONS MSGLEVEL=N | I;`
* `MSGLEVEL=N` - provides default notes
* `MSGLEVEL=I` - includes additional notes that might provide insight into how your code was processed
    * Example, the note *"The WHERE data set option on the DATA statement is not supported with DATA step in Cloud Analytic Services."*

***Comparing Proc SQL with Proc FedSQL***

* **PROC SQL:**
    * single threaded
    * runs only on SAS Compute Server
    * multi-threaded for sorting and indexing
    * is a SAS proprietary implementation of ANSI SQL-1992
    * processes only CHAR and DOUBLE data type
    * includes many non-ANSI SAS enhancements
    * you can use mnemonics (NE, EQ, GT, LT, etc)
    * use CALCULATED keyword 
    * Allow REMERGE

* **PROC FEDSQL:**
    * fully multi-threaded
    * executes in CAS
    * is a SAS proprietary implementation of ANSI SQL-1999
    * processes 17 ANSI data types
    * includes very few non-ANSI SAS enhancements
    * processes natively in CAS
    * must use only operators (no mnemonics) - <>, =, >, <, etc
    * must to specify expressions (*NO CALCULATED keyword*)
    * doens't allow REMERGING, you have to use alternate method
    * *SYNTAX:*
    * 
    ```
    PROC FEDSQL SESSREF=cas-session-name;
        SELECT col-name, col-name
            FROM cas-table
            WHERE expression
            GROUP BY item-name
            HAVING col-name
            ORDER BY item-name;
        QUIT;
    ```

* *Not all PROC FEDSQL SELECT statements features are supported in CAS*
    * **X** FORMAT= AND **X** LABEL=
    * **X** SET operations
    * **X** Correlated subqueries
    * **X** Views
    * **X** Dictionary table queries 
    * **X** CREATE TABLE with ORDER BY

**Fully supported statements:**
* `CREATE TABLE table-name AS query expression`
*  `SELECT`
* `DROP TABLE`

* > The ALTERTABLE statement is an efficient way to **update tables and column metadata** for an in-memory table:
    ```
    PROC CASUTIL;
            ALTERTABLE CASDATA="table" INCASLIB="caslib"
                                    COLUMNS={{NAME="column1" <options>}
                                    <,{NAME="column2" <options>}>};
    QUIT;

    Column options include:
        FORMAT="format-name"
        LABEL="label"
        RENAME="col-name"
        DROP=TRUE | FALSE
    ```
    * *CASDATA= option* - names the table
    * *INCASLIB= option* - specifies the caslib name
    * *COLUMNS= option* - enables you to apply formats and labels, and to rename and drop columns
* > ALTERTABLE statement can be used to modify table attributes:
```
ALTERTABLE CASDATA="table" INCASLIB="caslib"
    <COLUMNORDER={"col1" <, "col-n">}>
    <DROP={"col1" <, "col-n">}>
    <KEEP={"col1" <, "col-n">}>;
```

---

***Column Data Types supported by CAS***
* *Character Types:*
    * **CHAR** - fixed-length based on number of bytes
     	* Some characters require 2 or more bytes of storage
     	* If the string exceeds the number of bytes it is truncated
     	* Extra bytes are padded with spaces 
     	* CHAR columns are usu
    * **VARCHAR** - varying-length string based on the number of characters 
     	* Values will take only the necessary bytes without padding with spaces
     	* `VARCHAR(n)` - values will not exceed the specified length
     	* `VARCHAR(*)` - there is no limits on maximum length(there is but veryyy large)
        ```
        Declare a varchar column syntax:
        LENGTH column-name VARCHAR(n|*);
        ```
* *Numeric Types:*
    * **DOUBLE** - double-precision, floating-point number 
     	* stored as 8 bytes
    * **INT32** - integer with a precision of 10 digits
     	* stored as a 4 bytes
    * **INT64** - integer with a precision of 19 digits
     	* stored as a 8 bytes
* *Other ANSI-compliant data types are automatically mapped to the appropriate CAS data type*
    * ex: ANSI standard types *INT* - SAS standart *INT32* 
    * ex: ANSI standard types *BIGINT* - SAS standart *INT64*

---

***The CAS Language and CAS Actions:***

> ***CASL CAS Language*** ia a scripting language used to interact directly with CAS

### Methods to submit CASL code:
1) generated behind the scenes from SAS Viya visual applications(ex: SAS Visual Analytics, SAS Data Studio)
2) from open source programming environments like Java, Python, R or Lua
3) from REST APIs
4) from your SAS programs, most common written in SAS Studio

*CAS-enabled steps are converted to CASL behind the scenes*:
`CAS-enabled steps --> CASL --> run in CAS`

### You can write your own code to invoke *CAS Actions*

> ***CAS actions*** are executable routines that performs a single task on the CAS server

There are actions to:
* **Menage Session**
* **Data Management**
* **Process Data**
* **Analyze Data**
* **Admin Tasks**

### Actions are organized into groups called: **CAS action sets**
`actionSet.action`
* ***CAS Action Set:***
    * **table** :
        * *table.loadTable*
        * *table.tableDetails*
        * *table.fileInfo*
* Use **PROC CAS** to submit actions directly to the CAS server:
    ```
    PROC CAS;
           action-set.action-name / parameter=value, parameter=value...;
    QUIT;

    exemple:
    proc cas;
        table.tableInfo / caslib="casuser";
    quit; 
    ```

### Advantages of using CASL:
* Speak the native language of the CAS Server
* Submit only the actions that you need 
* Take advantage of actions and parameters not available in CAS-enabled steps

***Loading Data to Memory using CAS actions:***
1) See the data source files available in caslib:
```
CASL:
*******
PROC CAS;
    table.fileInfo / caslib="caslib-name";
QUIT;

Base SAS:
*******
proc casutil;
    list files incaslib="casuser";
quit;
```

2) ***Load a table into memory:***
```
PROC CAS;
    table.loadTable /
        path="string", caslib="caslib-name",
        casout={caslib="string", name="table-name", ...},
        <where="where-expression",>    
        <vars={"var-1"<, "var-2", ...>},> 
        ...;
QUIT;
```

3) ***Explore data , information about columns and values in our in-memory tables***
* **table.columnInfo** -> attributes: *column*, *label*, *ID*, *type*
```
CASL:
*******
proc cas;
    table.columnInfo / table={name="sales", caslib="casuser"};
quit;

Base SAS:
********
proc contents...;
```
* **table.fetch** -> allow to examine the first few rows, it fetches and prints rows from a table , by default 20 rows are printed
```
CASL:
*******
proc cas;
	table.fetch / table={name="sales", caslib="casuser"} to=10;
quit;

Base SAS:
********
proc print...;
```

***The Simple Analytics action set*** - provides actions for performing basic analytic functions. The name of the action set is **simple**. A few examples are:
* *simple.numrows*
* *simple.freq*
* *simple.distinct* - computes the number of distinct values for one or more input columns
    * *input* - can specifies the columns to analyze
```
proc cas;
    simple.distinct / table={name="sales", caslib="casuser"};
    input={"Job Title", "Country"};
quit;
```
* *simple.summary* - generates descriptive statistics for numeric columns, including mean, sum, variance, N and more.
```
PROC CAS;
    simple.summary /
        table={name="table-name", caslib="caslib-name", groupby="col-name"},
        input={"col-name"}; 
        subset={"MIN", "MAX", "N", "MEAN"}
        casOut={name="table-name", caslib="caslib-name", replace= "true"}; 
QUIT;
```
* *groupBy* - compute statistics for each unique value of one ore more columns
* *subset* - list the statistics to include in the results
* *casOut* - creates a in-memory table in which to store the summary statistics
