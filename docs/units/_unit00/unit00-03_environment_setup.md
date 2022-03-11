---
title: "Setting up the Working Environment"
toc: true
header:
  image: /assets/images/working-environment.png
  caption: "[Image](https://pixabay.com/de/users/joffi-1229850/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=1359136) [joffi via Pixabay](https://pixabay.com/de/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=1359136)"
  
---

We value freedom of choice as an important good but giving our long-term experience freedom of choice comes to an end when we talk about the mandatory working environment for this course. The reason for this is simple: you work with assignments and a piece of code written on the laptop of person A should run basically without any changes on the computer of person B. The situation gets nastier if you should test some code of a peer or if the instructors would like to run your script on their own system. Hence, let's save everybody's time and focus on the things which are really important. Once the course is finished, feel free to use any working environment structure you like.




## Setup of the environment in R

Setting up a working or project environment usually requires the definition and creation of different folder paths, the loading of necessary R packages and additional functions that are "always" needed. If additional software like GIS should also be accessible, respective binaries and software environments must be linked, too.
Setting up a working or project environment requires the definition of different folder paths and the loading of necessary R packages and additional functions. If additional software like GIS should also be accessible, respective binaries and software environments must be linked, too.

For setting up a project environment, one can use a set-up script and the `envimaR` package. 

## Coping with different absolute paths across different computers
The biggest problem when it comes to cross-computer project path environments is not the project environment itself, i.e. the subfolders of your project *relative* to your project folder but the *absolute* path to your project folder. Imagine you agreed on a project folder called mpg-envinsys-plygrnd. On the laptop of person A, the absolute path to the root folder might be `C:\Users\UserA\University\mpg-envinsys-plygrnd` while on the external hard disk of person B it might be `X:\university_courses\mpg-envinsys-plygrnd`. If you want to read from your data subfolder you have to change the absolute directory path depending on which computer the script is running. Not good.

One solution to this problem is to agree on a common structure *relative* to your individual R home directory using symbolic links (Linux flavor systems) or junctions (Windows flavor systems).
{: .notice--success}

Example: Create a top-level folder name which hosts all of your student projects. For example, this folder is called `edu`. Within `edu`, the project folder of this course is called `mpg-envinsys-plygrnd`. Create the folder `edu` anywhere you want, e.g. at `D:\stuff\important\edu`. Then create a so called symbolic link to this folder within your R home directory i.e. the directory where `path.expand("~")` points to.

To create this link on Windows flavor systems, start a command prompt window (e.g. press [Windows], type "CMD", press [Return]) and change the directory of the command prompt window to your R home directory which is `C:\Users\your-user-name\Documents` by default. Then use the `mklink \J` command to create the symbolic link. In summary and given the paths above:

```yaml
cd C:\Users\your-user-name\Documents
mklink /J edu D:\stuff\important\edu
```

On Linux flavor systems, the R home directory is your home directory by default, i.e. `/home/your-user-name/`. If you create your edu folder on `/media/memory/interesting/edu`, the symbolic link can be created using your bash environment:
```yaml
cd /home/<your-user-name>/
ln -s /media/memory/interesting/edu edu
```
Now one can access the `edu` folder on both the windows and the Linux example via the home directory, i.e. `~/edu/`. All problems solved.


While this will work on all of your private computers, it will not work on the ones in the University's computer lab. To handle that as smooth as possible, you can use the functionality of the envimaR package which allows to set defaults for all computers but one special type which is handled differently. 
{: .notice--info}

## Basic workflow of the setup procedure


There are two aspects we would draw your attention to:
1. Define everything once not twice: It is a good idea to define your project
folder structure, the required libraries and other settings in one place. A simple
approach would be a setup script, i.e. something like `000_setup.R` that is sourced
into each control or analysis script of your project.
1. Use the same project folder structure in a team: Use the same folder structure
relative to a fixed starting point across all computers and devices of your team.
As a starting point, a folder in your home directory  (e.g. `~/edu`) is a good choice.
Within this folder you can directly store your projects or use a symlink to any
other place on your storage devices. 1


### Special considerations at our computer labs
The R home directory on the computers in the labs at Marburg University points to
the network drive `H:/Documents`. For storage space reasons, you could not use
this drive for symlinks. Therefore, you have to define an alternative starting point
for your project folders to work with.

To handle your project relative on all computers using something beneath your 
home directory `~` as a starting point except on one type of computer (i.e. the
ones in the University computer labs), we recommend to use the functionality of
the `envimaR` package.



### Preliminary consideration
Again f the course we assume that 
* your local starting point is `~/edu`,
* your project is called `mpg-envinsys-plygrnd`,
* your git repository is called `name_of_github_team_repository`,
* your setup script is called `000_setup.R` and is located at `name_of_github_team_repository/src`.

Please install the `envimaR` package from GitHub.

```r
devtools::install_github("envima/envimaR")
```




## Template for setup script

First of all copy and paste the following code in a file named `000_setup.R`. You may use it as a tempate for a basic setup script which is called in the beginning of each analysis script you start. 

```r
# Set project specific subfolders
projectDirList   = c("data/",                # data folders the following are obligatory but you may add more
                    "data/auxdata/",  
                    "data/aerial/org/",
                    "data/lidar/org/",
                    "data/lidar/",
                    "data/grass/",
                    "data/lidar/level1/",
                    "data/lidar/level2/",
                    "data/lidar/level0/",
                    "data/data_mof", 
                    "data/tmp/",
                    "run/",                # folder for runtime data storage
                    "log/",                # logging
                    "src/",                # source code
                    "doc/",                # documentation  
                    "name_of_github_team_repository/src/",   # source code github
                    "name_of_github_team_repository/doc/")   # markdown etc.  github
# Set libraries
libs = c("link2GI", "raster", "rgdal", "sp")


# Automatically set root direcory, folder structure and load libraries
envrmt = createEnvi(root_folder = "~/edu/mpg-envinsys-plygrnd", 
                    folders = project_folders, 
                    path_prefix = "path_", libs = libs,
                    alt_env_id = "COMPUTERNAME", alt_env_value = "PCRZP",
                    alt_env_root_folder = "F:\\BEN\\edu")
                    
# For the `raster` package set the temp directory to the one of  project folders

rasterOptions(tmpdir = envrmt$path_tmp)

```

### Automated procedure

To source the setup script, you should reference its location in the same manner 
as any other folder/file. Therefore, include something like this into the header
of your control or analysis scripts.

```r
# Source setup script ----------------------------------------------------------
library(envimaR)
root_folder = alternativeEnvi(root_folder = "~/edu/mpg-envinsys-plygrnd", 
                              alt_env_id = "COMPUTERNAME",
                              alt_env_value = "PCRZP", 
                              alt_env_root_folder = "F:\\BEN\\edu")
source(file.path(root_folder, "name_of_github_team_repository/src/000_setup.R"))
```

One can now access the respective subfolders using the list entries of the variable `envrmt`. The entries are named according to the subfolder names with the prefix "path_".

```r
print(envrmt$path_data_mof)
```

```
## [1] "C:/Users/tnauss/Documents/edu/mpg-envinsys-plygrnd/data/data_mof"
```




### Linking more GI/RS Desktop Software 
If required, on can now go on in the setup script and link selected GIS software etc. using the link2GI package. This generaly starts looking for the respective installations (in case one does not know all the details). 

```r
# Link GIS software ------------------------------------------------------------
# link2GI has been loaded by the script

# Find GRASS installations
grass_path = findGRASS()

# Find SAGA installations
saga_path = findSAGA()

# Find OTB installations
otb_path = findOTB()
```




This setup is a bit more than a guideline, it is a rule. Read it, learn it, live it and have a nice ride.  
{: .notice--danger}

## More Information

For a more advanced and automated project setup have also a look at the [extended setup spotlight]({{ site.baseurl }}{% link _unit05/unit05-02_extended_setup.md %}). 

For a bsic control script which may be useful as a template for a supporting structure of your script have a look at the spotlight [Best Practice Scripts & Functions in R]({{ site.baseurl }}{% link _unit05/unit05-05_best_scripting.md %})

