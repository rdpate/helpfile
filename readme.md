Help
====

    help NAME [SUBTOPIC..]
    help --version NAME [SUBTOPIC..]
    help -c NAME [SUBTOPIC..]

Help displays an overview of purpose, options, version, and other closely related information.  For executables, Help replaces the "options" of --help and --version and does not require executing files which may be non-functional or untrusted.

Given -c/--command, lookup through $PATH if NAME does not contain "/" (like which(1)).  Otherwise NAME need not exist, as the path will be used to find the help text.

If NAME is missing and version is not requested, show help for Help.

Options
-------

    -c  --command     lookup through $PATH if NAME lacks "/", like which(1)
    -v  --version     version instead of help
    -s  --short       help synopsis (or version string) only
        --locale=L    use L instead of $LC_ALL, $LC_CTYPE, or $LANG

This implementation of Help also supports some formatting options:

        --wrap=N      wrap lines to N width or 0 for no wrap
        --no-wrap     --wrap=0


Help Sections
-------------

Help text is divided by section into files located in a "helpfile" subdirectory in the same directory as the file they describe, named after the file plus subtopic, section type, language, and file format, separated by periods.  The file may include periods, but no other component may.

    /path/to/file
    /path/to/helpfile/file.help.en.txt
    /path/to/helpfile/file.version.en.txt

If a section is not found and the parent file is a symlink, it is recursively resolved until either the section is found or the file is no longer a symlink:

    $ which example
    /path/to/example
    $ ls /path/to/helpfile/example.help.en.txt
    ls: /path/to/helpfile/example.help.en.txt: No such file or directory
    $ readlink /path/to/example
    ../../another/path/another-name
    $ ls /path/to/../../another/path/helpfile/another-name.help.en.txt
    /path/to/../../another/path/helpfile/another-name.help.en.txt

If the sections are still not found, internal sections are searched (see Internal Help Sections).


Subtopics
---------

A helpfile may have subtopics.  These are in directories named after the parents.

    /path/to/file
    /path/to/helpfile/file.help.en.txt
    /path/to/helpfile/file/subtopic.help.en.txt
    /path/to/helpfile/file/subtopic/subsubtopic.help.en.txt


Section Locales
---------------

The desired locale is given by --locale, or otherwise the first non-empty environment variable of either $LC\_ALL, $LC\_CTYPE, or $LANG.  This value is converted to lowercase and must start with "language[\_country]", where language is two ASCII letters and country, if present, is also two ASCII letters; any other value is ignored and "en" is used.

Help files are searched to find the first matching "language\_country", "language", "language" with any country (order unspecified), "en", any language without country (order unspecified), or any language with country (order unspecified).


Internal Help Sections
----------------------

Internal sections are for convenient self-contained scripts (or any file) using "#"-lines as comments, but separate section files are preferred.  Internal sections cannot specify subtopics or locales.

A file whose first line starts with "#" may have internal sections.  These sections must be within consecutive lines at the start of the file all starting with "#".  A section header starts with "#." followed by the section name.  A section line starts with "##" (excluded as a comment), starts with "# " (included with "# " removed), or is exactly "#" (a blank line after removing "#").  Any other line starting with "#" is not in that section.  Any other line not starting with "#" (including a blank line) ends all sections.

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


Comparison to Manpages
----------------------

Manpages should contain more depth than help text and should be preferred when an overview is insufficient.  Implementations of Help may even default to "man 1 COMMAND" when help text is not found (and Help is given a name without a slash).  However, help text is easier to specify, better fulfills its role, does not require installation into a global path, and isn't closely tied to terminal capabilities or restrictions.


Text Requirements
-----------------

- text MUST be encoded in UTF-8 without a byte-order mark
- text SHOULD be encoded in ASCII if convenient
- text MUST NOT be hard-wrapped, it will be wrapped according to the user's preferences and current display (which may not be a terminal)
- text MUST NOT contain markup or leading/trailing blank lines
- text lines MUST end with a newline, including the last line
- text MUST NOT contain control characters (besides newline), including carriage return and tab
- for help sections, a synopsis MUST be at the top followed by a blank line if text follows
- for help sections on commands, the synopsis MUST include the command name and common forms, eg. "command FOO [BAR..]"
- for version sections, the version string MUST be the first line followed by a blank line if text follows
    - the version string SHOULD be formatted according to semver.org
    - whether version strings may be compared according to semver.org is OPTIONAL
    - if unknown or missing, the version string MUST NOT be blank and SHOULD be "unknown"


Future Ideas
------------

- alternative implementations should be possible, and every feature must keep that in mind
    - allow for user config such as whether to use a pager or another file viewer (such as a browser), or to fallback to man (or something else)
- how should subtopics treat help synopsis and version string (--short)?
- option to list subtopics?
- should $PATH lookup not be default?
    - how does this interact with possibly replacing man?
        - eg. does "help /dev/XX" use "/dev/helpfile"?
        - how to handle ~/.file? (do not want "~/helpfile/.file.help...")
- will not support dynamic subtopics, where code must be executed to determine the final command (eg. hg and git aliases in .hgrc and .git/config)
- more formats than plain text
    - desirable: links, anchors, italic, bold, underline
    - maybe Markdown
    - probably only one more format, considering alternative implementations must support it too
        - something human-readable as text strongly preferred, allows treating it as plain text
- machine-readable specification for options and arguments to aid command completion
    - more general "properties" specification? (such as documentation URL) but quickly getting more complex than required
- internal sections could specify subtopic, locale, and format through the section header
    - these must be the only metadata in the section header, and correspond to the filename of a separate section
    - "#.help.LOCALE.FORMAT SUBTOPIC.."?
    - "#.help /SUBTOPIC.LOCALE.FORMAT"?
    - "#/SUBTOPIC.help.LOCALE.FORMAT"?
    - inline sections are exactly equivalent to the contents of a separate section with specific line munging for comments (prefixing "#" for comments and blank lines or "# " otherwise)
