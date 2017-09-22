# Relion on Sherlock

A relion image is on Sherlock for users with SBGrid accounts that need to use the software.
You can use (or copy the image for more control / safekeeping) from the shared folder:

```
cp /scratch/users/vsochat/sbgrid_stanford-skiniotis--relion_20170921.img /scratch/users/<username>
```

It is highly recommended to put singularity images on scratch, as they are large.

```
du -h sbgrid_stanford-skiniotis--relion_20170921.img 
6.6G	sbgrid_stanford-skiniotis--relion_20170921.img
```

## Interacting with the image

Whether you are on an interactive node or running a job, to interact with an image
you first need to load the Singularity software:

```
module load singularity
The following have been reloaded with a version change:
  1) singularity/2.3 => singularity/2.3.1
```

The latest on sherlock is version 2.3.1, which should be sufficient to use the image. It's 
good practice, if you think you might forget, to add this to your bash profile so that it
is automatically loaded and ready to go when you log in or run a job.

### Shell
The most basic "let me poke around" interactive mode is shell. You can shell inside your image:

```
CONTAINER=/scratch/users/vsochat/share/sbgrid_stanford-skiniotis--relion_20170921.img
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
 SBGrid installation last updated: 2017-09-07
 Please submit bug reports and help requests to:       <bugs@sbgrid.org>  or
                                                       <http://sbgrid.org/bugs>
********************************************************************************

 Capsule Status: Active
       For additional information visit https://sbgrid.org/wiki/capsules
********************************************************************************
Singularity sbgrid_stanford-skiniotis--relion_20170921.img:/scratch/users/vsochat/share> 
```

If you ls where you shell, you will notice you are in the same directory as before! A few things
about the sherlock setup for singularity:

 - `$PWD`,`/tmp`, `$HOME`, `/scratch`,`/local-scratch`, `/share`, and `oak` are automatically mounted. You shouldn't have trouble reading and writing data from the image via these mount (remember the image itself is read only)
 - importantly, the software under `/programs` is... there period! :)
 - `relion` is added to the path, at `/programs/x86_64-linux/system/sbgrid_bin/relion`

### Environment
You can grep the environment to look at various versions, for example cuda:

```
env | grep CUDA
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

If you have any questions or issues, especially with library loading (this was development
feature at time of 2.3.1) please post an issue at https://www.github.com/singularityware/singularity/issues
