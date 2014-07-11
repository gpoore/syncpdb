# Synchronized Python Debugger (syncpdb)

## Overview

Provides a wrapper for pdb that synchronizes code line numbers with the line
numbers of a document from which the code was extracted.  This allows pdb to
be used more effectively with literate programming-type systems.  The wrapper
was initially created to work with PythonTeX, which allows Python code 
entered within a LaTeX document to be executed.  In that case, syncpdb makes 
possible debugging in which both the code line numbers, and the corresponding
line numbers in the LaTeX document, are displayed.

All pdb commands function normally.  In addition, commands that take a line 
number or filename:lineno as an argument will also take these same values 
with a percent symbol (`%`) prefix.  If the percent symbol is present, then 
syncpdb interprets the filename and line number as referring to the document, 
rather than to the code that is executed.  It will translate the filename and 
line number to the corresponding code equivalents, and then pass these to the 
standard pdb internals.  For example, the pdb command `list 50` would list 
the code that is being executed, centered around line 50.  syncpdb allows the 
command `list %10`, which would list the code that is being executed, 
centered around the code that came from line 10 in the main document.  (If no 
file name is given, then the main document is assumed.)  If the code instead 
came from an inputed file `input.tex`, then `list %input.tex:10` could be 
used.


## Technical details

The synchronization is accomplished via a synchronization file with
the extension .syncdb.  It should be located in the same directory as the 
code it synchronizes, and should have the same name as the code, with the 
addition of the .syncdb extension.  For example, `code.py` would have
`code.py.syncdb`.  Currently, the .syncdb must be encoded in UTF8.  The file 
has the following format.  For each chunk of code extracted from a document 
for execution, the file contains a line with the following information:
    
    <code filename>,<code lineno>,<doc filename>,<doc lineno>,<chunk length>
    
The first line of the file must be
    
    <main code filename>,,<main doc filename>,,
    
All code filenames should be given relative to the main code filename.

The .syncdb format is thus a comma-separated value (csv) format.  The 
elements are defined as follows:

* `<code filename>`:  The name of the code file in which the current chunk
  of user code is inserted.  This should be double-quoted if it contains 
  commas.
* `<code lineno>`:  The line of the executed code on which the current chunk 
  of user code begins.
* `<doc filename>`:  The name of the document file from which the current 
  chunk of user code was extracted.  This should be double-quoted if it 
  contains commas.
* `<doc lineno>`:  The line number in the document file where the chunk of 
  user code begins.
* `<chunk length>`:  The length of the chunk of code (number of lines).

This information is sufficient for calculating the relationship of each line 
in the code that is executed (and that originated in the document, not a
template) with a line in the document from which the code was extracted.

As an example, suppose that a document main.tex contains 10 lines of Python 
code, beginning on line 50, that are to be executed.  When this code is 
inserted into a template for execution, it begins on line 75 of the code that 
is actually executed.  In this case, the .syncdb file would contain the 
following information for this chunk of code.
    
    code.py,75,main.tex,50,10
    
  
While the .syncdb format is currently only intended for simple literate 
programming-type systems, in the future it may be extended for more complex 
cases in which a chunk of code may be substituted into another chunk, perhaps 
with variable replacement.  In such cases, the `<doc filename>, <doc lineno>,` 
may be repeated for each location with a connection to a given line of code, 
allowing a complete traceback of the code's origin to be assembled.  The 
rightmost `<doc filename>, <doc lineno>,` should be the most specific of all
such pairs.


## pdb and bdb
  
This code is based on pdb.py and bdb.py from the Python standard library.
It subclasses Pdb(), overwriting a number of methods to provide 
synchronization between the code that is executed and the file from which it
was extracted.  It also provides a number functions adapted from pdb.py to
govern execution.  Most of the modifications to the pdb and bdb sources are
wrapped in the pair of comments `# SPdb` and `# /SPdb`.

This code is compatible with both Python 2 and Python 3.  It is based on the
pdb.py and bdb.py from Python 2.7.5 and Python 3.3.2.  Several minor 
modifications were required to get the code from both sources to play nicely
within the same file.


## License

Licensed under the BSD 3-Clause License

Copyright (c) 2014, Geoffrey M. Poore
