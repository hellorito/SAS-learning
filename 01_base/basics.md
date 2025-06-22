# Base SAS Comprehensive Notes

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

- INTCK
- INTNK

### 4.5. Conditional and Logical Functions

## Format

## Combining Tables

## Macro

## Restructuring Tables