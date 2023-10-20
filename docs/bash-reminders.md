# Bash Reminders

Reminders for code snippets and approaches for doing things in bash that I always forget... minimal and poorly organized for now...

## Bulk File Actions

### Delete a set of files

I often need to bulk delete, move, copy (etc.) many files.  This is of course pretty easy to do if you want the **entire** directory (just put `*` as the file arg to imply "everything in the current directory).  But I can never remember how to do this using regex to select certain files (super useful for example when I have an output directory containing updated model outputs and i want to select by the date string in the filename...).  Here's how:

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

### Count files

Sometimes it's useful to count the number of files in a directory.  This is pretty simple  - just list the files then pipe to the ord count command witht he lines option (this will list all the files and then count the lines of that list:

```
ls | wc -l
```

We can also count a subset of files in a directory by matching this code with the find code we did above **(but get rid of the `-delete` flag...)**.

```
find . -type f -name '*_5-20.rdata' | wc -l
```

## Arrays stored as variables

I always forget how to store things in an array and - especially - how to check the array to make sure it contains everything I think it does.

*Storing to array*:

The only thing to remember here really is how to get the output of some function into an array - it takes a weird combo of parenthesis and `$`:

```
x=($([function]))
```

*Printing and array*:

Let's make a simple array to play with:

```
x=(1 2 3 4 5)
```

Running `echo $x` will simply return "1".  Which is objectively a weird default behavior.  Instead we need to use subsetting (square braces) to ask for the entire array explcitly using `@` in the index.  But wait!  If you run `echo $x[@]` it will return "1[@]".  For reasons I do not understand, we need to wrap the x and the subset in curly braces...
```
echo ${x[@]}
```
Will print out th whole array.  Kind of  alot of work to print the contents of a named variable if you ask me.

*Check the length of an arry*:

To get the length of an array we take the code a bove and put a `#` in front of the array name:

```
echo ${#x[@]}
```


## Edit PATH

Sometimes you need to add directories to PATH so that you can call filles in that location as command line executables.  Do that with:

```
export PATH=$PATH:[filepath to add]
```

An example of this, on Mac - homebrew installed R goes to export `/usr/local/bin` which apparently not on path, so to add that, I ran:

```
export PATH=$PATH:/usr/local/bin
```
