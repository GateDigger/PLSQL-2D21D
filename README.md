# PL/SQL2D21D
is an oracle PL/SQL setup script capable of creating database structure (tables and procedures) which serve to output results of input SQL statements into a table where each row represents a single cell of the output.

## Overview
The setup [script sql_setup_db_structure.txt](src/sql_setup_db_structure.txt) constructs, prints and optionally executes DDL statements to setup a database structure consisting of the following:
1. Input - Main definition table consisting of
   - an artificial primary key
   - a timestamp of creation
   - a SQL statement to execute
   - a timestamp of processing start
   - a timestamp of processing end
   - optional timestamp and variable name
     - the main procedure is capable of substituting one into the other before execution
2. Output - Header table consisting of
   - a reference to the main definition table
   - information about columns of the result set
     - an index
     - a name
     - type code
     - type length
3. Output - Cell table consisting of
   - a reference to the main definition table
   - column and row indices
   - a value of the result set of the provided SQL statement at the specified row and column
4. Output - Log table consisting of
   - an artificial primary key
   - a reference to the main table
   - a timestamp of the message
   - a message
5. A simple logging procedure
   - good old boring PRAGMA AUTONOMOUS_TRANSACTION
6. The export procedure
   - accepts a PK of the main table
   - retrieves the SQL statement and other necessary information from the main table
     - FOR UPDATE NOWAIT followed by writing a start timestamp, in case multiple instances attempt to process the same definition
   - opens a SQL cursor
   - parses the retrieved SQL statement
   - optionally binds the custom timestamp variable
   - describes columns of the upcoming SQL result set
   - executes the cursor
   - bulk-fetches rows and inserts them into the cell table
   - writes a finish timestamp into the main table
   - closes the cursor
   - basic exception handling is included; no guarantees about the code being bulletproof though

Configuration of the script involves a mapping of type codes to DBMS_SQL collection types, which seem to be inevitable for bulk collection. A row-by-row collection procedure can be compiled completely autonomously based on an empty driving table where column definitions serve as type representatives and filters, but such export approach offers notably slower performance.

## Tiny example
After setting up the structure, let us suppose that
  - your main table is called MAIN
    - its PK column is called SET_ID
    - its SQL column is called SET_SQL_STMT
  - the main procedure is called EXPORTER

Pick your favourite data source DATASOURCE and run the following 2 statements:
```
INSERT INTO "MAIN" ("SET_SQL_STMT")
VALUES ('SELECT * FROM "DATASOURCE"');

DECLARE
    v_max INTEGER;
BEGIN
    SELECT MAX("SET_ID") INTO v_max FROM "MAIN";
    "EXPORTER"(v_max);
END;
```
Behold, your header and cell tables now contain flattened data based on the SQL statement. Or something failed, in which case check the log table.

Cheers.

## License

MIT License

Copyright (c) 2025 GateDigger

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
