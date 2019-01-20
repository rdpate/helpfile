Future Ideas
====

* alternative implementations should be possible, and every feature must keep that in mind
    * allow for user config such as whether to use a pager or another file viewer (such as a browser), or to fallback to man (or something else)
        * or just mandate the user have their own "help" command?
* how widely applicable is --synopsis for help sections?
    * subtopics?
    * non-commands?
    * what alternative synopsis will always make sense?
* separate command to list subtopics?  help-list?
* will not support dynamic subtopics, where code must be executed to determine the final command (eg. as with hg and git aliases in .hgrc and .git/config)
* support Markdown, or a subset
    * desirable: links, anchors, italic, bold, underline
    * probably only one more format, considering alternative implementations must support it too
    * human-readable as text strongly preferred, allows treating as plain text if format isn't parsed
* machine-readable specification for options and arguments to aid command completion
    * more general "properties" specification? (such as documentation URL) but quickly getting more complex than required
    * "command" section?
* internal sections could specify subtopic, language, and format through the section header
    * these must be the only metadata in the section header, and correspond to the filename of a separate section
    * "#.help.LANG.FORMAT SUBTOPIC.."?
    * "#.help /SUBTOPIC.LANG.FORMAT"?
    * "#/SUBTOPIC.help.LANG.FORMAT"?
    * inline sections are exactly equivalent to the contents of a separate section with specific line munging for comments (prefixing "#" for comments and blank lines or "# " otherwise)


Why use directories for topics and subtopics?
----

Instead of placing topic (and subtopics) into directory structure, they could be included in filenames, eg. ".../help/topic[.subtopic].lang.format".  However, this complicates and lengthens lookup with additional permutations on possible locations.  Employing directories unifies files as subtopics of their parent directories without necessitating existing files to use their corresponding help files.  Help for the "root" of a package is thus naturally placed at the topmost level, eg. "help -v .".
