# Quick R Tricks!

Collection of quick tips and tricks for stuff I found useful in R.

## Unused packages in a script
Sometimes as a script develops, I find that I've included `library([package])` for functions that subsequently got deleted from my script.  This can be particularly tricky for long scripts or when you're not sure which packages are in the namespace of other packages.  

There's, of course, (usually) no real problem with loading a package that subsquently is never used, but it's messy.  Trying to find an efficient solution to this problem, I cam across the following function from Michael Chirico;

First install his package `funchir` from GitHub
```
devtools::install_github("MichaelChirico/funchir")
```

Then, run the following line to find out which functions in your script come from which package - it will also alert you to packages loaded by the script which have no functions called.

```
funchir::stale_package_check('path_to_script.r')
```
