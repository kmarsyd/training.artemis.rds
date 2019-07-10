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




<br>
