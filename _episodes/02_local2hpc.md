---
title: "Transfer from LOCAL to Artemis HPC"
teaching: 10
objectives:
- "Leanr to move data between local machine and remote HPC environemnt"
keypoints:
- "Use scp"
- "Use GUI like Filezilla
---

# Transfer from LOCAL to Artemis HPC

Back to the case study... Since you haven’t run this kind of analysis before, your helpful collaborator has emailed you a ‘tar’ file (packaged directory) of scripts to use to analyse the data. 

This file was emailed to you via Eventbrite. Please download the dogScripts.tar.gz file to your local computer for your email. Don’t unpack it yet. You can also download it from here: https://cloudstor.aarnet.edu.au/plus/s/F5bB2g9Gemn1xMj

This local file needs to be transferred to Artemis HPC. This can be done via a Graphical User Interface (GUI) tool (ie “point and click”) such as [FileZilla](https://filezilla-project.org/) or [WinSCP](https://winscp.net/eng/download.php), or via the command line. You may choose whichever method you prefer. Please note: if you choose to install FileZilla, ensure to UNSELECT the sneaky inclusion of ‘Avast Antivirus’ and ‘Opera Browser’ during the installation!

Note for Windows users: if you would prefer to use command line rather than install a GUI, you need to use a *Nix command line, NOT the Windows command prompt. If you have Windows 10, you can [install Ubuntu](https://tutorials.ubuntu.com/tutorial/tutorial-ubuntu-on-windows#0) from the Microsoft store. Any version of Windows can use [Cygwin](https://www.cygwin.com/).  

Mac users: you can use your Mac terminal app. Keep your Artemis session open, and open a new Mac terminal for your local session. It's often convenient to work with multiple environments open at the same time.


## Instructions for using command line ‘scp’ to copy a local file to Artemis:

First, change into the directory that contains the data you want to transfer (or else you will need to prepend the full pathname in front of the file name). 

The syntax for scp is
~~~
scp <user@host:file> <user@host:to> 
~~~
{: .bash} 

Since the file to be transferred is local, you do not need to include user@host. Run the below command to copy the scripts archive to your working directory on Artemis:

~~~
scp dogScripts.tar.gz  ict_hpctrain<N>@hpc.sydney.edu.au:/project/Training/<yourDirectoryName>
~~~
{: .bash}
 
Then unpack the archive, move the scripts, and delete the archive and empty directory:

~~~
tar -zxvf dogScripts.tar.gz 
mv dogScripts/* . 
rmdir dogScripts 
~~~
{: .bash}


## Instructions for using FileZilla to copy a local file to Artemis:

Open FileZilla. 

In the 'Host' field, enter:
```
sftp://hpc.sydney.edu.au
``` 
In the Username field, enter your training login/unikey:
```
ict_hpctrain<N>
``` 
In the ‘Password’ field, enter the training password.

In the ‘Port’ field, you can leave it blank (default is 22).
Then click ‘Quick connect’. 
You will be prompted about saving passwords, you can choose whether or not to do this. When you are logged on, you should see the following message printed in the top panel
~~~
Status:    Directory listing of "/home/ict_hpctrain<N>" successful
~~~
{: .output}


<figure>
  <img src="{{ page.root }}/fig/pic03_filezilla.PNG" style="margin:10px;width:600px"/>
  <figcaption> The left-most two panels display your local computer’s filesystem. The right two panels are your connection to the Artemis filesystem. You can navigate the directory tree by expanding the yellow folders with your mouse, or typing the full directory pathname into the “Local site” or “Remote site” fields. Note that Windows use ‘\’ and Linux uses ‘/’ to separate directories. 
</figcaption>
</figure><br>


Before initiating the upload to Artemis, open the ‘destination’ location. This will be your working directory for today inside the ‘Training’ project. 

In the ‘Remote site’ field, type:

```
/project/Training/<yourDirectoryName>
```

then press enter. Now that your destination directory is open and ready to receive the upload, locate the ```dogScripts.tar.gz``` file on the left (local) (it may be in your Downloads folder?)

To upload the file to Artemis, you can use any of the following methods:

 * -Double click the file
 * -Right click the file then select ‘Upload’
 * -Drag and drop the file from the left (source) to the right (destination)

As you can see, FileZilla is very simple to use. It can be used just as easily to transfer files from Artemis to your local computer, by simply initiating the transfer from the right (source=Artemis) to the left (destination=local computer). 

Now go back to your terminal connected to Artemis and run the following commands to unpack the archive, move the scripts, and delete the archive and empty directory:

~~~
tar -zxvf dogScripts.tar.gz 
mv dogScripts/* . 
rmdir dogScripts 
~~~
{: .bash}




# Transfer from WEB to Artemis HPC

You need to get a reference file from an online database to use as input for your analysis. A simple way of retrieving files from the web is using the command line utility ‘wget’ (web-get). 

In a web browser, go to [https://ftp.ncbi.nlm.nih.gov/genomes/Canis_lupus_familiaris/CHR_05/](https://ftp.ncbi.nlm.nih.gov/genomes/Canis_lupus_familiaris/CHR_05/) . Right click on the file ```cfa_ref_CanFam3.1_chr5.fa.gz``` then select “Copy link address” or “Copy link location”. 

<figure>
  <img src="{{ page.root }}/fig/pic04_web.png" style="margin:10px;width:600px"/>
</figure><br>

Within your Artemis terminal, enter the ```wget``` command, then after a space, paste the URL:

~~~
wget https://ftp.ncbi.nlm.nih.gov/genomes/Canis_lupus_familiaris/CHR_05/cfa_ref_CanFam3.1_chr5.fa.gz 
~~~
{: .bash}

Unzip the file:

~~~
gunzip cfa_ref_CanFam3.1_chr5.fa.gz 
~~~
{: .bash}


<figure>
  <img src="{{ page.root }}/fig/pic05_wget.png" style="margin:10px;width:600px"/>
</figure><br>

While wget is very handy for small files, it poses some risks for larger files or large collections of files. Can you think of what these might be? 

 * Transfers will be aborted if internet or VPN connection is lost, or if your computer sleeps or is shut down, or if there is a network timeout.
 
 Can you think of a solution to this?
 * Run the transfer using ‘nohup’ or ‘screen’, or submit the transfer to the cluster using ‘qsub’. 
 
 Our next data transfer is a bit larger, so we will use a different method that is more robust.



# Transfer from WEB to Artemis HPC: a safer way with ‘dtq’ 

The raw DNA sequence data for this analysis is hosted online. You could run wget in the foreground as we did for the last data transfer, but we won’t because the files are large. 

We will instead ‘wrap’ the wget commands in a PBS script, as you are now familiar with from this morning's [Introduction to the Artemis HPC course](https://sydney-informatics-hub.github.io/training.artemis.introhpc/). We will submit them to a **dedicated data transfer queue** on Artemis called ***dtq*** which is exempt from fair share. This will be good news to those of you with big datasets to move around 😊 

View the script ```download.pbs``` with the ```cat``` command: 

~~~
cat download.pbs 
~~~
{: .bash}

<figure>
  <img src="{{ page.root }}/fig/pic06_download.png" style="margin:10px;width:600px"/>
</figure><br>

You don’t need to change anything in this script, but note that we have specified to run the job on the ***dtq*** queue using the ```-q``` flag. 

Also note that the data will be downloaded to your current working directory (ie, where you are situated when you run the ```qsub``` command). This is because the script changes you to that directory using the PBS environment variable ***$PBS_O_WORKDIR***. If you are not in the correct directory to receive your download, change there now, then submit the transfer with: 

~~~
qsub download.pbs 
~~~
{: .bash}

Check the status with ```qstat```: 

~~~
qstat -xu <unikey> 
~~~
{: .bash}

or 

~~~
qstat -x <jobID> 
~~~
{: .bash}

Once your transfer job has completed, confirm that you have the expected files (eg ```ls```). 

Note that you have the same 3 job log files, like any normal job submitted to Artemis with ```qsub```. 

<figure>
  <img src="{{ page.root }}/fig/pic07_jobfiles.png" style="margin:10px;width:600px"/>
</figure><br>


# Transfer from RCOS to Artemis HPC: a safer, easier way with ‘dt-script’ 

You now have scripts, raw data and a reference file. Your collaborator has informed you that the reference files also needs associated  ‘index files’ and that they have previously prepared these and saved them on RCOS. The Artemis compute nodes can’t read directly from RCOS. They may appear seamless, but they are actually different physical machines situated at opposite ends of Sydney! So these files need to be copied from RCOS to Artemis. 

What methods could you use to copy from RCOS to Artemis? Which do you think is the best method?

 * ```cp``` is easy to use, but is subject to the same issues as running any command in the foreground: it is subject to termination if your session is lost 

 * ```mv``` same as above, but worse, since ‘mv’ removes the file on the source so you may lose your data altogether in the event of a prematurely aborted transfer! 

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

Your collaborator has placed the reference files needed in the ```Training``` RCOS space in a directory called ```Dog_disease/Ref```. To get these files into your current working directory for today, run the following command: 

~~~
dt-script -P Training -f /rds/PRJ-Training/Dog_disease/Ref/ -t /project/Training/<yourDirectoryName>/ 
~~~
{: .bash}

Once your transfer job has completed, confirm that you have the expected files (eg ```ls -l```). 

Note that you have the same 3 job log files, like any normal job submitted to Artemis with ```qsub```. 




# Transfer from RCOS to Artemis HPC and run a dependent compute job 

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


# Transfer from Artemis to ‘classic RDS’ 

Your collaborator wants you to bring a local copy of the data to the conference next week. You don’t have much space for this massive dataset on your laptop, so you back it up to ‘classic RDS’ so you can easily show the data via your mapped network drive. 

This part will be demonstrated (you do not have to perform this). 

To map ‘classic RDS’ network drive, follow the instructions here. Note that the path will differ depending on when your RDS was created.

To transfer data between Artemis and classic RDS, we use an ftp-like command line tool called ‘smbclient’. FileZilla could be used, however this is not recommended (especially for large files) as the transfer is bottlenecked by your local computer. Smbclient performs a direct transfer. 

In general, to connect with smbclient use the command: 

```
smbclient <path> -U <unikey> -W SHARED 
```

In this case: 

~~~
smbclient //research-data-2.shared.sydney.edu.au/RDS-02 -U ict_hpctrain1 -W SHARED 
~~~
{: .bash}


Note: the different command prompt ```\>``` instead of ```$```. Some Linux commands work (eg ```ls```, ```pwd```, ```cd```, ```mkdir```). Using the ```!``` in front of any normal linux expression will exectue the command on the "local/host" machine, otherwise the commands are executed on the "remote" machine.

By default, ‘prompting’ (the system prompts you between transferring each file and awaits ‘Y’ or ‘N’ input) is ON. By default, ‘recurse’ (recursively copy directories) is OFF. 

Since we want to copy our Output directory to classic RDS, we should turn prompt OFF and recurse ON: 

~~~
prompt off 
recurse on 
~~~
{: .bash}


Now, move into one of our project folders and make a directory for the dataset: 

~~~
cd PRJ-sjkClassic

mkdir Dog_disease 

cd Dog_disease 
~~~
{: .bash}

Then transfer the Output directory and its contents: 

~~~
mput Output 
~~~
{: .bash}


<figure>
  <img src="{{ page.root }}/fig/pic10_smb.png" style="margin:10px;width:600px"/>
</figure><br>


The output data can now be readily accessed via the local file explorer under the mapped drive, without having to save the data onto your local hard drive 😊 

Single files can be transferred from Artemis to classic RDS with ```put```. Single files can be transferred from classic RDS to Artemis with ```get```, or multiple files with ```mget```. To log out of smblicent, enter ```control + C```.


## Mounting ‘classic RDS’ on your local machine.
Dependng on your operating system there are few ways to mount the classic RDS as a network drive.

The steps wor Windoes 10 are:

* Click on This PC from the Desktop.
* On the Computer tab, click on Map network drive in the Network section.
* Choose a drive letter and enter your Classic RDS path: \\research-data-2.shared.sydney.edu.au\RDS-02.
* Enter SHARED\<UniKey> and your password.
* Click Finish.

<figure>
  <img src="{{ page.root }}/fig/pic09_classicmount.png" style="margin:10px;width:600px"/>
</figure><br>

For a full discussion, and mounting instructions for Windows/Mac OSX, and Linux, see here:
https://sydneyuni.atlassian.net/wiki/spaces/RC/pages/229146744/Classic+RDS


# Catch up summary

If you get lost at all, these few lines should get all the data required for any of the steps.

wget -O dogScripts.tar.gz https://cloudstor.aarnet.edu.au/plus/s/F5bB2g9Gemn1xMj/download
tar -zxvf dogScripts.tar.gz 
mv dogScripts/* .

wget https://ftp.ncbi.nlm.nih.gov/genomes/Canis_lupus_familiaris/CHR_05/cfa_ref_CanFam3.1_chr5.fa.gz
gunzip cfa_ref_CanFam3.1_chr5.fa.gz 

# Wrap up 

That completes this afternoon’s session! Thank you all for your attendance and participation. Hopefully now you have an idea of what methods can be used to transfer data between RDS, Artemis HPC, web and your local computer.  

Please feel free to contact us with any questions: [sih.info@sydney.edu.au](mailto:sih.info@sydney.edu.au)

We welcome your feedback, please complete the brief survey: [https://goo.gl/aMJSA7](https://goo.gl/aMJSA7)

***Reminder: Artemis is not backed up!***

<figure>
  <img src="{{ page.root }}/fig/artemisbackup.png" style="margin:10px;width:600px"/>
  <figcaption> Caption 
</figcaption>
</figure><br>




<br>
