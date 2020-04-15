# SDSC HPC User Training
 
## Running Jupyter Notebooks On Comet using SSH connection
### by Mary Thomas, mpthomas @ ucsd.edu

Goal: Learn how to run/edit a jupyter notebook on comet using Python notebooks 
### Note: 
* For this notebook, we will use Singularity containers to run a notebook on Comet. 
* For details, see the Comet User Guide notes on [Using Singularity on Comet](https://www.sdsc.edu/support/user_guides/comet.html#singularity)
* 04/16/2020: Comet supported singularity images are located in 
   * /share/apps/gpu/singularity 
   * /share/apps/compute/singularity 

* Log onto comet.sdsc.edu  
```
ssh -Y -l <username> comet.sdsc.edu
```
You will be logged on to Comet, and you will see this type of message:
```
Last login: Wed Apr 15 12:04:06 2020 from 76.176.117.51
Rocks 7.0 (Manzanita)
Profile built 12:32 03-Dec-2019

Kickstarted 13:47 03-Dec-2019
                                                                       
                      WELCOME TO 
      __________________  __  _______________
        -----/ ____/ __ \/  |/  / ____/_  __/
          --/ /   / / / / /|_/ / __/   / /
           / /___/ /_/ / /  / / /___  / /
           \____/\____/_/  /_/_____/ /_/
###############################################################################
NOTICE:
The Comet login nodes are not to be used for running processing tasks.
This includes running Jupyter notebooks and the like.  All processing
jobs should be submitted as jobs to the batch scheduler.  If you don’t
know how to do that see the Comet user guide
https://www.sdsc.edu/support/user_guides/comet.html#running.
Any tasks found running on the login nodes in violation of this policy
 may be terminated immediately and the responsible user locked out of
the system until they contact user services.
###############################################################################
(base) [mthomas@comet-ln2:~] hostname
comet-ln2.sdsc.edu
(base) [mthomas@comet-ln2:~] 
```
Notice the name of the login node that you are on.  Next: 
* create a test directory for testing a notebook, or ```cd``` into one you have already created
* Clone this repository (developed by Bob Sinkovits):   
    -- [https://github.com/sinkovit/PythonSeries](https://github.com/sinkovit/PythonSeries)
```
git clone git@github.com:sinkovit/PythonSeries.git
```

Obtain an interactive node using the `debug` queue for 30 minutes (you may want to increase that), with a `bash` shell:
```
(base) [mthomas@comet-ln2:~] hostname
comet-ln2.sdsc.edu
(base) [mthomas@comet-ln3:~/comet101] srun --pty --nodes=1 --ntasks-per-node=24 -p debug -t 00:30:00 --wait 0 /bin/bash
srun: job 32652512 queued and waiting for resources
srun: job 32652512 has been allocated resources
(base) [mthomas@comet-14-03:~/comet101] 
```
Wait for your node to be allocated - this can take some time (10's of minutes if the system is busy).
Once your node has been allocated, notice that you are now located on a different host
```
(base) [mthomas@comet-14-03:~/comet101] hostname
comet-14-03.sdsc.edu
```
The command we need to run to fire up a notebook is the `ipython` command. You can use one of two versions:
* the version supported and maintained by the Comet user support staff
* the version you install locally using the [Anaconda](https://www.anaconda.com/) software system

There are pros and cons for choosing your own installation (you can customize it and maintain it), or choosing the system version -- it will always work, and will be updated as other software dependencies change. For these examples, we will use the _system supported_ version.

The next step is to load the singularity module that knows about system wide supported jupyter notebooks and then load a singularity image. Note that by default the version of `ipython` is a local installation in my home directory.
```
(base) [mthomas@comet-14-03:~/comet101] which ipython
~/miniconda3/bin/ipython
(base) [mthomas@comet-14-03:~/comet101] module load singularity
(base) [mthomas@comet-14-03:~/comet101] which ipython
~/miniconda3/bin/ipython
```
Once I launch the Singularity container, the `ipython` command location changes, and we are now using the version supported by system admins.
```
(base) [mthomas@comet-14-03:~/comet101] singularity shell /share/apps/compute/singularity/images/jupyter/jupyter-cpu.simg
Singularity> which ipython
/opt/anaconda3/bin/ipython
```

You should also notice that you are in a new shell by the new prompt and that your environment is different:
```
Singularity> 
Singularity> ls -al
total 325
drwxr-xr-x 3 mthomas use300    27 Apr 15 12:17 .
drwxr-xr-x 3 mthomas use300     4 Apr 15 12:28 ..
drwxr-xr-x 8 mthomas use300    13 Apr 15 12:17 .git
-rw-r--r-- 1 mthomas use300  1157 Apr 15 12:17 .gitignore
-rw-r--r-- 1 mthomas use300  9427 Apr 15 12:17 Decision trees.ipynb
-rw-r--r-- 1 mthomas use300   399 Apr 15 12:17 Dockerfile
-rw-r--r-- 1 mthomas use300 35147 Apr 15 12:17 LICENSE
-rw-r--r-- 1 mthomas use300  4405 Apr 15 12:17 LaTeX math.ipynb
-rw-r--r-- 1 mthomas use300  3851 Apr 15 12:17 Markdown.ipynb
[snip]
```
If you have a `.bash_profile` script, you might want to run that in order to setup your normal environment.

Check out the PythonSeries/Readme.md file, it will explaing what is in the different notebooks.
Launch the Jupyter Notebook application. Be sure to navigate to a directory where there are notebooks located. 

Note: this application will be running on comet, and you will be given a URL which will connect your local web browser the interactive comet session:
```
ipython notebook --no-browser --ip=`/bin/hostname`
```
This will give you an address which has the comet hostname in it and a token. Something
like:
```
http://comet-14-0-4:8888/?token=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```
You can copy and  paste this URL into your local Web browser. You will see a running Jupyter
notebook and a listing of the notebooks in your directory. From there everything should be working as a regular notebook.

<img src="images/jupyter-notebook-comet-http.png" alt="SSH Connection" width="300px" /

Notes: 
* The token is part of your authentication and should not be shared so don't share it.
* It will go away when you stop the notebook. 
* There is a well-known security flaw in hosting Jupyter Notebooks using this method: note that the URL is only HTTP, not HTTPS. As a result, your notebook connection is hackable. There are solutions, which include hosting the notebook using SSH tunneling, and using the newly developed SDSC Reverse Proxy Server (release date will be in April 2020).

To learn about Python, run the ```Python basics.ipynb```   notebook.
To see an example of remote visualization, run the  ```Matplotlib.ipynb```  notebook!


