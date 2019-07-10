---
title: "Artemis, local, and the Classic RDS"
teaching: 20
objectives:
- "Learn to move data from Artemis to Classic RDS"
- "Learn to move data from local to Classic RDS"
keypoints:
- "Use smbclient"
- "Make a local network mount"
---

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
  <img src="{{ page.root }}/fig/pic11_smb.png" style="margin:10px;width:600px"/>
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


<br>
