# Guide: Using `conda`, `mamba`, and `boa` for reproducible package management

This is a living document describing my current thinking and methods for reproducible package management. The basic goal is to use `conda` (or `conda`-like systems) to control computing environments. This has several benefits:  
*  Manages and resolves conflicts and dependencies - this is especially useful fo workflows that span languages/programs.  
*  Creates a portable description of the computing environment used for code execution ensuring reproducibility of results (and future functionality of code as new package versions are released).  
*  Portable environment description is also helpful for e.g., sharing commonly needed tools within teams or creating a consistent environments between local environments and cloud-based computing resources such as research clusters.

*Note:* The notes below assume you are working on Linux, MacOS, or Windows Subsystem for Linux (WSL).  There are (probably) ways to do this effectively on native Windows or with a Bash emmulator-type program in Windows (thing cygwin or Git Bash) but I don't know about them...  I run a WSL on a Windows machine -- I tried using Bash emmulators and found them clunky and in need of some pretty hacky solutions.  Conversely, WSl has been (thus far) very useful.  (I will post a guide/notes on WSL for researhc computing at some point.)

## Basics

We are going to use a combination of `conda`, `mamba`, and `boa`.  The latter two are just wrappers/replacements to `conda` that speed it up.

## Installation

### Without Conda Previously Installed
I think this is preferred? It at least has `conda-forge` as default and only channel (see below).  In either case, if you don't have `anaconda` or `miniconda` already installed on your machine, start here:

```bash
wget https://github.com/conda-forge/miniforge/releases/latest/download/Mambaforge-$(uname)-$(uname -m).sh

bash Mambaforge-$(uname)-$(uname -m).sh
```

### With Conda Previously Installed
If you already have a `conda` installation, just use it to get `mamba` - this will be useful, for example, on an HPC.

On a local machine it's best to run this from your base environment (N.B. You don't want to put much of anything in your base environment, but this is an exception to that rule.) The base environment might be kind of a weird thing on an HPC (at least it is on the one I use), so I create a "dummy" base environment and install mamba there (see HPC instructions below).

```bash
conda install mamba
```

## Basic Operations
Fundamentally, these all work just like basic `conda` commands - we just replace some with mamba or boa for some of the calls.  If you choose to just use regular old `conda`/`miniconda` you can swap in `conda` anywhere you see e.g., `mamba` in almost all cases. 

**Make a new environment**

```bash
mamba create -n [env name]
```

**"Typical"** package installation:

```bash
mamba install [package] -n [environment]
```

**Activate an environment:**

```bash
conda activate [environment]
```

**Adding channels:**

Channels are places that softwar packages are stored for you to download and install in your environments. Depending on how you installed `conda`/`mamba` you may have access to some or all the channels you will want.  You can put your preferred channels in the list that `conda`/`mamba` will always search with `conda config --add channels [channel]`.  I recommend you run the following - I've included my own personal channel where I store some common packages for movement ecology research that I was unable to find on other channels.

```bash
conda config --add channels defaults
conda config --add channels bioconda
conda config --add channels conda-forge
Conda config --add channels syanco
```

**Find packages:**
`mamba` comes with some nice tools (not availbale in `conda`) for searching for packages and seeing dependencies.

To search for a package:  
```bash
mamba repoquery search xtensor
```

Similarly, to find the dependencies:  
```bash
mamba repoquery depends xtensor 
```

**Build Packages from CRAN (or other)***
Some packages are not available on any channels - but they “can” be built from CRAN (I primarily use R so CRAN is what matters to me - there are some other pssoble sources for this process like CTAN). I quote "can" becuase I've found this to be finnicky sometimes - so only use it when absolutely necessary.  That said, since switching to `boa` from basic `conda` I've encountered far fewer glitches.

To build packages, we want the `boa` package installed in our base environment, so from (base) (or your personal base environment on an HPC):

```bash
mamba install boa
```

To get a package from CRAN, the follow steps are required

```bash
conda skeleton cran [pkg] # download the package recipe
conda mambabuild r-[pkg] # build the package locally
mamba install -n [environment] -c local r-[pkg] # install it in the environment
```

**To upload to your channel:** 
(You should! Play nice, upload your packages so others can find them...)

First install the anaconda-client package that allows you to conect to your [Anaconda.org](https://anaconda.org) account. If you don't have such an account, go make one - it's free.

Again, do this into (base) (i think)

```bash
mamba install anaconda-client
```

Then to upload, to your channel

```bash
anaconda login  # (only need to run this once per session)
anaconda upload [tarball]
```
Regarding the [tarball]: when you finish the `conda mambabuild` step above, at the end, it will display the location of the build (and even suggest this command to you!) it will end in ".tar.bz2"

Alternatively you can always upload to your channel by running

```bash
conda config --set anaconda_upload yes
```

## Sharing Environments

One of the key benefits of this system is the ability to share your environment.  Three key use cases of this are:  
1.  Including a description of your environment when publically sharing code (key for reproducibility and open science);  
2.  Sharing your environment directly with colleagues; and,  
3.  using the same environment across platforms for a single analysis (e.g., a local machin *and* and HPC).  

**Export and environment:**
To export a description of your environment, first activate the environment you want to export.

```bash
conda activate [environment]
```

Then run:

```bash
conda env export > environment.yml
```

Note, for me, this exports the environment with *very* specific versions and builds which may be overprescriptive. For some purposes and depending on, for example, the channels available to the downstream user this may result in the inability to re-build the environment on another machine.  For now, you can either create the environment description by hand or manually edit (i.e., just delete the versions and builds after some or all of the package names).

#TODO: See if there's a more elegant way to do this.

**Build an environment from file:**
There are a few options here.  If you are using just `conda`, all you have to do is:

```bash
conda env create --file [filename] # typically [filename] is just "environment.yml"
```

But we want to gain some speed up from `mamba` though so two options here:

This mostly just wraps around the `conda` version but uses `mamba` for the solve, so is faster
```bash
mamba env create -f [filename]
```
This method takes an extra command but uses `mamba` for both the solve and the install, so should be fastest -- this is the method I prefer (at least for now).
```bash
mamba create -n [env name] # make a new "empty" environment
mamba env update -n [env name] --file [filename] #update it from the file (e.g., environment.yml)
```

## HPC
I initially got into all of this because the HPC at my institution uses `miniconda` to install and manage packages (then I saw the light of the other reproducibility/open science benefits).  In my limited expereince, there are idiosyncracies to each HPC, so this workflow is a little more general though I've included specific code that I was able to use. 

**Entirely on the HPC:**
If you want to create and manage environments on your HPC, well, just follow the steps above - should work just fine. The only note is that i found the concept of a base conda environment on the HPC to be note quite the same as my local machin, and it casued errors.  So I created a new environment called "my_base" and I treat that as my base environment on the HPC.

**Mixing local and HPC:**
If you have an environment from your local machine or from a colleage or team, and need to use it on an HPC, here are the steps:

1. Transfer the environment.yml to your home directory.  For me this looks like:
```bash
# run locally
scp [environment.yml] [cluster]:[directory path]
```

2. Install `mamba` on your HPC
```bash
# run on HPC
conda create -n my_base # create alt base env
conda activate my_base # activate base environment
conda install mamba # install mamba
```

3.  Create the shared environment on the HPC
```bash
# run on HPC
mamba create -n [env name] # make a new "empty" environment
mamba env update -n [env name] --file [filename] #update it from the file (e.g., environment.yml)
```

Now you're up and running!

## Endnote

If you found errors, have questions, or would like to see something added, please reach out.  You can contact me directly at scott.yanco [at] yale.ed.  This site was built with GitHub, so you can also (I think...) use the features of the repository to do so.  The repo for this site lives here: [https://github.com/syanco/scientific_computing_refs](https://github.com/syanco/scientific_computing_refs)  You can file an issue if you found a problem.  If you want to contribute you could also use a [Pull Request to suggest an edit](https://docs.github.com/en/github/collaborating-with-pull-requests)!

If you found this helpful, please let me know!  If/when appropriate, you can also cite this work!