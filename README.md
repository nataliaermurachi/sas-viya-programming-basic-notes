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

***The CAS Language and CAS Actions:***

> CASL is a scripting native CAS language that's used to call CAS actions:
* an action performs a single task
* CAS actions are grouped into action sets 
* action sets are based on common functionality. 
* use `action-set.action`:

Types of actions:
* the table action set:
    * loading a table - `table.loadTable ...;`
    * getting information about a table `table.tableInfo;`
    * modifying table attributes `table.attribute;`
    * dropping a table. `table.dropTable;`

* actions to:
    * compute summary statistics- `simple.summary...;`
    * run DATA step code - `dataStep.runCode ...;`
    * transform data
    * perform analytics - 
    
> The CAS language and CAS actions execute only in CAS. They don't execute on the Compute Server.

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
