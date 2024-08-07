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

## Running Jobs

### Interactive Jobs

```
salloc --account=bcweeks0 --mem-per-cpu=10GB --cpus-per-task=1
```

### Using R

R can be loaded with `module load R`.  There are some libraries pre-installed as modules that I basically load every time I want to use R - thus, I run the folliwing in all my slurm scripts and immediately after starting an interactive job:

```
module load R
module load Rgeospatial
module load Rtidyverse
```


#### TMUX
**TODO**

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

## Storage

The home directory (`~` from the login node) caps out at 80GB.  At least for me that's (somehow??!!) not enought.  We also have 10TB on Turbo.  That is directly accessible from the cluster at `/nfs/turbo/seas-bcweeksturbo`.  So to get to that go to:

```
cd /nfs/turbo/seas-bcweeksturbo
```
#### TODO:  Scott Add Weeks Lab directory structure and standards here.

## Reports and Account Status

### See full job names

The basic way to see all your running jobs is: `squeue --me`.  Unfortunately, this abbreviates the job name column too much...  This bit of code is an ugly way to fix it:

``` 
squeue --format="%.18i %.9P %.30j %.8u %.8T %.10M %.9l %.6D %R" --me
```

You can change the numbers to add/subtract from column character limits...

#### TODO:  I think there's a way to put something like this as an alias in `.bashrc`...

### Checking personal Storage

This checks how much space is used in your home directory on Great Lakes (there is way more space on Turbo)

```
home-quota -u [uniqname]
home-quota -u syanco
```

### Checking CPU Usage

The command below will report total CPU minutes used by user:

```
sreport cluster UserUtilizationByAccount account=bcweeks0
```
