# Basic data science at the command line

Linux command line tools can be used to perform many of the key data cleaning and exploration activities that make up data science workflow.

If you ssh into another server, this is a fast way to take a look at data. It also lets you do quick processing without the overhead of writing a full python program. And parsing big files at the command line can speed up subsequent data processing and reduce memory usage by discarding unnecessary data.

The main tools are grep, awk, sed, tr, and a few others, all of which come with ubuntu.

grep is a utility for finding patterns in a file

sed is a stream editor that lets you find and replace text in a file; tr is a simpler utility for finding and replacing text

awk is a programming language that operates on individual lines in a data stream

Linux also has some commands like sort, head, etc.

I will go through an example and explain what the commands and options are doing as the are used:

*Weather data example*
