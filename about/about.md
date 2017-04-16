Help
====

    help NAME [SUBTOPIC..]
    help --version NAME [SUBTOPIC..]
    help -c NAME [SUBTOPIC..]

Help displays an overview of purpose, options, version, and other closely related information.  For executables, Help replaces the "options" of --help and --version and does not require executing files which may be non-functional or untrusted.

Given -c/--command, lookup through $PATH if NAME does not contain "/", like which(1).  If $PATH lookup is not required, then NAME need not exist, as the path will be used to find the help text.

If NAME is missing and version is not requested, show help for Help.

Options
-------

    -v  --version     version instead of help
    -s  --short       help synopsis (or version string) only
        --lang=L      use L instead of $LC_ALL, $LC_CTYPE, or $LANG

This implementation of Help supports additional options:

    -c  --command     lookup through $PATH if NAME lacks "/", like which(1)
        --wrap=N      wrap lines to N width or 0 for no wrap
        --no-wrap     --wrap=0


Help Sections
-------------

Help text is divided by section into files located in a shadow directory tree from a "help" subdirectory in one of the parents of the file they describe.  The help text files are named after the target file's basename and subtopic (separated by slash), plus section type, language, and file format (separated by periods).

    /path/to/file

    /help/path/to/file.help.en.txt
    /path/help/to/file.help.en.txt
    /path/to/help/file.help.en.txt

The target file's parent directory is first resolved to be absolute without symlinks (eg. readlink -f).  Then help directories are searched "outside-in", under the presumption that system- and user-supplied files are found first and override package-supplied files.  The first file found matching basename, subtopic, section, and an acceptable language (see Section Languages) stops the search, even if continuing would find different languages or formats in another directory.

If a matching help file is still not found and the target file is a symlink, it is resolved and the search restarts:

    $ readlink -f example
    /path/to/example
    $ ls -d /help /path/help /path/help/to/example.help.en.txt /path/to/help
    ls: /help: No such file or directory
    /path/help
    ls: /path/help/to/example.help.en.txt: No such file or directory
    ls: /path/to/help: No such file or directory
    $ readlink example
    ../../another/path/another-name
    $ ls /path/to/../../another/path/help/another-name.help.en.txt
    /path/to/../../another/path/help/another-name.help.en.txt

The first (outside-in) candidate with any language is saved and used if the search is finally exhausted.  If a help file is still not found, internal sections are searched (see Internal Help Sections).


Subtopics
---------

A file's help may have subtopics in directories named after the parent topics:

    /path/to/file
    /path/to/help/file.help.en.txt
    /path/to/help/file/subtopic.help.en.txt
    /path/to/help/file/subtopic/subsubtopic.help.en.txt


Section Languages
-----------------

The desired language is given by --lang or the first acceptable value from $LC\_ALL, $LC\_CTYPE, or $LANG.  This value is converted to lowercase and must match "language\[\_country\]\[.\*\]", where language is two ASCII letters and country, if present, is also two ASCII letters.

Candidate directories are searched to find the first file matching language\_country, language, or "any".  If --lang is empty or not specified and an acceptable value cannot be found from the environment or if --lang=none, this changes to find the first file matching "any" or, in an unspecified order, any language with or without country.


Internal Sections
-----------------

Internal sections are for convenient self-contained scripts (or any file) which uses "#"-lines as comments.  Internal sections cannot specify subtopics or languages and must be within consecutive lines at the start of the file all starting with "#".  A section header starts with "#." followed by the section name.  A section line starts with "##" (excluded as a comment), starts with "# " (included with "# " removed), or is exactly "#" (a blank line after removing "#").  Any other line starting with "#" is not in that section.  Any other line not starting with "#" (including a blank line) ends all sections.

    #!/anything/here
    # not in any section
    #.version
    # 0.0.0
    #
    # Version details.
    #.help
    # synopsis
    #
    # Help text.
    #   indented
    ## section comment
    #
    # More help text.
    not in any section


Hidden Files
------------

As part of the search for help text, one leading period (representing a "hidden" file) is removed from basenames under the "help" directory.  If any component of that basename originally had two leading periods (eg. "..bad"), help must fail and no help text can be found.  When multiple filenames (".example" and "example") thus resolve to the same name, they will have the same help text.

Example target paths with possible help text paths:

    /home/user/.hgrc
    /help/home/user/hgrc.help.en.txt

    /example/path/.to/file
    /example/help/path/to/file.help.en.txt

    /example/.path/to/file
    /example/.path/help/to/file.help.en.txt


Comparison to Manpages
----------------------

Manpages should contain more depth than help text and should be preferred when an overview is insufficient.  Implementations of Help may even default to "man 1 COMMAND" when help text is not found (and Help is given a name without a slash).  However, help text is easier to specify, better fulfills its role, does not require installation into a global path, and isn't closely tied to terminal capabilities or restrictions.


Text Format
-----------

The text format is "plain" and denoted with "txt" file extension.  Any line starting with "#" (no indentation allowed) is a comment and will be ignored.  This list of requirements must be followed:

- text MUST be encoded in UTF-8 without a byte-order mark
- text SHOULD be encoded in ASCII if convenient
- text MUST NOT be hard-wrapped, it will be wrapped according to the user's preferences and current display (which may not be a terminal)
- text MUST NOT contain markup not intended to be directly read by a person
- text MUST NOT have leading blank lines
- text SHOULD NOT have trailing blank lines (including after comment lines are deleted)
- lines MUST end with a newline (including the last line)
- text MUST NOT contain control characters (besides newline), including carriage return and tab
- for help sections, a synopsis MUST be at the top followed by a blank line if text follows
- for help sections on commands, the synopsis MUST be one or more lines showing usage, eg. "command FOO [BAR..]"
    - ideally, only one line is listed; otherwise the first line SHOULD be the most common usage
- for version sections, the version string MUST be the first line followed by a blank line if text follows
    - the version string SHOULD be formatted according to semver.org
    - whether version strings may be compared according to semver.org is OPTIONAL
    - if unknown or missing, the version string MUST NOT be blank and SHOULD be "unknown"


Future Ideas
------------

- alternative implementations should be possible, and every feature must keep that in mind
    - allow for user config such as whether to use a pager or another file viewer (such as a browser), or to fallback to man (or something else)
- how widely applicable is --short for help sections?
    - subtopics?
    - non-commands?
    - what alternative synopsis will always make sense?
- separate command to list subtopics?
- will not support dynamic subtopics, where code must be executed to determine the final command (eg. as with hg and git aliases in .hgrc and .git/config)
- more formats than plain text
    - desirable: links, anchors, italic, bold, underline
    - maybe Markdown
    - probably only one more format, considering alternative implementations must support it too
        - something human-readable as text strongly preferred, allows treating it as plain text if format isn't parsed
- machine-readable specification for options and arguments to aid command completion
    - more general "properties" specification? (such as documentation URL) but quickly getting more complex than required
- internal sections could specify subtopic, language, and format through the section header
    - these must be the only metadata in the section header, and correspond to the filename of a separate section
    - "#.help.LANG.FORMAT SUBTOPIC.."?
    - "#.help /SUBTOPIC.LANG.FORMAT"?
    - "#/SUBTOPIC.help.LANG.FORMAT"?
    - inline sections are exactly equivalent to the contents of a separate section with specific line munging for comments (prefixing "#" for comments and blank lines or "# " otherwise)
- fixed-width option (--file?) to show a given helpfile without locate
    - could refactor fixed-width into three programs
        - extract internal section text
        - format helpfile text for fixed-width
        - glue for these and ./locate (ie. current version)
- find(1)-like utility that lists all help files starting from a given directory; also lists subtopics for a given filename
    - first line of synopsis will be treated as the "title"
- provide a way to specify help files for a directory within that directory
    - ie. help for "/a/b" never considers "/a/b/help/..."
