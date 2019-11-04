.. _curriculum_spelling:

Curriculum Spelling
=======================================

Automated Spelling Checks
-------------------------

When initiating a PR, one of the checks that is run will check for
spelling errors in the guide or notebook. We use the aspell library for
validating the spelling of words.  If the check fails, the document will
need to be reviewed and the spelling corrected or added to the dictionary.

This check will notate the file and any words that were misspelled
in the file.  All spelling errors will need to be corrected for the
PR to be reviewed.

What is not checked
-------------------

Any words that occur in code blocks are ignored.  When discussing a
variable, reference the variable using the single ticks to have the
check bypass that variable.  An example is `myCustomVar`, this variable
will not be included in the spelling check.  For scripts, wrapping them
in the markdown triple ticks or the html `<pre></pre>` blocks will
also be excluded from the spelling checks.

Adding to the Dictionary
------------------------

In some cases the Aspell dictionary may not have the word you are
trying to use.  This will happen more often on industry words.
First, validate the correct spelling of this word, then you can
add the word to the `.aspell.en.pws` file. This list does not
need to be in alphabetical order, but many words are listed in
that order for ease.  Capitalization does matter and numbers or
hyphens are not allowed. The first line references the file
format, the language and an estimated number of words in the file.
The number does not need to be updated as it is only used as a
reference.  For more information on custom dictionaries, please
see `Aspell man <http://aspell.net/man-html/Format-of-the-Personal-and-Replacement-Dictionaries.html>`_

