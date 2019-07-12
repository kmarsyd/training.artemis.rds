---
title: "Transfer from RCOS to Artemis HPC"
teaching: 30
objectives:
- "Learn to move data between Artemis HPC and RCOS"
- "Learn to make dependant compute jobs"
- "Learn to connect directly to RCOS"
keypoints:
- "Use cp, mv, rsync"
- "Use dt-scripts"
- "Use -W depend flags"
- "Connect from local to RCOS"
---

# Transfer from RCOS to Artemis HPC 

You now have scripts, raw data and a reference file. Your collaborator has informed you that the reference files also needs associated  ‘index files’ and that they have previously prepared these and saved them on RCOS. The Artemis compute nodes can’t read directly from RCOS. They may appear seamless, but they are actually different physical machines situated at opposite ends of Sydney! So these files need to be copied from RCOS to Artemis. 

What methods could you use to copy from RCOS to Artemis? Which do you think is the best method?

 * ```cp``` is easy to use, but is subject to the same issues as running any command in the foreground: it is subject to termination if your session is lost 

 * ```mv``` same as above, but worse, since ‘mv’ removes the file on the source so you may lose your data altogether in the event of a prematurely aborted transfer! 
 
Your collaborator has placed the reference files needed in the ```Training``` RCOS space in a directory called ```Dog_disease/Ref```. To get these files into your current working directory for today, you could run the following unsafe command:  
~~~
 cp -v /rds/PRJ-Training/Dog_disease/Ref/* /project/Training/<yourname>
~~~
{: .bash}


# The dt-script

 * ```cp``` or ```rsync``` within a PBS script is safe and recommended, particularly ```rsync``` as it has more flexibility, eg retain timestamps and file permissions, "sync" a directory (only copy what is new or changed on the destination compared to the source), and more.  
 
There is an even easier way! Our HPC’s management team have created a handy utility for us called ```dt-script```. This script (full path ```/usr/local/bin/dt-script``` for the interested) is a ‘wrapper’ for running ```rsync``` in a PBS script. It enables you to perform data transfers as a “one-liner” without the need for a script and the transfer is actually submitted to the data transfer nodes of the cluster (and not running in the foreground of your current terminal).

The minimum syntax to run dt-script is: 

~~~
dt-script -P <Project> -f <from> -t <to> 
~~~
{: .bash}

You can include additional arguments, such as: 

| ```-N <jobname>``` | default is ‘dt-script' |
|```-w <walltime>``` | default is 24 hours |
|```-ncpus <ncpus>``` | default is 2 |
|``` -m <mem>``` | default is 2gb |

and others. Run ```dt-script -h``` for a full list.

Your collaborator has placed the reference files needed in the ```Training``` RCOS space in a directory called ```Dog_disease/Ref```. To get these files safely into your current working directory for today, run the following command: 

~~~
dt-script -P Training -f /rds/PRJ-Training/Dog_disease/Ref/ -t /project/Training/<yourDirectoryName>/ 
~~~
{: .bash}

Once your transfer job has completed, confirm that you have the expected files (eg ```ls -l```). 

Note that you have the same 3 job log files, like any normal job submitted to Artemis with ```qsub```. 


# Dependent compute jobs

You are just about to submit the analysis job when your supervisor calls you with the bad news that you’ve downloaded the wrong data! The sample you downloaded from the web has already been processed, the new data is actually on RCOS! Since it’s getting late on a Friday and you really just want to submit this analysis and go home without having to wait for another download to complete, you use a handy feature of PBS Pro that allows you to ‘chain’ jobs to other jobs, so that one only submits when the dependent job finishes with an **exit status** of 0.

Jobs are chained together by including the following on your qsub command line: 

```
-W depend=afterok:<joBID> 
```

A job can depend on multiple jobs, using the syntax: 

```
-W depend=afterok:<joBID1>:<jobID2>:<jobID3> 
```

The process is to submit the first job, then submit the second job including the jobID of the first job as the argument to ‘depend’. We will now use this method to transfer the correct raw data from RCOS to Artemis, and submit the analysis job to run when the data transfer completes. 

The data we need is in the ```PRJ-Training``` RCOS space in a directory called ```/rds/PRJ-Training/Dog_disease/Data```. The script we want to run when the data is transferred is called ```map.pbs```. You don’t need to make any changes to this script, although you may wish to change the job name to clearly identify your job in the queue.  View the script with the ```cat``` command: 

~~~
cat map.pbs 
~~~
{: .bash}

Note the line ```cd $PBS_O_WORKDIR``` . The analysis command ```bwa mem``` looks for the input files in the present working directory, and since PBS considers this to be your HOME directory, you need to ensure bwa can find the right files. This can be done using ```cd``` to your present working directory as we have here, or else you need to include full pathnames to the input and output files. There are pros and cons to each method, but we have used ```cd``` for simplicity here. So ensure you are in the right directory when you submit the job to PBS, or else it will fail! 

Submit the data transfer job: 

~~~
dt-script -N getData<yourName> -P Training -ncpus 1 -m 1gb -w 00:10:00 -f /rds/PRJ-Training/Dog_disease/Data/ -t `pwd` 
~~~
{: .bash}


Output:
~~~
459974.pbstraining
~~~
{: .output}

Note the job ID, you will feed this into the next command line: 

~~~
qsub -W depend=afterok:<jobID> map.pbs 
~~~
{: .bash}
 

```pwd``` command means "print working directory". You could type the full pathname of your directory, but this is easier 😊. ***Dt-script does not accept the ```.``` abbreviation for ‘this directory’. ***


Check job status: 

~~~
qstat -u ict_hpctrain<N> 
~~~
{: .bash}

Note that the map job shows status ```H``` (held). It will remain ‘held’ until the ‘getData’ job completes. If ‘getData’ completes successfully, ‘map’ job will enter the queue (it may run right away and show ```R``` status, or it may queue for a while and show ```Q``` status). If ‘getData’ does not complete successfully (ie returns a non-zero exit status) then ‘map’ will not run at all.

<figure>
  <img src="{{ page.root }}/fig/pic08_depend.png" style="margin:10px;width:600px"/>
</figure><br>

Pretty neat right? A very flexible and time-saving trick! 


# Transfer from Artemis HPC to RCOS: Backup project! 

It is great practice to routinely back up your Artemis project (whether the data be in /project or /scratch) to RCOS on a regular basis (ideally, after every day you have worked on the project). This is made so simple with dt-script, there is no excuse not to 😊 

Today you want to back up your working directory, to your personal location on RCOS. Since many of you do/will have many colleagues working in the same Artemis project as you, this may be more applicable at times than backing up an entire project folder. You do not need to first create the destination directory on RCOS, because ‘rsync’ makes precise use of trailing slashes in directory paths to make or not make new directory levels. If we leave the trailing slash from the source (our working directory) it will automatically create that directory for us on the destination if that directory doesn’t exist. See ```man rsync``` for more information on this behavior. 

To back up your working directory for today, make sure you are situated within the directory containing the data to backup (so that you can make use of `pwd` shortcut, which omits the trailing slash), then run: 

~~~
dt-script -P Training -N backup<yourName> -f `pwd` -t /rds/PRJ-Training 
~~~
{: .bash}

Backup sorted! 

Check using ```ls /rds/PRJ-Training``` that your data has been backed up on RCOS. Note that the time stamps are preserved. 

# Transfer from RCOS to local with FileZilla 

Just as we accessed Artemis through the FileZilla interface earlier, we can likewise access the RCOS server. The hostname for RCOS server is 

From external to university network:
```
rcos.sydney.edu.au 
```

From internal to university network:
```
rcos-int.sydney.edu.au  
```

Open FileZilla and in the host field enter: 
~~~
sftp://rcos-int.sydney.edu.au 
~~~
{: .bash}
 

The username, password and port are as before. Trust the host if prompted. 

Once successfully connected, notice that the ```Remote site``` indicates you are situated in ```/home/<yourUnikey>```. 

Please DO NOT USE THIS LOCATION TO STORE DATA! Change to the correct directory by entering in the ‘Remote site’ field: 

~~~
/rds/PRJ-Training 
~~~
{: .bash}

Expand out the directory tree to find your directory that was created earlier when you used ‘dt-script’ to backup. If you wanted to transfer any of these files to your local computer, you could do so by either drag and drop/right click download/double click. 

<figure>
  <img src="{{ page.root }}/fig/pic10_rcosfilezilla.png" style="margin:10px;width:600px"/>
</figure><br>


<br>
