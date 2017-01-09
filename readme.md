Help
====

Help displays an overview of a command's purpose, options, and other closely related information.  It replaces the --help "option" popular with some programs.

Help text is not a substitute for manual pages or other documentation.  It is only available for commands and thus uses a command name with a $PATH lookup.  The text itself can either be specified in the executable file (for scripts) or in a file alongside the executable.

The text itself has several guidelines:

- prefer ASCII if convenient, otherwise the text must be UTF-8
- do not hard-wrap lines, they will be wrapped according to the user's preferences and current display, which may not be a terminal
- first line is a synopsis of the command including the command name, eg. "command FOO [BAR..]", followed by a blank line
- no markup
- no leading or trailing blank lines
- every line, including the last, should end with a newline
- no control characters (besides newline), including carriage return and tab


Help in Inline Text
-------------------

If the file starts with "#!", then immediately following lines which all start with "#" will be searched.  Help text starts after a "#.help" line.  Subsequent lines are output if they are blank (represented by "#") or start with "# " (which is removed prior).  Subsequent lines starting with "##" are ignored, or any other line ends the section.


Example:

    #!/anything/here
    # this is not help text
    #.help
    # example
    #
    # above line is blank
    #   this line is indented
    ## comment
    #
    # more help text

    this is not help text

Although Help checks for inline text before a separate file, prefer writing help text in separate files whenever equally convenient.  The size, maintenance requirements, and language of the program will guide the decision on which form to use.


Help in Separate File
---------------------

After looking up a file path from a command name, the directory of that file is checked for a "helptext" subdirectory containing a file matching the executable plus ".txt".  For example, if the command is "example" and the file path is "/path/to/example", then help text may be at "/path/to/helptext/example.txt".  If the help text is not present and the last component of the file path is a symbolic link, resolve that link and try again.

Example:

    $ which example
    /path/to/example
    $ less /path/to/helptext/example.txt
    /path/to/helptext/example.txt: No such file or directory
    $ readlink /path/to/example
    ../../another/path/another-name
    $ less /path/to/../../another/path/helptext/another-name.txt
    [...]


Comparison to Manpages
----------------------

Manpages should contain more depth than help text and should be preferred when an overview is insufficient.  Help may even default to "man 1 COMMAND" when help text is not found.  However, help text is easier to specify, better fulfills its role, does not require installation, and isn't closely tied to terminal capabilities.


Future Ideas
------------

- subsections for "compound commands" (eg. hg)
    - support for dynamic compound commands, where code must be executed to determine the final command
- more formats than plain text, such as Markdown
- machine-readable specification for options and arguments to aid command completion
    - more general "properties" specification? (such as documentation URL) but quickly getting more complex than required
