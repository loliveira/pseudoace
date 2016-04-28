# pseudoace

Provides a Clojure library for use by the [Wormbase][1] project.

Features include:

  * Model-driven import of ACeDB data into a [Datomic][2] database.
    * (Dynamic generation of an isomorphic Datomic schema from an
      annotated ACeDB models file)
  * Conversion of ACeDB database dump files into a datomic database
  * Routines for parsing and dumping ACeDB "dump files".
  * Utility functions and macros for querying WormBase data.
  * A command line interface for utilities described above (via `lein run`)

## Installation

 * Java 1.8 (Prefer official oracle version)

 * [leiningen][3].

 * Datomic Transactor (Local)
   * Visit https://my.datomic.com/downloads/pro
   * Download the version of datomic-pro that matches the version
	 specified in `project.clj`.
   * When upgrading datomic, download the latest version and update
     `project.clj` accordingly.
   * Unzip the downloaded archive, and run: `bin/maven-install`

## Development

Follow the [GitFlow][6] mechanism for branching and committing changes:

  * Feature branches should be derived from the `develop` branch:
    i.e:. git checkout -b feature-x develop

### Coding style
This project attempts to adhere to the [Clojure coding-style][7] conventions.

### Testing & code QA
Run all tests regularly, but in particular:

  * before issuing a new pull request

  * after checking out a feature-branch
  
  
```bash
alias run-tests="lein with-profile dev,test do eastwood, test"
run-tests
```

Other useful leiningen plugins for development include:

#### kibit 
Recommend [idiomatic source code changes][10].

There is editor support in Emacs. e.g: `M-x kibit-current-file`

Command line examples:

  ```bash
  # whole project
  lein with-profile dev kibit
  # single file
  lein with-profile dev kibit src/pseudoace/core.clj
  ```
#### bikeshed
Reports on [subjectively bad][11] code.
This tool checks for:
  1. "files ending in blank lines"
  2. redefined var roots in source directories"
  3. "whether you keep up with your docstrings"
  4. arguments colliding with clojure.core functions
  
Of the above, only 1. 2. and 3. are generally useful to fix,
since 4. requires creative (short) naming that may not be intuitive
for the reader. Use your discretion when choosing to "fix" any "violations"
reported in category 4.

## Releases

### Initial setup

[Configure leiningen credentials][9] for [clojars][8].

Test your setup by running:
```bash
# Ensure you are Using `gpg2`, and the `gpg-agent` is running.
# Here, gpg is a symbolic link to gpg2
gpg --quiet --batch --decrypt ~/.lein/credentials.clj.gpg
```

The output should look like (credentials elided):

```
;; my.datomic.com and clojars credentials
{#"my\.datomic\.com" {:username ...
                      :password ...}
 #"clojars" {:username ...
             :password ...}}
```

### Releases

This process re-uses the [leiningen deployment tools][12]:

  1. Checkout the `develop` branch if not already checked-out.
    1.1 Update changes entries in the CHANGES.md file
    1.2 Replace "un-released" in the latest version entry with the current date.
    1.3 Commit and push all changes.
  2. Merge the `develop` branch into to `master` (via a github pull
     request or directly using git)
  3. Checkout master.
  4. Run:
	 
	 `lein release`
	 
  5. Checkout the develop branch, update CHANGES.md with the next version
     number and a "back to development" stanza,	e.g:

	```markdown
	# 0.3.2 - (unreleased)
	  - nothing changed yet.
	```

    Commit and push these changes, typically with the message:

		"Back to development"

#### As a standalone jar file for running the import peer on a server

```bash
# GIT_RELEASE_TAG should be the annotated git release tag, e.g:
#   GIT_RELEASE_TAG="0.3.2"
#
# If you want to use a local git tag, ensure it matches the version in
# projet.clj, e.g:
#  GIT_RELEASE_TAG="0.3.2-SNAPSHOT"
#
# TARGET_DATOMIC_TYPE can be any named lein profile,
# examples:
#   TARGET_DATOMIC_TYPE="ddb"
#   TARGET_DATOMIC_TYPE="sql"
#   TARGET_DATOMIC_TYPE="dev
./scripts/bundle-release.sh $GIT_RELEASE_TAG $TARGET_DATOMIC_TYPE
```

An archive named `pseudoace-$GIT_RELEASE_TAG.tar.gz` will be created in the
`release-archives` directory.

The archive contains two artefacts:

   ```bash
   tar tvf pseudoace-$GIT_RELEASE_TAG.tar.gz
   ./pseudoace-$GIT_RELEASE_TAG.jar
   ./sort-edn-log.sh
   ```

> **To ensure we comply with the datomic license
>   ensure this tar file, and specifically  the jar file
>   contained therein is *never* distributed to a public server
>   for download, as this would violate the terms of any preparatory
>   Congnitech Datomic license.**
 

## Usage

### Development

A command line utility has been developed for ease of usage:

```bash

URL_OF_TRANSACTOR="datomic:dev://localhost:4334/*"

lein run --url $URL_OF_TRANSACTOR <command>

```

`--url` is a required option for most sub-commands, it should be of
the form of:

`datomic:<storage-backend-alias>://<hostname>:<port>/<db-name>`

Alternatively, for extra speed, one can use the Clojure routines directly
from a repl session:

```bash
# start the repl (Read Eval Print Loop)
lein repl
```

Example of invoking a sub-command:

```clojure
(list-databases {:url (System/getenv "URL_OF_TRANSACTOR")})
```

### Staging/Production

Run `pseudoace` with the same arguments as you would when using `lein run`:

  ```bash
  java -jar pseudoace-$GIT_RELEASE_TAG.jar -v
  ```

### Import process

#### Prepare import

Create the database and parse .ace dump-files into EDN.

Example:

```bash
java -jar pseudoace-$GIT_RELEASE_TAG.jar \
     --url $DATOMIC_URL \
	 --acedump-dir ACEDUMP_DIR \
	 --log-dir LOG_DIR \
	 -v prepare-import
```

The `prepare-import` sub-command:

- Creates a new database at the specified `--url`
- Converts `.ace` dump-files located in `--acedump-dir` into pseudo
[EDN][4] files located in `--log-dir`.
- Creates the database schema from the annotated ACeDB models file
specified by `--model`.
- Optionally dumps the newly created database schema to the file
specified by `--schema-filename`.

#### Sort the generated log files

The format of the generated files is:

<ace-db-style_timestamp> <Parsed ACE data to be transacted in EDN format>

The EDN data is *required* to sorted by timestamp in order to
preserve the time invariant of Datomic:

```bash
find $LOG_DIR \
    -type f \
	-name "*.edn.gz" \
	-exec ./sort-edn-log.sh {} +
```

#### Import the sorted logs into the database

Transacts the EDN sorted by timestamp in `--log-dir` to the database
specified with `--url`:

```bash
java -jar pseudoace-$GIT_RELEASE_TAG.jar \
	 --log-dir LOG_DIR \
	 -v import-logs
```

Using a full dump of a recent release of Wormbase, you can expect the
import process to take in the region of 8-12 hours depending on the
platform you run it on.

[1]: http://www.wormbase.org/
[2]: http://www.datomic.com/
[3]: http://leiningen.org/
[4]: https://github.com/edn-format/edn/
[5]: http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Introduction.html
[6]: https://datasift.github.io/gitflow/IntroducingGitFlow.html
[7]: https://github.com/bbatsov/clojure-style-guide
[8]: http://clojars.org
[9]: https://github.com/technomancy/leiningen/blob/master/doc/DEPLOY.md#authentication
[10]: https://github.com/jonase/kibit
[11]: https://github.com/dakrone/lein-bikeshed
[12]: https://github.com/technomancy/leiningen/blob/master/doc/DEPLOY.md#deployment

