# Relion on Sherlock

A relion image (`sbgrid_stanford-skiniotis--relion_20171026.img`) is on Sherlock for users with SBGrid accounts that need to use the software. You can use (or copy the image for more control / safekeeping) from the shared folder. The image comes with the following software installed:

```
Installed packages:

ctf@20130307
ctf@20140303
ctf@20140609
ctffind-4@4.0.15
ctffind-4@4.0.16
ctffind-4@4.0.17
ctffind-4@4.1.5
ctffind-4@4.1.8
ctffind-4@4.1.8_intel16
cuda@7.5
cuda@8.0
gctf@0.50
gctf@1.06
motioncor2@1.0.0
motioncor2@20160822
motioncor2@20161019
motioncor2@20170130
motioncorr@2.1
relion@1.3
relion@1.4
relion@1.4b
relion@1.4-randomphase3d
relion@2.0.3_cu7.5
relion@2.0.3_cu8.0
relion@2.0.3_SP
relion@2.0.4_cu7.5
relion@2.0.4_cu8.0
relion@2.0.6_cu7.5
relion@2.0.6_cu8.0
relion@2.1b1_cu7.5
relion@2.1b1_cu8.0
relion@2.1b1_SP
relion@2.1-beta_cu7.5
relion@2.1-beta_cu8.0
resmap@1.1.4
sbgrid-installer@1.0.660
sbgrid-installer@latest
summovie@1.0.2
unblur@1.0_150529
unblur@1.0.2
```

If you are moving this image from its provided location, it is highly recommended to put singularity images on scratch, as they are large. We are looking to create a smaller image size (of the format "squashfs") to help this a bit.

```
du -h sbgrid_stanford-skiniotis--relion_20171026.img 
13G	sbgrid_stanford-skiniotis--relion_20171026.img
```

Yes, fat.

## Interacting with the image

Whether you are on an interactive node or running a job, to interact with an image
you first need to load the Singularity software:

```
The following have been reloaded with a version change:
  1) singularity/2.3 => singularity/2.4
```

You'll notice that Sherlock 2 is now supporting Singularity 2.4, hooray! It's 
good practice, if you think you might forget, to add this to your bash profile so that it
is automatically loaded and ready to go when you log in or run a job.

### Shell
The most basic "let me poke around" interactive mode is shell. You can shell inside your image:

```
CONTAINER=sbgrid_stanford-skiniotis--relion_20171026.img
singularity shell $CONTAINER
```

You will likely see a familiar interface:

```
[vsochat@sh-ln02 login! /scratch/users/vsochat/share]$ singularity shell $CONTAINER
Singularity: Invoking an interactive shell within container...

********************************************************************************
                  Software Support by SBGrid (www.sbgrid.org)
********************************************************************************
                              SBGrid Announcements
 The "sbgrid" command has been renamed to "sbgrid-info".
********************************************************************************
 Your use of the applications contained in the /programs  directory constitutes
 acceptance of  the terms of the SBGrid License Agreement included  in the file
 /programs/share/LICENSE.  The applications  distributed by SBGrid are licensed
 exclusively to member laboratories of the SBGrid Consortium.
******************************************************************************** 
 SBGrid was developed with support from its members, Harvard Medical School,    
 HHMI, and NSF. If use of SBGrid compiled software was an important element     
 in your publication, please include the following reference in your work:      
                                                                                      
 Software used in the project was installed and configured by SBGrid.                   
 cite: eLife 2013;2:e01456, Collaboration gets the most out of software.                
********************************************************************************
 SBGrid installation last updated: 2017-10-25
 Please submit bug reports and help requests to:       <bugs@sbgrid.org>  or
                                                       <http://sbgrid.org/bugs>
********************************************************************************
 Capsule Status: Active
       For additional information visit https://sbgrid.org/wiki/capsules
********************************************************************************
Singularity sbgrid_stanford-skiniotis--relion_20171026.img:~> 

```

If you `ls` where you shell, you will notice you are in the same directory as before! A few things
about the sherlock setup for singularity:

 - `$PWD`,`/tmp`, `$HOME`, `/scratch`,`/local-scratch`, `/share`, and `/oak` are automatically mounted. You shouldn't have trouble reading and writing data from the image via these mount (remember the image itself is read only)
 - Importantly, the software under `/programs` is... there period! :)
 - `relion` is added to the path, at `/programs/x86_64-linux/system/sbgrid_bin/relion`


### Exec
The most common use case is executing a specific command to the image, and for this you
want to use `exec`. For example, to list the content of programs in the image (from the host):

```
singularity exec $CONTAINER ls -1 /programs
legacy.cshrc
legacy.shrc
sbgrid.cshrc
sbgrid.shrc
share
x86_64-linux
```

### Environment
You can grep the environment to look at various versions, for example cuda:

```
singularity exec $CONTAINER env | grep CUDA
CUDA_X=8.0
```

For libraries (eg, graphics) that are needed from the host, you will want to load them and run the container 
with the `--nv` option:

```
module load cuda
singularity exec --nv $CONTAINER
```

To run a contained environment (not grabbing from the host):

```
singularity exec --cleanenv $CONTAINER env
```

### Relion
Look at details for installation of relion

```
# (from inside the container, use exec to do from outside)
sbgrid-info -L relion
  Version information for: /programs/x86_64-linux/relion

Default version:                    2.0.3_cu8.0
In-use version:                     2.0.3_cu8.0
Other available versions:           2.1b1_cu8.0 2.1b1_cu7.5 2.1-beta_cu8.0 2.1-beta_cu7.5 \ 
Overrides use this shell variable:  RELION_X

No program directory by that name found in /programs/i386-linux.
No program directory by that name found in /programs/i386-mac.
No program directory by that name found in /programs/powermac.
```

### Job / Submission
The best way to think of a Singularity image is like any other executable. So if you
want to run a step of a pipeline using the image, you just run the image as a command,
and no special binding is needed for Sherlock's standard storage spaces (eg, `/scratch`) because 
they are bound automatically. So to submit with a batch job you generally want to:

 - load the singularity module
 - exec a command to the image - it's good practice to use full paths to executables in the rare case that there is an executable on the host found first.
 - direct inputs / outputs to standard locations

Essentially, the only thing that would change in a standard job submission file is to have the executable / command be performed by the image. 

If you have any questions or issues, especially with library loading (this was development
feature at time of 2.3.1) please post an issue at https://www.github.com/singularityware/singularity/issues
