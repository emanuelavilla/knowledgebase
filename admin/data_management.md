# Data managment on O2

As an HMS Core we get storage on O2 for free. We are not the biggest user but we are in the top 10. As such it is smart for us to be good citizens and keep our footprint as low as possible. Ways to reduce our footprint include
## Reduce space used by active consults
* delete redundant files 
  * a special case of this is when you receive raw data as some seq facilities (cough, BPF, cough) will send us both foo.fastq.gz and foo.fastq.bz2 files, we can delete the bz2 files
* run bcbio analyses so temporary work folders are put on scratch 
  * the brute force approach to this is to setup on scratch with everything as a symlink, run the analysis and move the final folder over to the PIs project folder
  * a more elegant way to do this is to set the output folder of the bcbio run to be in the project folder in the PIs folder
* keep project folders tidy, delete things you are no longer using (this is a judgement call and not really enforced but can be an issue once the project is complete)
* compress raw data files
  * bcbio can handle gzipped and bzipped (bz2) files, compress those raw fastqs!

## Get the data out of our main storage area
### Return data from completed analyses to the researcher
* this is a hassle so we typically only do it for larger data folders
* can be by
  1) "sneaker net" - downloading onto a drive and handing it off to the researcher (not preferred)
  2) GLOBUS - https://www.globus.org/ If the researcher sets up a GLOBUIS account, we can work with HMS RC to make their directory available to GLOBUS so the researcher can download it through the web or CL interface. While we haven't tested it, this method has the potential to be the least work for us.
  3) upload to the researcher's server - preferred if access to the server is simple. A good example would be the research data storage that HMS PIs have access to or an FTP server. Passwords, logins and occasionally VPN access are typically required.
  
I recommend avoiding things like Dropbox, Google Drive or Box unless the data is small. They aren't really built for this purpose.

### Archive the data
We have access to standby storage on O2 (/n/standby/cores/bcbio/). standby dir is only accessible from the **transfer** node. For projects that are either too small to bother with returning to the researcher or projects where we think we may want to access the data again, we can tar.gz them and store them here. Leave a symlink in the original directory to allow easy restoration of the project 

**Once you have restored the project, delete the standby file.**

**Once you are finished with the project, rearchive it**

Please don't keep an archived copy of the diretory in two places plus the expanded folder. Duplicated data is wasted space and makes John cry.

## How to decide what to do with the data
With the caveat that every project is different here are some general guidelines to help guide your decision making process.

1) Is the data "large" (>500GB)? As much as possible, we'd prefer to get rid of these ASAP

2) Will you need to access the data and derived files again? If yes, tidy up any unnecessary files and archive
 
 The following points can inform your decision making about how likely we will need to reaccess the data
  * Is the analysis published? If its published, its likely we won't be using it again. Ask the researcher what they want returned to them and delete the rest.
  * If the data is unpublished, did the analysis work? If the data is garbage, we likely won't be using it again. Ask the researcher what they want returned to them and delete the rest.
  * Did the consult end well? If it didn't its likely we won't be using it again. Ask the researcher what they want returned to them (with a time warning in case they don't respond) and delete the rest.
  * Is the consult unique in its approach? i.e. do you think its something we might come back to in the future for other analyses. If yes, tidy it up and archive. If it is something like a standard RNA-seq analysis, we probably don't need it. 
 * How long has it been since the data was accessed? If its older than 2 years, we likely won't be using it again. Ask the researcher what they want returned to them and delete the rest.

## Globus howto (in progress)

### 1. Arrange the transfer with the researcher

#### Warn them about need for xfer 
>Hi everyone, 
>I hope you are all doing well. 
>We need to clear some space on the server and would like to remove your data and analysis files if they are no longer needed. Do you have everything you need from us?
>Let us know if you need anything and we can setup a transfer via O2 or their new Globus data transfer system. 
>If we don't hear back by the 10th of March (i.e. two weeks) we will go ahead and remove the files. 

#### Followup with Globus info

>Hi, 

>I’ll be handling the data transfer.

>Currently, we have your data on HMS RC's O2 server. If you have an account there and appropriate space, we can simply transfer ownership to you there. You can then move it to your directory on O2 and decide whether to keep it on O2 (note that they will start charging for data storage in the summer) or move it to another server/drive.

>Alternatively, we can use the new Globus secure transfer service that HMS offers. It has worked quite well for us to transfer from O2 to external servers and individual laptops/computers. I'm attaching some of their guidance docs on setting up the client and initiating a transfer. We would need you to sign into the system with your Harvard ID first (see below) and give us your Globus user ID (see the “Account” section in the Globus interface) or the email you used to sign in. We'd then share the folder with you to access via Globus and you will receive an email with instructions. 

>**Globus LogIN help**
>Go to https://www.globus.org/ and click on "LogIn/Use your existing organizational login/Harvard University", this will take you to Harvard Key. You can also login with your ecommons ID. If you don’t have Harvard Key you can create a globus id - (https://www.globusid.org/create).

>Please let us know if you have O2 access and if not, please share your Globus ID and we'll set up your data for transfer.


#### Alternate followup script
>HI  – 
 
>With the help of HMS, we’ve been using globus to facilitate transferring the data. (globus.org) It requires you to sign up for a free account and install their client on your system (mac/windows, etc.). We’ll give you access to the data on the HMS cluster from your globus account and globus can transfer it to whatever storage you have access to at MGH, on an external drive, etc. (ie it’s a glorified FTP, file transfer capability and takes care of error recovery & handling etc.)
 
>If this sounds good, let us know your globus account info and we can go from there. 
 
>Thanks, 


**Note, share these two help docs:**

[Globus-Connect-Personal-install-Windows](https://www.dropbox.com/s/aq2g2i06hdctf38/Globus-Connect-Personal-Install-Windows.pdf?dl=1) - setting up the client

[GlobusOneTimeTransfer](https://www.dropbox.com/s/461sxorxsxcoc5e/GlobusOneTimeTransfer.pdf?dl=1) - initiate the data transfer

### Setup the transfer

**On server**
1. Check permissions to make sure you have access to everything 
2. Prepare an archive to transfer (optional)

```bash
tar cf project_dir.tar project_dir
md5sum project_dir.tar > project_dir.tar.md5
```

**On Globus:**

See [SelfServiceCollections](https://www.dropbox.com/s/gyl41z3y0kwe276/SelfServiceCollections.pdf?dl=1) for how to setup a share. 

Search for the HMS-RC endpoint in the file manager and login with your HMS-RC ID

Share the folder with the researcher after searching for them via their Globus ID , email or name.




## Potential issues
### Permissions 
- initiator needs proper permissions to actually share data
### Non-standard characters in filenames
- files whose names contain non-alphanumeric characters (eg. ; ? =) may be rejected by windows machines during transfer
### Changes in files on host or client
- due to Globus monitoring for file integrity, changes that are made to files or directories on either side during transfer can interrupt the transfer


## Notes
* Email from Globus only contains share information, unclear where the researcher learns about the app
* CAn add additional endpoint folders through Preferences/Options on OSX and Windows.
* Console interface or client may not tell you that transfer is complete








