---
title: "Building R Package using Command Line"
date: 2018-02-10T05:04:40-06:00
draft: false
tags: ["R"]
categories: ["Tools"]
---

This post describes the process of building an R package without using RStudio. This can come in handy for anyone who is using vim (and of course Nvim-R!) as a code editor for their development work. Alright, let's get moving. We are going to create a small package offering a new ggplot theme to demonstrate the process.

### Step 0: Install the required packages

{{< highlight R >}}
install.packages(c("devtools", "roxygen2", "testthat", "knitr"))
library(devtools)
has_devel()
{{< /highlight >}}
The command `has_devel()` basically checks that everything is installed as expected. At the end of its output, you should see `TRUE` if everything goes alright.

### Step 1: Create a package directory
First, we are going to create the package directory where everything is going to reside related to this package. This will also generate some boilerplate files inside the package directory that we will edit soon. We will use the following command:


{{< highlight R >}}
create("~/projects/themeSimple")
{{< /highlight >}}

In this case, I have created a directory called 'themeSimple' inside 'projects' directory. `themeSimple` will also be the name of our package. It is generally not recommended to use both cases for the package name since it becomes difficult for folks to remember or type correctly when installing. 

### Step 2: Edit the DESCRIPTION file
[Here](https://github.com/pchhina/themeSimple/blob/master/DESCRIPTION) is how my file looks like after editing.

### Step 3: git it out!

{{< highlight R >}}
$git init
$git add .
$git commit 
{{< /highlight >}}

### Step 4: Add any raw data if needed
In our case we do not need any data. In case you do, add any raw data in `inst/extdata` directory. Add any processed data (.RData or .rda) in `/data` directory.

### Step 5: Create .Rmd for analysis
It is good idea to keep notes in .Rmd file so it can be converted into vignette later if needed. In that case, save the file in `/vignettes` directory. To have `devtools` automatically create this for you and populate it with boilerplate content, use:

{{< highlight R >}}
use_vignette("analysis")
{{< /highlight >}}

In the above example, it will create 'analysis.Rmd' in `/vignettes` directory.

### Step 6: Carry out the work
Carry out the analysis, write cool functions. Add all functions in /R directory. I am using a function created in [this]({{< ref "creating_theme.md" >}}) blog post.

### Step 7: Update documentation
{{< highlight R >}}
document()
{{< /highlight >}}

This uses Roxygen2 to create/update NAMESPACE file and .Rd files in `/man` directory for each function.

### Step 8: Build package
{{< highlight R >}}
build("themeSimple")
{{< /highlight >}}

This creates a \*.tar.gz file that can be installed on any platform.

### Step 9: git it!
{{< highlight R >}}
$git add <files>
$git commit
{{< /highlight >}}

### Step 10: Check it
{{< highlight R >}}
check()
{{< /highlight >}}
This runs all sorts of checks on the contents (including examples) of the package, and gives warnings/errors when it finds that things aren't right.    

### Step 11: Install locally to test

{{< highlight R >}}
install(build_vignettes = TRUE)
{{< /highlight >}}

### Step 12: Upload on Github
First, create a new repo on Github. Follow instructions in [this]({{< ref "git-commands.md" >}}) blog if needed.

Connect local repo to Github repo:

{{< highlight R >}}
$git remote add origin <github repo name>
{{< /highlight >}}

Push everything on Github:

{{< highlight R >}}
$git push -u origin master
{{< /highlight >}}

### Step 13: Test Github package

Remove package from your installation:

{{< highlight R >}}
remove.packages("themeSimple")
{{< /highlight >}}

Install package from Github:

{{< highlight R >}}
install_github("pchhina/themeSimple")
{{< /highlight >}}

### Step 14: Test

Test your new package and you are done!!

### Care to learn more?
- This [book](http://r-pkgs.had.co.nz/) on R packages by Hadley Wickham is a great resource.
- This [post](https://hilaryparker.com/2014/04/29/writing-an-r-package-from-scratch/) by Hillary Parker is a good resource for those in a hurry.
- Karl Broman has put together a more comprehensive [tutorial](http://kbroman.org/pkg_primer/).
