Help
====

    help COMMAND [SUBTOPIC..]
    help --version COMMAND [SUBTOPIC..]

Help displays an overview of a command's purpose, options, version, and other closely related information.  Help replaces the "options" of --help and --version and does not require executing files, which may be non-functional or untrusted.  Help text is looked up by filename; a name without a slash follows a $PATH lookup.  The text can either be in a separate file or inside a self-contained script.

Options
-------

    -v  --version     version text instead of help text
    -s  --short       help synopsis (or version string) only
        --locale=L    use L instead of $LC_ALL, $LC_CTYPE, or $LANG

This implementation of Help also supports some formatting options:

        --wrap=N      wrap lines to N width or 0 for no wrap
        --no-wrap     --wrap=0


Help Sections
-------------

Help text is divided by section into files located in a "helpfile" subdirectory in the same directory as the command they describe, named after the file plus subtopic, section type, language, and file format, separated by periods.  The command may include periods, but no other component may.

    /path/to/file
    /path/to/helpfile/file.help.en.txt
    /path/to/helpfile/file.version.en.txt

If not present and the command is a symlink, it is recursively resolved until either the sections are found or the command is no longer a symlink:

    $ which example
    /path/to/example
    $ ls /path/to/helpfile/example.help.en.txt
    ls: /path/to/helpfile/example.help.en.txt: No such file or directory
    $ readlink /path/to/example
    ../../another/path/another-name
    $ ls /path/to/../../another/path/helpfile/another-name.help.en.txt
    /path/to/../../another/path/helpfile/another-name.help.en.txt

If the sections are still not found, internal sections are searched.


Subtopics
---------

A helpfile may have subtopics.  These are in directories named after the parents.

    /path/to/file
    /path/to/helpfile/file.help.en.txt
    /path/to/helpfile/file/subtopic.help.en.txt
    /path/to/helpfile/file/subtopic/subsubtopic.help.en.txt

Internal sections cannot have subtopics.


Section Locales
---------------

The desired locale is the first non-empty environment variable of either $LC\_ALL, $LC\_CTYPE, or $LANG.  This value must start with "language[\_country]", where language is two ASCII letters and country, if present, is also two ASCII letters.  This value is converted to lowercase.  Help files are searched to find the first file matching "language\_country", "language", or "en".

Internal sections do not specify a locale and cannot be localized.


Internal Sections
-----------------

A script whose first line starts with "#!" may have internal sections.  These must be immediately following that first line in consecutive lines which all start with "#".  A section header starts with "#." followed by the section name, and following lines are in that section.  A section line starting with "##" is excluded, starting with "# " is included, and either blank or starting with anything else ends the section.

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

Internal sections are for convenient self-contained scripts, but separate section files are preferred.


Comparison to Manpages
----------------------

Manpages should contain more depth than help text and should be preferred when an overview is insufficient.  Help may even default to "man 1 COMMAND" when help text is not found.  However, help text is easier to specify, better fulfills its role, does not require installation, and isn't closely tied to terminal capabilities or restrictions.


Text Requirements
-----------------

- text MUST be encoded in UTF-8 without a byte-order mark
- text SHOULD be encoded in ASCII if convenient
- text MUST NOT be hard-wrapped, it will be wrapped according to the user's preferences and current display (which may not be a terminal)
- text MUST NOT contain markup or leading/trailing blank lines
- text lines MUST end with a newline, including the last line
- text MUST NOT contain control characters (besides newline), including carriage return and tab
- for help sections, a synopsis of the command (including the command name) MUST be at the top, eg. "command FOO [BAR..]", followed by a blank line if text follows
- for version sections, the version string MUST be the first line followed by a blank line if text follows
    - the version string SHOULD be formatted according to semver.org
    - whether version strings may be compared according to semver.org is OPTIONAL
    - if unknown or missing, the version string MUST NOT be blank and SHOULD be "unknown"


Future Ideas
------------

- alternative implementations should be possible, and every feature must keep that in mind
    - allow for user config such as whether to use a pager or another file viewer (such as a browser), or to fallback to man (or something else)
- how should subtopics treat help synopsis and version string (--short)?
- should $PATH lookup not be default?
    - how does this interact with possibly replacing man? eg. does "help /dev/XX" use "/dev/helpfile"?
- will not support dynamic subtopics, where code must be executed to determine the final command (eg. hg and git aliases in .hgrc and .git/config)
- more formats than plain text, such as Markdown
    - probably only one more format, considering alternative implementations must support it too, and Markdown may not be it
- machine-readable specification for options and arguments to aid command completion
    - more general "properties" specification? (such as documentation URL) but quickly getting more complex than required
- internal sections could specify subtopic and locale through the section header
    - these must be the only metadata in the section header, and corresponds to the filename of a separate section
    - inline sections are exactly equivalent to the contents of a separate section with specific line munging for comments (prefixing "#" for comments and blank lines or "# " otherwise)
