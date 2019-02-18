# pgdump-sort
Sort entries of pg_dump output for the purpose of diffing database structure and contents.

## Description

Sometimes a maintainer of a Postgresql database needs to track changes made to his DB.  The utility pg_dump shipped within Postgresql distribution nearly does the trick.  However it is not guaranteed that the entries dumped by pg_dump will be in a canonical order suitable for creating minimal diff.

This program solves the issue by sorting entries of pg_dump output and outputting it in a separate file.

## Installation and prerequisites

Installation not needed.  The following software has to be installed prior to pgdump-sort usage:

* Python version 3 (however you may run the utility with python2 as well)
* Python module docopt

## Usage

```shell
$ pg_dump ... --file dump.sql
$ pgdump-sort dump.sql dump-sorted.sql
```

This will create dump-sorted.sql with the same data (and number of lines) reordered according to record type, owner, schema and name.

The utility will also transform particular lines of the dump into a canonical form.  The transformations include:

* Changing timestamps in `-- Started on` and  `-- Completed on` commentary lines to the beginning of the epoch, e.g. `-- Started on 1970-01-01 00:00:00 UTC`.
* Resetting `-- TOC entry` and `-- Dependencies:` commentary lines to all zeroes, e.g. `-- TOC entry 0 (class 0 OID 0)`
* Resetting all sequence values to 1, e.g. `SELECT pg_catalog.setval('foo.bar_seq', 1, false);`
* Sorting all data in DML blocks lexicographically.  Both `COPY` and `INSERT` (one per line) modes are supported.

## Known issues

* The tool works on the initial dump line by line without deep inspection. This means that the dump may be broken and not suitable for injecting into psql.  However, the utility will anyway fulfill its main purpose: bring the dump to a diffable form.
* During operation pgdump-sort creates a temporary directory and stores each entry in an individual file which name as constructed as a concatenation of various object properties including object name.  Since Postgresql has function overloading the name must be stored with all fuction arguments which may result into the file name exceeding OS limits (255 chars).  In this case filename is truncated to 252 chars plus '...'.