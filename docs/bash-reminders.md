# Bash Reminders

Reminders for code snippets and approaches for doing things in bash that I always forget... minimal and poorly organized for now...

## Bulk File Actions

I often need to bulk delete, move, copy (etc.) many files.  This is of course pretty easy to do if you want the **entire** directory (just put `*` as the file arg to imply "everything in the current directory).  But I can never remember how to do this using regex to select certainf iles (super useful for example when I have an output directory containing updated model outputs and i want to select by the date string in the filename...).  Here's how:

You can use the find command to do this pretty easily - you can read this as 'find files here with filename matching some string and delete them. 
```
find . -type f -name '[pattern to match]' -delete   
```

I usually use this to match based on the end of a file name - either to delete files with a certain extension or with a certain date string - to do this use the '\*' in the regex as in this example:

```
find . -type f -name '*_5-20.rdata' -delete   
```

Finally you can reverse this to negtate the match and instead delete everything *except those files that match the pattern* just by putting `!` in front of the `-name` flag:

```
find . -type f ! -name '[pattern to match]' -delete 
```
