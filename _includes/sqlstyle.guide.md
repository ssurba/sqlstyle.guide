# SQL style guide

## Overview

You can use this set of guidelines, [fork them][fork] or make your own - the
key here is that you pick a style and stick to it. To suggest changes
or fix bugs please open an [issue][issue] or [pull request][pull] on GitHub.

These guidelines are designed to be compatible with Joe Celko's [SQL Programming
Style][celko] book to make adoption for teams who have already read that book
easier. This guide is a little more opinionated in some areas and in others a
little more relaxed. It is certainly more succinct where [Celko's book][celko]
contains anecdotes and reasoning behind each rule as thoughtful prose.

It is easy to include this guide in [Markdown format][dl-md] as a part of a
project's code base or reference it here for anyone on the project to freely
read—much harder with a physical book.

SQL style guide by [Simon Holywell][simon] is licensed under a [Creative Commons
Attribution-ShareAlike 4.0 International License][licence].
Based on a work at [https://www.sqlstyle.guide/][sqlstyleguide].

## General

### Do

* Naming convention as PascalCase. 
* Use consistent and descriptive identifiers and names.
* Make judicious use of white space and indentation to make code easier to read.
* Store [ISO 8601][iso-8601] compliant time and date information
  (`YYYY-MM-DDTHH:MM:SS.SSSSS`).
* Try to only use standard SQL functions instead of vendor-specific functions for
  reasons of portability.
* Keep code succinct and devoid of redundant SQL, such as unnecessary quoting or
  parentheses or `WHERE` clauses that can otherwise be derived.
* Include comments in SQL code where necessary. Use the C style opening `/*` and
  closing `*/` where possible otherwise precede comments with `--` and finish
  them with a new line.
* Avoid using nested SELECTs at all possible, replace with CTEs which are more readable.

```sql
SELECT FileHash  -- Stored ssdeep hash
  FROM FileSystem
 WHERE FileName = '.vimrc';
```
```sql
/* Updating the file record after writing to the file */
UPDATE FileSystem
   SET FileModifiedDate = '1980-02-22 13:19:01.00000',
       FileSize = 209732
 WHERE FileName = '.vimrc';
```

### Avoid

* Leading commas — it is difficult to scan quickly.
* Descriptive prefixes or Hungarian notation such as `sp_` or `tbl`.
* Inconsistent naming, e.g. Process vs Refresh, ChgId vs ExecutionID, etc.
* Plurals—use the more natural collective term where possible instead. For example
  `staff` instead of `employees` or `people` instead of `individuals`.
* Quoted identifiers—if you must use them then stick to SQL-92 double quotes for
  portability (you may need to configure your SQL server to support this depending
  on vendor).
* Object-oriented design principles should not be applied to SQL or database
  structures.
  

## Naming conventions

### General

* Ensure the name is unique and does not exist as a
  [reserved keyword][reserved-keywords].
* Keep the length to a maximum of 30 bytes—in practice this is 30 characters
  unless you are using a multi-byte character set.
* Names must begin with a letter and may not end with an underscore.
* Only use letters, numbers and underscores in names.
* Avoid abbreviations and if you have to use them make sure they are commonly
  understood.

```sql
SELECT FirstName
  FROM Staff;
```

### Tables

* Use a collective name or, less ideally, a plural form. For example (in order of
  preference) `staff` and `employees`.
* Do not prefix with `tbl` or any other such descriptive prefix or Hungarian
  notation.
* Never give a table the same name as one of its columns and vice versa.

### Columns

* Always use the singular name.
* Do not add a column with the same name as its table and vice versa.

### Aliasing or correlations

* Should relate in some way to the object or expression they are aliasing.
* As a rule of thumb the correlation name should be the first letter of each word
  in the object's name.
* If there is already a correlation with the same name then append a number.
* Always include the `AS` keyword—makes it easier to read as it is explicit.
* For computed data (`SUM()` or `AVG()`) use the name you would give it were it
  a column defined in the schema.

```sql
SELECT FirstName AS fn
  FROM Staff AS s1
  JOIN Students AS s2
    ON S2.MentorID = s1.StaffNum;
```
```sql
SELECT SUM(s.MonitorTally) AS MonitorTotal
  FROM staff AS s;
```

### Stored procedures

* The name must contain a verb.
* Do not prefix with `sp_` or any other such descriptive prefix or Hungarian
  notation.

## Query syntax

### Reserved words

Always use uppercase for the [reserved keywords][reserved-keywords]
like `SELECT` and `WHERE`.

Do not use database server specific keywords where an ANSI SQL keyword already
exists performing the same function. This helps to make the code more portable.

```sql
SELECT ModelNum
  FROM Phones AS p
 WHERE p.ReleaseDate > '2014-09-30';
```

### White space

To make the code easier to read it is important that the correct complement of
spacing is used. Do not crowd code or remove natural language spaces.

#### Spaces (TBC)

Spaces should be used to line up the code so that the root keywords all end on
the same character boundary. This forms a river down the middle making it easy for
the readers eye to scan over the code and separate the keywords from the
implementation detail. Rivers are [bad in typography][rivers], but helpful here.

```sql
(SELECT f.SpeciesName,
        AVG(f.Height) AS AverageHeight, AVG(f.Diameter) AS AverageDiameter
   FROM Flora AS f
  WHERE f.SpeciesName = 'Banksia'
     OR f.SpeciesName = 'Sheoak'
     OR f.SpeciesName = 'Wattle'
  GROUP BY f.SpeciesName, f.ObservationDate)

  UNION ALL

(SELECT b.SpeciesName,
        AVG(b.Height) AS AverageHeight, AVG(b.Diameter) AS AverageDiameter
   FROM BotanicGardenFlora AS b
  WHERE b.SpeciesName = 'Banksia'
     OR b.SpeciesName = 'Sheoak'
     OR b.SpeciesName = 'Wattle'
  GROUP BY b.SpeciesName, b.ObservationDate);
```

Notice that `SELECT`, `FROM`, etc. are all right aligned while the actual column
names and implementation-specific details are left aligned.

Although not exhaustive always include spaces:

* before and after equals (`=`)
* after commas (`,`)
* surrounding apostrophes (`'`) where not within parentheses or with a trailing
  comma or semicolon.

```sql
SELECT a.title, a.release_date, a.recording_date
  FROM albums AS a
 WHERE a.title = 'Charcoal Lane'
    OR a.title = 'The New Danger';
```

#### Line spacing

Always include newlines/vertical space:

* before `AND` or `OR`
* after semicolons to separate queries for easier reading
* after each keyword definition
* after a comma when separating multiple columns into logical groups
* to separate code into related sections, which helps to ease the readability of
  large chunks of code.

Keeping all the keywords aligned to the righthand side and the values left aligned
creates a uniform gap down the middle of the query. It also makes it much easier to
to quickly scan over the query definition.

```sql
INSERT INTO Album (Title, ReleaseDate, RecordingDate)
VALUES ('Charcoal Lane', '1990-01-01 01:01:01.00000', '1990-01-01 01:01:01.00000'),
       ('The New Danger', '2008-01-01 01:01:01.00000', '1990-01-01 01:01:01.00000');
```

```sql
UPDATE Album
   SET ReleaseDate = '1990-01-01 01:01:01.00000'
 WHERE Title = 'The New Danger';
```

```sql
SELECT a.Title,
       a.ReleaseDate, a.RecordingDate, a.ProductionDate -- Grouped dates together
  FROM Album AS a
 WHERE a.Title = 'Charcoal Lane'
    OR a.Title = 'The New Danger';
```

### Indentation

To ensure that SQL is readable it is important that standards of indentation
are followed.

### Preferred formalisms

* Make use of `BETWEEN` where possible instead of combining multiple statements
  with `AND`.
* Similarly use `IN()` instead of multiple `OR` clauses.
* Where a value needs to be interpreted before leaving the database use the `CASE`
  expression. `CASE` statements can be nested to form more complex logical structures.
* Avoid the use of `UNION` clauses and temporary tables where possible. If the
  schema can be optimised to remove the reliance on these features then it most
  likely should be.

```sql
SELECT
  CASE Postcode
      WHEN 'BN1' THEN 'Brighton'
      WHEN 'EH1' THEN 'Edinburgh'
  END AS City
FROM OfficeLocations
WHERE Country = 'United Kingdom'
  AND OpeningTime BETWEEN 8 AND 9
  AND Postcode IN ('EH1', 'BN1', 'NN1', 'KW1');
```

## Create syntax

When declaring schema information it is also important to maintain human-readable
code. To facilitate this ensure that the column definitions are ordered and
grouped together where it makes sense to do so.

Indent column definitions by four (4) spaces within the `CREATE` definition.

### Choosing data types

* Where possible do not use vendor-specific data types—these are not portable and
  may not be available in older versions of the same vendor's software.
* Only use `REAL` or `FLOAT` types where it is strictly necessary for floating
  point mathematics otherwise prefer `NUMERIC` and `DECIMAL` at all times. Floating
  point rounding errors are a nuisance!

### Specifying default values

* The default value must be the same type as the column — if a column is declared
  a `DECIMAL` do not provide an `INTEGER` default value.
* Default values must follow the data type declaration and come before any
  `NOT NULL` statement.

### Constraints and keys

Constraints and their subset, keys, are a very important component of any
database definition. They can quickly become very difficult to read and reason
about though so it is important that a standard set of guidelines are followed.

#### Choosing keys

Deciding the column(s) that will form the keys in the definition should be a
carefully considered activity as it will effect performance and data integrity.

1. The key should be unique to some degree.
2. Consistency in terms of data type for the value across the schema and a lower
   likelihood of this changing in the future.
3. Can the value be validated against a standard format (such as one published by
   ISO)? Encouraging conformity to point 2.
4. Keeping the key as simple as possible whilst not being scared to use compound
   keys where necessary.

It is a reasoned and considered balancing act to be performed at the definition
of a database. Should requirements evolve in the future it is possible to make
changes to the definitions to keep them up to date.

#### Defining constraints

Once the keys are decided it is possible to define them in the system using
constraints along with field value validation.

##### General

* Tables must have at least one key to be complete and useful.
* Constraints should be given a custom name excepting `UNIQUE`, `PRIMARY KEY`
  and `FOREIGN KEY` where the database vendor will generally supply sufficiently
  intelligible names automatically.

##### Layout and order

* Specify the primary key first right after the `CREATE TABLE` statement.
* Constraints should be defined directly beneath the column they correspond to.
  Indent the constraint so that it aligns to the right of the column name.
* If it is a multi-column constraint then consider putting it as close to both
  column definitions as possible and where this is difficult as a last resort
  include them at the end of the `CREATE TABLE` definition.
* If it is a table-level constraint that applies to the entire table then it
  should also appear at the end.
* Use alphabetical order where `ON DELETE` comes before `ON UPDATE`.
* If it make senses to do so align each aspect of the query on the same character
  position. For example all `NOT NULL` definitions could start at the same
  character position. This is not hard and fast, but it certainly makes the code
  much easier to scan and read.

##### Validation

* Use `LIKE` constraint to ensure the integrity of strings
  where the format is known.
* Where the ultimate range of a numerical value is known it must be written as a
  range `CHECK()` to prevent incorrect values entering the database or the silent
  truncation of data too large to fit the column definition. In the least it
  should check that the value is greater than zero in most cases.
* `CHECK()` constraints should be kept in separate clauses to ease debugging.

##### Example

```sql
CREATE TABLE staff (
    PRIMARY KEY (staff_num),
    staff_num      INT(5)       NOT NULL,
    first_name     VARCHAR(100) NOT NULL,
    pens_in_drawer INT(2)       NOT NULL,
                   CONSTRAINT pens_in_drawer_range
                   CHECK(pens_in_drawer BETWEEN 1 AND 99)
);
```

### Designs to avoid

* Object-oriented design principles do not effectively translate to relational
  database designs—avoid this pitfall.
* Placing the value in one column and the units in another column. The column
  should make the units self-evident to prevent the requirement to combine
  columns again later in the application. Use `CHECK()` to ensure valid data is
  inserted into the column.
* [Entity–Attribute–Value][eav] (EAV) tables—use a specialist product intended for
  handling such schema-less data instead.
* Splitting up data that should be in one table across many tables because of
  arbitrary concerns such as time-based archiving or location in a multinational
  organisation. Later queries must then work across multiple tables with `UNION`
  rather than just simply querying one table.

[simon]: https://www.simonholywell.com/?utm_source=sqlstyle.guide&utm_medium=link&utm_campaign=md-document
    "SimonHolywell.com"
[issue]: https://github.com/treffynnon/sqlstyle.guide/issues
    "SQL style guide issues on GitHub"
[fork]: https://github.com/treffynnon/sqlstyle.guide/fork
    "Fork SQL style guide on GitHub"
[pull]: https://github.com/treffynnon/sqlstyle.guide/pulls/
    "SQL style guide pull requests on GitHub"
[celko]: https://www.amazon.com/gp/product/0120887975/ref=as_li_ss_tl?ie=UTF8&linkCode=ll1&tag=treffynnon-20&linkId=9c88eac8cd420e979675c815771313d5
    "Joe Celko's SQL Programming Style (The Morgan Kaufmann Series in Data Management Systems)"
[dl-md]: https://raw.githubusercontent.com/treffynnon/sqlstyle.guide/gh-pages/_includes/sqlstyle.guide.md
    "Download the guide in Markdown format"
[iso-8601]: https://en.wikipedia.org/wiki/ISO_8601
    "Wikipedia: ISO 8601"
[rivers]: https://practicaltypography.com/one-space-between-sentences.html
    "Practical Typography: one space between sentences"
[reserved-keywords]: #reserved-keyword-reference
    "Reserved keyword reference"
[eav]: https://en.wikipedia.org/wiki/Entity%E2%80%93attribute%E2%80%93value_model
    "Wikipedia: Entity–attribute–value model"
[sqlstyleguide]: https://www.sqlstyle.guide/
    "SQL style guide by Simon Holywell"
[licence]: https://creativecommons.org/licenses/by-sa/4.0/
    "Creative Commons Attribution-ShareAlike 4.0 International License"
