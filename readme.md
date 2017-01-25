Help
====

Help displays an overview of a command's purpose, options, version, and other closely related information.  Help replaces the "options" of --help and --version and does not require executing files, which may be non-functional or untrusted.

Help text is not a substitute for manual pages or other documentation.  Help text is looked up by filename; a name without a slash follows a $PATH lookup.  The text itself can either be specified in a file or inside a self-contained script.


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


Help Sections
-------------

Help text is divided by section into files located in a "helpfile" subdirectory in the same directory as the command they describe, named after the command, section, language, and file format, separated by periods.  The command may include periods, but no other component may.

    /path/to/command
    /path/to/helpfile/command.help.en.txt
    /path/to/helpfile/command.version.en.txt

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


Section Locales
---------------

The desired locale is the first non-empty environment variable of either $LC\_ALL, $LC\_CTYPE, or $LANG.  This value must start with "language[\_country]", where language is two ASCII letters and country, if present, is also two ASCII letters.  This value is converted to lowercase.  Help files are searched to find the first file matching "language\_country", "language", or "en".

Internal sections do not specify a locale and cannot be localized.


Internal Sections
-----------------

A script whose first line starts with "#!" may have internal sections.  These must be immediately following that first line in consecutive lines which all start with "#".  A section header starts with "#." followed by the section name.  After a header and before the next header are lines in that section.  A section line starting with "##" is excluded, starting with "# " is included, and either blank or starting with anything else ends the section.

Help recognizes two sections: help and version.

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

Manpages should contain more depth than help text and should be preferred when an overview is insufficient.  Help may even default to "man 1 COMMAND" when help text is not found.  However, help text is easier to specify, better fulfills its role, does not require installation, and isn't closely tied to terminal capabilities or restrictions.


Future Ideas
------------

- alternative implementations should be possible, and every feature must keep that in mind
    - allow for user config such as whether to use a pager or another file viewer (such as a browser), or to fallback to man (or something else)
- sections for "compound commands" (eg. hg, git)
    - will not support dynamic compound commands, where code must be executed to determine the final command (eg. hg and git aliases in .hgrc and .git/config)
- more formats than plain text, such as Markdown
    - probably only one more format, considering alternative implementations must support it too, and Markdown may not be it
- machine-readable specification for options and arguments to aid command completion
    - more general "properties" specification? (such as documentation URL) but quickly getting more complex than required
