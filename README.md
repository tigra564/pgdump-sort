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

## Alternatives
`pgdump-sort` solves one narrow problem: making `pg_dump` output canonical so that diff is meaningful. Several tools address this problem, and it is worth knowing what they are.

`apgdiff` is the de-facto standard for PostgreSQL schema diff. It parses `pg_dump` output and generates DDL to migrate one schema to another. It is available in Homebrew, Debian, and Ubuntu. However, it is unmaintained: the last release was years ago, and it does not understand newer PostgreSQL features.

`pg-dump-compare` (npm) canonicalizes dump files and produces a unified diff. It is small and focused, but it only compares dumps—it does not connect to live databases or generate migration scripts.

`PostgresCompare` is a commercial tool that compares schemas between live databases, files, and snapshots. It supports 38 object types and generates migration scripts. It recently added an MCP server for AI-assisted schema management. It is powerful, but it is not free and not open-source.

`SchemaKeeper` takes a different approach: instead of one monolithic dump, it stores each schema object as a separate file, so schema changes appear as small, reviewable diffs in Git. It also includes drift detection in CI. This is closer to a workflow tool than a diff tool.

`migraguard` is a newer entrant that treats migration files as the source of truth and uses schema dumps as derived artifacts for drift detection. It supports PostgreSQL, MySQL, and SQLite, but its lint coverage for MySQL and SQLite is limited to 17 generic rules versus 38 PostgreSQL-specific rules.

## Where does pgdump-sort fit?
It does not. `pgdump-sort` is not a schema diff tool. It is a preprocessing step that makes `pg_dump` output suitable for `diff`. Tools like `apgdiff` and `pg-dump-compare` solve the same problem at a higher level: they parse the dump and compare it structurally, not line by line.

The reason `pgdump-sort` exists is that sometimes you do not want a full schema diff tool. Sometimes you just want to know if two dumps differ, and you want to use `git diff` to see how. `pgdump-sort` makes that possible. It is a small tool for a small job.

If you need schema comparison, use `apgdiff` or `PostgresCompare`. If you need schema tracking in Git, use `SchemaKeeper`. If you need drift detection in CI, `migraguard` or `SchemaKeeper` will do the job. `pgdump-sort` is not a replacement for any of them.
