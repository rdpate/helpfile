Help
====

Help contains an overview of purpose, options, version, and other closely related information.  For executables, Help replaces the "options" of --help and --version without executing files which may be missing, non-functional, or untrusted.  For non-executables (including directories), Help provides these details for a file located anywhere in the filesystem through a consistent interface -- the commands provided by this package.

Quickstart:

    $ PATH="$PWD/cmd:$PATH"
    $ cd doc/examples
    $ help


Sections
----

Help is split into named sections:

* *help* -- overview
* *version* -- version string and details
* *legal* -- license, copyright, and related
* *x-* prefix -- reserved for experimental (and only experimental) use

Each section starts with a synopsis whose meaning depends on the section:

* *help* -- descriptive phrase
* *version* -- version string (eg. "2.1.3", "20170103", "0.1.0-dev")
* *legal* -- name of license (or lack thereof)

Every synopsis should be short, ideally one line and no longer than 40 characters, with the intention to be useful when displayed in a table-like layout using "name" and "synopsis" columns (so the name doesn't need to be repeated in the synopsis) as well as automatic extraction (eg. version string).

When retrieving a synopsis, a line which only consists of one or more equal signs ("====") is deleted.  This allows Markdown heading syntax without explicitly supporting that format.

After the synopsis is the rest of the section:

* *help* -- varies greatly by type of file
    * commands: operation details, example command lines, and option descriptions
* *version* -- extraneous or verbose details; see doc/case-study/vim-version.md for an example
* *legal* -- copyright statements and legal text


Locating Sections
----

Sections are located in various places, in order of precedence:

1. shadow directory tree under a "help" directory
2. adjacent to the names they describe as "FILE.SECTION.LANG.FORMAT"
3. (files) inside the files using lines prefixed by "#"
4. fallback names


Help Directory
----

A "help" shadow directory tree contains one directory per topic and one file per section.  The files are name with section type, language, and file format separated by periods.

    /path/to/topic

    /help/path/to/topic/help.en.txt
    /path/help/to/topic/help.en.txt
    /path/to/help/topic/help.en.txt
    # additionally, if "/path/to/topic" is a directory:
    /path/to/topic/help/help.en.txt

Help directories are searched "outside-in", under the presumption that system- and user-supplied files are found first and override package- and locally-supplied files.  The first file found matching topic, section, and an acceptable language (see Section Languages) stops the search, even if continuing would find different languages or formats in another directory.

The first (outside-in) candidate with any language is saved and used if the search is exhausted without finding a matching language.  If a help file is still not found, internal sections are searched (see Internal Sections).

Use --debug-locations to see location details on stderr.

### Hidden Files

As part of the search for help text, one leading period (representing a "hidden" file) is removed from basenames under the "help" directory.  If any component of that basename originally had two leading periods (eg. "..bad"), help must fail and no help text can be found.  When multiple filenames (".example" and "example") thus resolve to the same name, they will have the same help text.

Example target paths with possible help text paths:

    /home/user/.hgrc
    /help/home/user/hgrc/help.en.txt

    /example/path/.to/file
    /example/help/path/to/file/help.en.txt

    /example/.path/to/file
    /example/.path/help/to/file/help.en.txt


Internal Sections
----

Internal sections are for convenient self-contained files using #-comments; for example, shell scripts or most scripting languages.  All internal sections form a block at the start of a file of consecutive lines, which may start with optional spaces, have trailing spaces removed, and otherwise start with:

* "#." section header
* "# " (hash and one space are removed to get line contents)
* "#" and end-of-line (a blank line)
* "##" (ignored comment)
* "#" and anything else (ignored)

Example:

    #! hashbangs ignored
    # line not in any section
    #.version
    # 0.1.0
    #.legal
    # Public Domain
    #
    # This material is dedicated to the public domain.
    #.help
    # synopsis
    #
    # Help text.
    #     indented 4 spaces, because "# " is removed
    ## ignored comment
    #
    # More help text.
    not in any section and no more sections in file

Any line not starting with "#" (including a blank line) ends all sections.  Internal sections cannot specify subtopics or languages.


Fallback Names
----

For directories only and as a concession to the immense popularity of GitHub establishing practice based on their convention, some fallback names are supported for directories only.  Encouragement is given to only use these names if using GitHub or another tool which requires them.

* help section may be named README.md or README.txt
* legal section may be named license.md or license.txt

For GitHub in particular, note that README.md may be a symlink (eg. to help/help.en.md), but license.md will not be specially recognized if it is a symlink.


Subtopics
----

A file may have subtopic help files in subdirectories:

    /path/to/file
    /path/to/help/file/help.en.txt
    /path/to/help/file/subtopic/help.en.txt
    /path/to/help/file/subtopic/subsubtopic/help.en.txt

For a directory, subtopics are equivalent to help on that directory's files.


Section Languages
----

The desired language is given by --lang or the first acceptable value from $LC\_ALL, $LC\_CTYPE, or $LANG.  This value is converted to lowercase and must match "language\[\_country\]\[.\*\]", where language is two ASCII letters and country, if present, is also two ASCII letters.

Candidate directories are searched to find the first file matching language\_country, language, or "any" with a known format.  If --lang is empty or not specified and an acceptable value cannot be found from the environment, or if --lang=any, this changes to find the first file matching "any" or, in an unspecified order, any language with or without country, also with a known format.

In either case, if a known format is not found but a matching language exists with an unknown format, that file is selected -- when it cannot be displayed, an error is given.  This prevents a file in an unsupported format from being hidden.


Comparison to Manpages
----

Manpages will be preferred (as the accepted standard), but could be generated from help sections.  This implementation defaults to "man 1 COMMAND" when no help file is found, the given name does not contain slash, and --command is specified.  (Help could be installed into $PATH as a wrapper which always uses --command; see doc/examples/help-with-c).  However, help text is easier to specify, better fulfills its role, does not require installation into a global path, and isn't closely tied to terminal capabilities or restrictions.


Formats
----

Format file extensions must only use the characters "-A-Za-z0-9.,\_" (note: includes period).  Text format is the only format currently supported by this implementation, but hypothetical examples include "html" and "txt.7z".

### Text Format

The text format is "plain" and denoted with "txt" file extension.  Any line starting with "#" (no indentation allowed) is a comment and will be ignored.  This list of requirements must be followed:

* text MUST be encoded in UTF-8 without a byte-order mark
* text MUST NOT be hard-wrapped, it will be wrapped according to the user's preferences and current display (which may not be a terminal)
* text MUST NOT contain markup not intended to be directly read by a person
* text MUST NOT have leading blank lines
* text SHOULD NOT have trailing blank lines (including after comment lines are deleted)
* lines MUST end with a newline (including the last line)
* text MUST NOT contain control characters (besides newline), including carriage return and tab
* for help sections, a synopsis MUST be at the top followed by a blank line if text follows
* for help sections on commands, the synopsis MUST be one or more lines showing usage, eg. "command FOO [BAR..]"
    * ideally, only one line is listed; otherwise the first line SHOULD be the most common usage
* for version sections, the version string MUST be the first line followed by a blank line if text follows
    * if version string is unknown or missing but non-synopsis text is included, the version string MUST NOT be blank and SHOULD be "unknown"
