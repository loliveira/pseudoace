# Change-log

## [0.4.15] (2017-02-16)
  - Added generated schema for WS257.
  - Make output of generated schema valid EDN.
  - Use latest version of datomic 0.9.5554.
  - Added API for getting the name of the main datomic database.
  - Added API for getting the version of package.
  
## [0.4.14] (2016-11-11)
  - Allow migrations with no locatable schema and/or schema fixups.

## [0.4.13] (2016-11-09)
  - Bugfix release for 0.4.12

## [0.4.12] (2016-11-09) (DO NOT USE)
  - Support suppression of schema "fix-ups" (for smallace/training
    databases).

## [0.4.11] (2016-10-10)
  - Add generated schema for WS256
  - Ability to specify max-count and max-text for log partitioning via
    a command line option to the `import-logs` command.
  - Use composite lein profiles.

## [0.4.10] (2016-07-22)
  - Bump version of various packages, notably datomic-pro and dynamodb
    aws-sdk libraries.
  - Explicitly pass the annotated models filename as arguments to the CLI
    (Annotated ACeDB models file now lives in the wormbase-pipeline repository).

## [0.4.9] - (2016-07-07)
  - When dumping .ace files, only present the leaf tag (as the `tace`
    `Show` command) - enables easier comparisons of ace output.
  - Add functions for dumping a query to .ace files

## [0.4.8] - (2016-06-24)
  - Fix wrong exit-code when helper.gz is encountered in `sort-edn-log.sh`.

## [0.4.7] - (2016-06-20)
  - `2_point_data` ACeDB class renamed to two-point-data in annotated
    models (#33).
  - Fix for circular xrefs for `phenotype` and
    `phenotype_not_observed` in annotated models (#46)
  - Bug in sort-edn-log.sh - Sort command now uses the temporary sort
    directory.

## [0.4.6] - (2016-06-15)
  - Added back missing `xbin` function in `pseudoace.binning`.

## [0.4.5] - (2016-06-14)
  - Remove hard-dependency on datomic-pro.
    Library users will now have to choose which flavour (and version)
    of datomic to include.
  - Made all functions in the `pseudoace.binning` module public.
  - EDN log sorting script now allows parallel execution.
  - Annotated models for WS255 (including generated EDN schema)

## [0.4.4] - (2016-05-12)
  - Support distributing a release bundle using either datomic-free or datomic-pro.
  - Add generated schema for WS253, WS254
  - Various minor fixes for the command line interface.

## [0.4.3] - (2016-04-28)
  - Rollback version of aws library.

## [0.4.2] - (2016-04-28)
  - Updated documentation.

## [0.4.1] - (2016-04-28)
  - Use `lein release` for releases.
  - Bump versions of datomic and storage dependencies.

## [0.4] - (2016-04-25)
 - Moved sorting of logs files back out to the shell.
 - Fixed lingering namespace issues in `pseudoace.acedump`.
 - Use `datomic.api.tempid` in code, not reader-literals.
 - Use the script `scripts/bundle-release.sh` to create a bundle for
   deploying to servers and running the importer.
 - Updated README (development and release instructions).
 - Added development tools to "dev" profile in leiningen project configuration.
 - Enable multiple storage profiles in lein project.
 - Added script to generate release bundle.
 - Models file is now defaults to `models/models.wrm.annot` (no need
   to pass on command line to sub-commands)
 - Unified the various additional schemata into a single module.
 - Import will no longer attempt to transact EDN logs older than the
   latest transaction date.
 - Report now generates two files per class when non-zero missing and
   added counts are encountered for easy diffing of object names.

## [0.3.1]
 - Added ability to generate a report for comparing a dump of classes to values
   against those stored in the database.

## [0.3.0]
 - Fix error with old-namespace:
	`acetyl` is no longer used, and has been in-lined into the
    pseudoace package.

## [0.2.0]
 - Re-organised project structure.
 - Separation of components into separate repositories.
 - Added basic unit test structure.
 - Removed dependency on perl for sorting EDN files.
 - Fixed minor issues with command line interface.

## [0.1.0]
 - Initial version
