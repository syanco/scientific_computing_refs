# U Mich Great Lakes Tips and Tricks

Working document to stash some helpful tips and tricks for working in the University of Michigan Great Lakes ARC (HPC).


## Login

### Nickname for the login

Typing the entire SSH address is a pain and it's hard to remember, especially if you regularly use SSH connections to more than one HPC system.  But you can nickname the host inside your ssh configuration file in the following way:

You need to edit the file `~/.ssh/config` (on your local machine).  I find this easiest to do from the command line with the command `nano ~/.ssh/config`.  If you don't have a config file, just make one (`nano` will do that)

Add the following text to your config file:

```
Host greatlakes
     Hostname greatlakes.arc-ts.umich.edu
     User [uniquename]
```


## File transfers

There are multiple ways to get files on and off the HPC - I pretty much only ever use `scp` - it's easy, customizable, you can even build file transfers into your workflow if you're fancy like that.

The format is super simple:

```
scp [from] [to]
```

Which ever is the remote location (on the HPC) needs a `[hostname]:` at the start, then just the filepath. See examples below.

### SCP Address

The SCP address is a PITA to remember (`[uniqname]@greatlakes-xfer.arc-ts.umich.edu`) so stashing it here for ease of access...

```         
syanco@greatlakes-xfer.arc-ts.umich.edu
```

You can also set up a nickname following the same steps as for ssh.  I don't know if you can name it the same thing and didn't feel like spending time figuring it out so i didn't...

I added the following to my `~/.ssh/config` file:

```
Host gltransfer
     hostname greatlakes-xfer.arc-ts.umich.edu
     User syanco
```

This allows, for example:

```
scp gltransfer:[from] [to]
```

Rather than:

```
scp syanco@greatlakes-xfer.arc-ts.umich.edu:[from] [to]
```


### Transfer entire directories

For some unknown reason the traditions `scp -r [from] [to]` format gives an error about 'path canonicalization failure'. It has something to do with new versions of SCP utilities. simply adding the `-O` flag after the `-r` fixes it. For example:

```         
scp -r -O [from] [to]

scp -r -O syanco@greatlakes-xfer.arc-ts.umich.edu:./project/groundbreaking_science_project .
```
