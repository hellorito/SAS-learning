# Base SAS Comprehensive Notes
- [1. Introduction](#1-introduction)
  - [1.1. SAS Libraries and Data Sets](#11-sas-libraries-and-data-sets)
  - [1.2. SAS Program Structure](#12-sas-program-structure)
  - [1.3. Basic Syntax Rules](#13-basic-syntax-rules)
- [2. DATA Step](#2-data-step)
  - [2.1. Processing](#21-processing)
  - [2.2. Syntax](#22-syntax)
  - [2.3. Output in the DATA Step](#23-output-in-the-data-step)
    - [2.3.1. Default (Implicit) Output Behavior](#231-default-implicit-output-behavior)
    - [2.3.2. Explicit Output Using the OUTPUT Statement](#232-explicit-output-using-the-output-statement)
  - [2.4. Creating Accumulating Column - RETAIN](#24-creating-accumulating-column---retain)
  - [2.5. Processing Data in Groups](#25-processing-data-in-groups)
  - [2.6. IF-THEN / ELSE Statements](#26-if-then--else-statements)
- [3. Reading Data](#3-reading-data)
  - [3.1. Reading External Files: INFILE](#31-reading-external-files-infile)
  - [3.2. Reading Inline Data: DATALINES](#32-reading-inline-data-datalines)
  - [3.3. Accessing Data through Libraries](#33-accessing-data-through-libraries)
    - [Example: Reading Excel files](#example-reading-excel-files)
  - [3.4. Importing Data](#34-importing-data)
- [4. Data Manipulation with Functions](#4-data-manipulation-with-functions)
  - [4.1. Categories of SAS Functions](#41-categories-of-sas-functions)
  - [4.2. Numeric Funtions](#42-numeric-funtions)
  - [4.3. Character Functions](#43-character-functions)
  - [4.4. Date and Time Functions](#44-date-and-time-functions)
  - [4.5. Conditional and Logical Functions](#45-conditional-and-logical-functions)
- [5. Formats and Informats](#5-formats-and-informats)
  - [5.1. Custom Format: Using FORMAT Procedure](#51-custom-format-using-format-procedure)
    - [Creating Custom Formats from Tables](#creating-custom-formats-from-tables)
    - [External materials: Common Formats by Categories](#external-materials-common-formats-by-categories)
- [6. Procedures for Exploring, Summarizing, and Analyzing Data](#6-procedures-for-exploring-summarizing-and-analyzing-data)
  - [6.1. PROC PRINT – View Raw Data](#61-proc-print--view-raw-data)
  - [6.2. PROC CONTENTS – Dataset Metadata](#62-proc-contents--dataset-metadata)
  - [6.3. PROC MEANS – Summary Statistics (Numerical)](#63-proc-means--summary-statistics-numerical)
  - [6.4. PROC FREQ – Frequency Tables (Categorical)](#64-proc-freq--frequency-tables-categorical)
  - [6.5. PROC UNIVARIATE – Distribution + Summary](#65-proc-univariate--distribution--summary)
  - [6.6. PROC SORT – Ordering Data](#66-proc-sort--ordering-data)
- [7. Combining Tables](#7-combining-tables)
  - [7.1. Concatenating Tables](#71-concatenating-tables)
  - [7.2. Combining with MERGE](#72-combining-with-merge)
  - [7.3. Combining with PROC SQL](#73-combining-with-proc-sql)
- [8. Processing Repetitive Code](#8-processing-repetitive-code)
  - [8.1. DO Loops - Repeat Tasks Efficiently](#81-do-loops---repeat-tasks-efficiently)
    - [Iterative DO loops:](#iterative-do-loops)
    - [Conditional DO loops:](#conditional-do-loops)
  - [8.2. CALL Routines - Buit-in Utility Functions](#82-call-routines---buit-in-utility-functions)
    - [CALL MISSING - Reset Values](#call-missing---reset-values)
    - [CALL SYMPUT - Create Macro Variable Dynamically](#call-symput---create-macro-variable-dynamically)
- [9. Restructuring Tables Using PROC TRANSPOSE](#9-restructuring-tables-using-proc-transpose)

## 1. Introduction

### 1.1. SAS Libraries and Data Sets

- A Library is a collection of SAS files (data sets, catalogs, etc.).
- The WORK library is temporary and deleted after session ends.
- Permanent libraries must be assigned with a libname statement.

### 1.2. SAS Program Structure

- DATA Step: Creates or modifies SAS data sets.
- PROC Step: Performs analysis, reporting, or management tasks.
- Program is composed of one or more DATA and PROC steps, each ends with a `RUN;` or `QUIT;` statement.

### 1.3. Basic Syntax Rules

- SAS programs consist of DATA and PROC **steps**, and each step consists of **statements**.
- Global statements are outside of steps (e.g., `TITLE`,`OPTIONS`,`LIBNAME`,`FOOTNOTE`)
- All statements end with a semicolon `;`.
- Case insensitive, but good practice to capitalize keywords.
- Consistent program spacing is a good practice to make programs legible. For SAS Studio, click the Format Code icon to automate spacing.
- Comments can be written as:
  * `* This is a comment;` 
  * `/* This is a block comment */`
- Refer to the log to help diagnose and resolve syntax errors.

## 2. DATA Step

### 2.1. Processing

- The DATA step is processed in two phases: compilation and execution
- During **compilation**, SAS creates the program data vector (PDV) and establishes data attributes and rules for execution.
- The **PDV** is an area of memory established in the compilation phase. It includes all columns that will be read or created, along with their assigned attributes. The PDV is used in the execution phase to hold and manipulate **one row of data at a time**. 
- During **execution**, SAS **reads, manipulates, and writes data**. All data manipulation is performed in the PDV.

### 2.2. Syntax

```sas
data output_dataset_name;
  set input_dataset_name; /* optional */
  /* statements */
run;
```

> `data` statement names the ouput data set
> 
> `set` statement reads observations from an existing data set (optional if creating from scratch)
> 
> The code between data and run is executed once per observation

### 2.3. Output in the DATA Step

#### 2.3.1. Default (Implicit) Output Behavior

By default, SAS automatically writes one observation to the output data set for each iteration of the DATA step. This is called implicit output.

You don’t need to explicitly use the output statement unless you want to:
- Write multiple observations in one iteration, or
- Suppress output for certain cases.

```sas
data test;
  input ID $ Score;
  datalines;
A 80
B 90
;
run;
```
Each line of input automatically becomes one observation in the output data set.

#### 2.3.2. Explicit Output Using the OUTPUT Statement

> If an explicit OUTPUT statement is used in the DATA step, it disables the implicit OUTPUT at the end of the DATA step.

Use the OUTPUT statement when:
- You want to control when to write an observation.
- You want to write multiple versions of the same observation.

```sas
data doubled;
  input ID $ Score;
  Score2 = Score * 2;
  output;
  Score2 = Score * 3;
  output;
  datalines;
A 80
B 90
;
run;
```
This will output ***two rows per input row***.

### 2.4. Creating Accumulating Column - RETAIN

- At the beginning of the first iteration of the DATA step, all column values are set to missing.
- By default, all **computed columns** are **reset to missing** at the beginning of each subsequent iteration of the DATA step. This is called reinitializing the PDV. **Columns read from the SET statement** automatically **retain their value** in the PDV.
- `RETAIN` prevents variables from being reset to missing at the beginning of each DATA step iteration, **allowing them to hold their values across observations**

Syntax:

```sas
DATA output;
    RETAIN column <initial-value>;
    SET input;
    column+expression;
run;
```

### 2.5. Processing Data in Groups

- To process data in groups, the data **first must be sorted by the grouping column or columns**. This can be accomplished with `PROC SORT` before the processing step.
- The BY statement in the DATA step indicates how the data has been grouped. Each unique value of the BY column will be identified as a separate group.
- The BY statement creates **two temporary variables in the PDV** for each column listed as a BY column: **First.bycol and Last.bycol**.
- `First.bycol` is **1 for the first row within a group** and 0 otherwise. `Last.bycol` is **1 for the last row within a group** and 0 otherwise.
- Conditional `IF-THEN` logic can be used based on the values of the First./Last. variable to execute statements in the DATA step.

Syntax:

```sas
proc sort data=input out=input;
    by col-name(s);
run;

DATA output;
    SET input;
    BY <DESCENDING> col-name(s);
    if FIRST.bycol=1 and LAST.bycol=0 then ...;
    else ...;
run;
```

### 2.6. IF-THEN / ELSE Statements

Syntax:

```sas
data output;
    set input;
    if condition then statement;
    <else if condition then statement;>
    <else statement;>
run;
```

Processing multiple statements with `IF-THEN-DO`:

```sas
IF expression THEN DO;
    <executable statements>
END;
<ELSE IF expression THEN DO;
    <executable statements>
END;
ELSE DO;
    <executable statements>
END;
```

> Notes: Close each DO block with an `END` statement.

- `=`: equal to 
- `^=`, ne: not equal to
- `>`: greater than
- `<`: less than
- `>=`, `<=`: greater/less or equal
  

Example:

```sas
data eligibility;
  input Name $ Age Income;
  if Age >= 18 and Income > 20000 then Eligible = 'Yes';
  else Eligible = 'No';
  datalines;
Tom 19 25000
Sara 17 30000
Max 22 15000
;
run;
```

## 3. Reading Data

### 3.1. Reading External Files: INFILE

Syntax:

```sas
data dataset_name;
  infile 'filepath'; /* Make sure file path is correct and accesible */
  input var1 var2 $  var3 date9. ...;
run;
```

### 3.2. Reading Inline Data: DATALINES

Syntax:

```sas
data dataset_name;
  input var1 var2 $ var3;
  datalines;
value1 value2 value3
...
;
run;
```

### 3.3. Accessing Data through Libraries

Syntax:

```sas
LIBNAME libref engine "path";
LIBNAME libref CLEAR;
```

- A `libref` is the name of the library that can be used in a SAS program to read data files.
- The `engine` provides instructions for reading SAS files and other types of files.
- The `path` provides the directory where the collection of tables is located.
- The libref remains active until you clear it, delete it, or shut down SAS.

#### Example: Reading Excel files

```sas
OPTIONS VALIDVARNAME=V7;
LIBNAME libref XLSX "path/file.xlsx";
```

- The `XLSX` engine enables us to read data directly from Excel workbooks. 
- The `VALIDVARNAME=V7` system option forces table and column names read from Excel to
**adhere to recommended SAS naming conventions**. Spaces and special symbols are replaced with underscores, and names greater than 32 characters are truncated.
- Each work sheet displays as a SAS data set (.sas7bdat)

### 3.4. Importing Data

Syntax:

```sas
/* Excel */
PROC IMPORT DATAFILE="file.xlsx" DBMS=XLSX OUT=output-table <REPLACE>;
<SHEET=sheet-name;>
RUN;

/* csv */
PROC IMPORT DATAFILE="file.csv" DBMS=CSV OUT=output-table <REPLACE>;
<GUESSINGROWS=n>; 
RUN;
```

- The `DBMS` option identifies the **file type**. 
- The `OUT=` option provides the **library and name** of the SAS output table.
- The `REPLACE` option is necessary to **overwrite the SAS output table if it exists** (optional).
- SAS assumes that column names are in the first line of the text file and data begins on the second line.
- Date values are automatically converted to numeric SAS date values and formatted for easy interpretation.
- The `GUESSINGROWS=` option indicates the number of rows the IMPORT procedure scans in the input file to **determine the appropriate data type and length of columns**. The default value is 20 and the allowed range is 1 to 2147483647 (or ***using `MAX` to avoid truncation***).

## 4. Data Manipulation with Functions

### 4.1. Categories of SAS Functions

|Category|Examples|
|--------|--------|
|Mathematical|`SUM`,`MEAN`,`ROUND`,`CEIL`,`FLOOR`,`INT`,`LARGEST`|
|Character|`COMPBL`,`COMPRESS`,`STRIP`,`SCAN`,`SUBSTR`,`TRIM`,`UPCASE`,`PROPCASE`,`CATX`,`TRANWRD`,`LENGTH`|
|Date & Time|`DATEPART`,`TIMEPART`,`INTCK`,`INTNX`,`TODAY`|
|Conditional|`COALESCE`,`IFN`,`IFC`|

> You can convert column types using `INPUT` or `PUT`
> 
> The `INPUT` function converts a character value to a numeric value using a specified informat. SAS automatically tries to convert character values to numeric values using the w. informat.
> 
> The `PUT` function converts a numeric or character value to a character value using a specified format. SAS automatically tries to convert numeric values to character values using the BEST12. format.

### 4.2. Numeric Funtions

- sum(x, y): Add values, ignores missing
- mean(x, y): Returns average, ignores missing
- largest(k, value1<, value2, value3, ...>): Returns the *kth* largest non-missing value.
- round(x, n): Rounds x to nearest n. Default is 1, round x to the nearest integer; if n = 0.1, round x to one decimal place
- int(x): Integer portion of x (truncate)
- ceil(x): Returns the smallest integer that is greater than or equal to x (**4.3 -> 5**)
- floor(x): Returns the largest integer that is less than or equal to x (**4.3 -> 4**)

Example:

```sas
data math_example;
  a1 = 5.6;
  a2 = 3.2;
  a3 = 8.8;
  a4 = 3.4;
  sum1 = sum(a, b);           /* 8.8 */
  rounded = round(a1, 1);      /* 6.0 */
  int_part = int(a2);          /* 3 */
  remainder = mod(10, 3);     /* 1 */
  second_large = largest(2, of a:); /* 5.6 */
run;
```

### 4.3. Character Functions

- SUBSTR(x, p, n): Extract substring from x starting at p for n characters
- SCAN(x, n <, 'delimiters'>): Returns the *nth* word in x. If n is negative, the SCAN function begins reading from the right side of the string
- FIND(string, substring <, 'modifiers'>): Return a numeric value that identifies the location of selected substring. 
- CATX('delimiter', string1, ... stringn): Concatenates strings together, removes leading or trailing blanks from each string, and inserts the delimiter between each string
  
- UPCASE(x): Converts text to uppercase
- LOWCASE(x): Converts text to lowercase
- PROPCASE(x<, 'delimiters'>): Converts to uppercase the first character of each word. The default delimiter is blank

- STRIP(x): Removes leading and trailing blanks
- TRIM(x): Removes trailing blanks
- COMPBL(x): Returns a character string with all multiple blanks in the source string converted to single blanks
- COMPRESS(x`<,characters>`): Returns a character string with specified characters removed from the source string. Default is blanks

Example:
```sas
data string_example;
  name = '  Alice  ';
  first3 = substr(name, 1, 3);        /* "  A" */
  name_upper = upcase(name);         /* "  ALICE  " */
  trimmed = trim(name);              /* "  Alice" */
  stripped = strip(name);            /* "Alice" */
  fullname = catx(' ', 'Alice', 'Smith'); /* "Alice Smith" */
run;
```

### 4.4. Date and Time Functions

- INTCK('interval',start-date,end-date <,'method'>): Count the number of intervals that have occured between a start and end date. You can specify 'C' to use the continuous method for counting intervals.
- INTNX('interval',start,increment <,'alignment'>): Adds *increment* intervals to a date (*start*). Alignment can be start/middle/end/same to align with the start specified.

Example:
```sas
data date_example;
  today = today();
  next_month = intnx('month', today, 1); /* 1 means next month here, 0 means this month */
  days_passed = intck('day', '01jan2023'd, today);
  format today next_month date9.;
run;
```

### 4.5. Conditional and Logical Functions

These functions allow conditional assignment **within an expression** (unlike IF-THEN statements).

- IFN(cond, val1, val2): Numeric IF-THEN logic (if *cond* then *val1* else *val2*)
- IFC(cond, val1, val2): Character version of IFN
- COALESCE(x1, x2, ...): Returns **first** non-missing value

Example:
```sas
data logic_example;
  score = 85;
  result = ifn(score >= 60, 1, 0);          /* Numeric */
  status = ifc(score >= 60, 'Pass', 'Fail');/* Character */
  x = .; y = 3; z = 5;
  first_nonmissing = coalesce(x, y, z);     /* 3 */
run;
```

## 5. Formats and Informats

- Formats: **Display** variables in reports, printed output, or for grouping (e.g., date/time, currency, percent)
- Informats: **Read** data, especially raw text files 

Example:

```sas
/* Format for Reporting */
data finance;
  input Name $ Sales TaxRate;
  Total = Sales * TaxRate;
  format Sales dollar10.2 TaxRate percent8.1 Total comma10.2;
  datalines;
Amy 10000 0.075
Bob 15000 0.085
;
run;

proc print data=finance;
run;

/* Read Raw Data with Informats */
data salary;
  infile datalines dsd;
  informat Name $10. Income dollar8.;
  input Name $ Income;
  datalines;
Alice,$40000
Bob,$55000
;
run;

proc print data=salary;
run;
```

### 5.1. Custom Format: Using FORMAT Procedure

Syntax:

```sas
PROC FORMAT;
    VALUE format-name value-or-range-1 = 'formatted-value'
                        value-or-range-2 = 'formatted-value'
. . . ;
RUN;
```

- A `VALUE` statement specifies the **criteria** for creating one custom format.
- The **format name** can be **up to 32 characters in length***, must *begin with a **$** followed by a letter or underscore* for **character formats**, and must *begin with a letter or underscore* for **numeric formats**.
- On the left side of the equal sign, you specify individual values or a range of values that you want to convert to formatted values. **Character** values must be **in quotation marks**; **numeric** values are **not quoted**.
- The keywords LOW, HIGH, and OTHER can be used in the VALUE statement. (e.g., low-<10='Small' 10-high='Large';)
- Include **period (.)** when using the format.
- Custom formats can be used in the **FORMAT statement** and the **PUT function**. (e.g., `agegroup=put(age, agegrp.);`,`gender=put(sex, $sexfmt.)`)

Example: Numeric format

```sas
proc format;
  value agegrp
    0 - 12 = 'Child'
    13 - 19 = 'Teen'
    20 - 64 = 'Adult'
    65 - high = 'Senior';
run;

data people;
  input Name $ Age;
  format Age agegrp.; /* include period, no $ sign*/
  datalines;
Tom 10
Linda 25
Paul 70
;
run;

proc print data=people;
run;
```

Example: Character format

```sas
proc format;
  value $genderfmt
    'M' = 'Male'
    'F' = 'Female';
run;

data students;
  input ID $ Gender $;
  format Gender $genderfmt.; /* Include period, use $ sign */
  datalines;
001 M
002 F
;
run;

proc print data=students;
run;
```

#### Creating Custom Formats from Tables

Syntax:

```sas
PROC FORMAT CNTLIN=input-table FMTLIB;
    SELECT format-names;
RUN;
```

- The `CNTLIN=` option specifies a **table** from which PROC FORMAT builds formats.
- The input table must contain at a minimum three character columns:
 > - **Start**, which represents the raw data values to be formatted. 
 > - **Label**, which represents the formatted labels.
 > - **FmtName**, which contains the name of the format that you are creating. Character formats start with a dollar sign.

Example:

```sas
/*Create the $SBFMT format for subbasin codes*/
data sbdata;
    retain FmtName '$sbfmt';
    set pg2.storm_subbasincodes(rename=(Sub_Basin=Start 
                                        SubBasin_Name=Label));
    keep Start Label FmtName;
run;
proc format cntlin=sbdata;
run;

/*Create the CATFMT format for storm categories*/
data catdata;
    retain FmtName "catfmt";
    set pg2.storm_categories(rename=(Low=Start 
                                     High=End
                                     Category=Label));
    keep FmtName Start End Label;
run;
proc format cntlin=catdata;
run;

proc format fmtlib library=work;
	select $sbfmt catfmt;
run;     /* --> listing the format content in the result tab */
```

#### External materials: [Common Formats by Categories](https://documentation.sas.com/doc/en/pgmsascdc/9.4_3.5/leforinforref/n0p2fmevfgj470n17h4k9f27qjag.htm "SAS Documentation")


## 6. Procedures for Exploring, Summarizing, and Analyzing Data

SAS procedures (PROCs) are powerful prebuilt routines for **data exploration, summarization, and basic statistical analysis.**

### 6.1. PROC PRINT – View Raw Data

Purpose:
- Print selected variables and rows
- Apply formatting or simple logic

Syntax:

```sas
PROC PRINT DATA=input-table(OBS=n);
    VAR col-name(s);
    <where condition;>
RUN;
```

> PROC PRINT lists all columns and rows in the input table by default. 
> 
> The `OBS=` data set option limits the number of rows listed. 
> 
> The `VAR` statement limits and orders the columns listed.

### 6.2. PROC CONTENTS – Dataset Metadata

Purpose:
- View dataset structure (Data set name, engine size, obs count)
- See variable types, lengths, formats, labels

Syntax:

```sas
PROC CONTENTS DATA=dataset;
RUN;
```

### 6.3. PROC MEANS – Summary Statistics (Numerical)

Purpose:
- Compute numeric summaries: mean, min, max, std, n

Syntax:

```sas
PROC MEANS DATA=input-table <stat-list> <options>;
    VAR col-name(s);
    <CLASS groupvar;>
    <WAYS n>
RUN;
```

> PROC MEANS generates simple summary statistics for each numeric column in the input data by default. The VAR statement limits the variables to analyze.
>
> The `CLASS` statement specifies variables to group the data before calculating statistics.
> 
> The `WAYS` statement specifies the number of ways to make unique combinations of class variables.

Example:

```sas
proc means data=sashelp.class mean min max std;
  var height weight;
  class sex;
run;
```

### 6.4. PROC FREQ – Frequency Tables (Categorical)

Purpose:
- Count **occurrences** of categories
- Generate cross-tabulations

Syntax:

```sas
PROC FREQ DATA=input-table <ORDER=freq|FORMATTED|DATANLEVELS>;
    TABLES col-name(s) < / options>;
RUN;
```

> PROC FREQ creates a frequency table for each variable in the input table by default. You can limit the variables analyzed by using the TABLES statement.
>
> You can use PROC FREQ to list **unique values and frequency**
>
> Using `ORDER=FREQ` to list the highest count first 
>
> TABLES statement options: 
> 
> `NOCUM`: Suppresses display of cumulative frequencies and percentages
>
> `NOPERCENT`: Suppresses display of percentages
>
> `PLOT=FREQPLOT` (must turn on ODS graphics): Requests plots from ODS Graphics
>
> [More TABLES Statement Options](https://documentation.sas.com/doc/en/statug/15.2/statug_freq_syntax08.htm)

Example:

```sas
/* place an asterisk between two columns to produce a two-way frequency or crosstabulation report. */
proc freq data=input;
    tables Type*Region /  nocol crosslist 
                plots=freqplot(groupby=row scale=grouppercent 
                orient=horizontal);
run;
```

### 6.5. PROC UNIVARIATE – Distribution + Summary 

Purpose:
- Full distribution profile: histogram, skewness, outliers
- Includes **extreme values** and quantiles

Syntax:

```sas
PROC UNIVARIATE DATA=input-table;
    VAR col-name(s);
    <histogram varname / normal;>
RUN;
```

Example:

```sas
proc univariate data=sashelp.class;
  var weight;
  histogram weight / normal;
run;
```

> Returns Summary table (mean, std, skewness) and Histogram with overlaid normal curve

### 6.6. PROC SORT – Ordering Data

Purpose:
- Sort data by one or more variables
- Needed before using BY-group processing
  
Syntax:

```sas
PROC SORT DATA=input-table <OUT=output-table> <NODUPKEY> <DUPOUT=output-table>;
    BY <DESCENDING> col-name(s);
RUN;
```

## 7. Combining Tables

### 7.1. Concatenating Tables

Syntax:

```sas
DATA output-table;
    SET input-table1(rename=(current-colname=new-colname))
        input-table2 ...;
RUN;
```

> Vertically concatenate two or more datasets.
>
> Similar to UNION ALL in SQL or pd.concat([...], axis=0) in Python.
> 
> Columns with the same name are automatically aligned. The column properties in the new table are inherited from the first table that is listed in the SET statement. 
>
> Additional DATA step statements can be used after the SET statement to manipulate the data.

### 7.2. Combining with MERGE

Syntax:

```sas
PROC SORT DATA=input-table OUT=output-table;
    BY BY-column;
RUN;

DATA output-table;
    MERGE input-table1<(IN=var1)> input-table2<(IN=var2)> ...;
    BY BY-column(s);
RUN;
```

> Any tables listed in the MERGE statement **must be sorted by the same column (or columns) listed in the BY statement**.
>
> By default, both matches and nonmatches are written to the output table in a DATA step merge.
>
> Consider using `IN=` flags to track where each observation came from.
>
> During execution, the IN= variable is assigned a value of 0 or 1. 0 means that the corresponding table did not include the BY column value for that row, and 1 means that it did include the BY-column value.
> 
> The subsetting IF or IF-THEN logic can be used to subset rows based on matching or nonmatching rows. (e.g., `IF var1 and var2;` -> only matches output)


### 7.3. Combining with PROC SQL

Syntax:

```sas
proc sql;
  create table result as
  select a.var1, b.var2, ...
  from table1 as a
  <left join> table2 as b
  on a.key = b.key;
quit;
```

> More flexible joins: inner, left, right, full

## 8. Processing Repetitive Code

When working with many similar variables, repetitive code can be replaced by loops and arrays to make your programs shorter, cleaner, and easier to maintain.

### 8.1. DO Loops - Repeat Tasks Efficiently

#### Iterative DO loops:

```sas
DATA output-table;
    SET input-table;
    ...
    DO index-column = start TO stop <BY increment>;
    ...repetitive code...
    <OUTPUT;>
    END;
    ...
    <OUTPUT;>
RUN;
```

> The **index-column** parameter names a column whose value **controls execution of the DO loop**. This column is **included in the table that is being created unless you drop it**.
>
> The `start` and `stop` values are inclusive
>
> The increment value specifies a positive or negative number to control the incrementing of the index column. The BY keyword and the increment are optional. If they are omitted, the index column is increased by 1.
>
> An `OUTPUT` statement **between the DO and END** statements outputs **one row for each iteration** of the DO loop.
>
> An `OUTPUT` statement **after the DO loop** outputs **a row based on the final iteration** of the DO loop. **The index column will be an increment beyond the stop value.** (e.g., do i =1 to 5; xxx; end; ---> output of i will be 6)
>
> DO loops can be nested.

#### Conditional DO loops:

```sas
DATA output-table;
    SET input-table;
    ...
    DO < index-column = start TO stop <BY increment> > UNTIL | WHILE (expression);
    ...repetitive code...
    <OUTPUT;>
    END;
RUN;
```

> A conditional DO loop executes based on a condition, whereas an iterative DO loop executes a set number of times.
> 
> A `DO UNTIL` executes until a condition is true, and the condition is **checked at the bottom of the DO loop**. A DO UNTIL loop **always executes at least one time**.
> 
> A `DO WHILE` executes while a condition is true, and the condition is **checked at the top of the DO loop**. A DO WHILE loop does not iterate even once if the condition is initially false.
> 
> The expression needs to be in a set of **parentheses** for the DO UNTIL or DO WHILE

**Combined loop**:

> An iterative DO loop can be combined with a conditional DO loop. The index column is listed in the DO statement before the DO UNTIL or DO WHILE condition.
>
> For an iterative loop combined with a DO UNTIL condition, the condition is checked **before** the index column is incremented **at the bottom of the loop**.
> 
> For an iterative loop combined with a DO WHILE condition, the condition is checked at the **top** of the loop and the index column is incremented at the bottom of the loop.

### 8.2. CALL Routines - Buit-in Utility Functions

CALL routines **perform actions but do not return a value** like functions do. They are used within the DATA step, often in loops or conditionals.

#### CALL MISSING - Reset Values

Purpose: set the values of one or more variables to missing (. for numeric, "" for character). More flexible than IF-THEN statement when it comes to several variables.

```sas
data reset;
  set some_data;
  if status = 'invalid' then
    call missing(age, score, height);
run;

data clear_char;
  set people;
  if gender not in ('M', 'F') then call missing(gender);
run;
```

#### CALL SYMPUT - Create Macro Variable Dynamically

```sas
data _null_;
  set summary;
  call symput('max_val', value);
run;

%put &=max_val;
```

> Puts the value of `value` into macro variable `&max_val`.

## 9. Restructuring Tables Using PROC TRANSPOSE

Purpose: Reverse tables between **wide** and **long** format

Syntax:

```sas
PROC TRANSPOSE DATA=input-table OUT=output-table <PREFIX=column> <NAME=column>;
    <BY column(s);>  /* to group transposition */
    <ID column;>     /* to turn values into column names */
    <VAR columns(s);> /* columns to transpose */
RUN;
```

> By default, all numeric columns in the input table are transposed into rows in the output table.
>
> The output table will include a separate column for each value of the ID column. There can be only one ID column. **The ID column values must be unique in the column or BY group.**
>
> The `PREFIX=` option provides a prefix for each value of the ID column in the output table.


Example: restructure SUPPAE to merge with AE (**long to wide**)

```sas
PROC TRANSPOSE data=dataprot.SUPPAE out=SUPPAE;
    BY USUBJID IDVARVAL;
    ID QNAM;
    VAR QVAL;
RUN;

PROC SQL;
    CREATE TABLE AE AS SELECT * FROM DATAPROT.AE AS A LEFT JOIN SUPPAE AS B 
    ON A.USUBJID=B.USUBJID AND A.AESEQ=INPUT(B.IDVARVAL, best.)
QUIT;
```

# Thanks for reading!

**Written by @hellorito**