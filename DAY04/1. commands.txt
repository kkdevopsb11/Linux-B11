umask [user mask]
—----------------------
            umask  default file	default dir
—-----------------------------------------------
root          0022        644              755
normal        0022        644              755
  RHEL 9.x —-> umask value is same

Base permissions for file is:                0666
                                          (-)0022
                                       —-------------
                                               644
                                      —-----------------

Base permissions for dir is:                 0777
                                          (-)0022
                                       —-------------
                                               755
                                      —-----------------

r —> read  →4
w —> write → 2
x —> execute →1

create one dir and file —> explain the permissions

How to change the umask value?
only root user
umask →display umask value
umask 222 

NOTE: permissions are not going to change for previous files and directories.



to understand more
===================

Understanding File Permissions
In Linux, each file and directory has three types of permissions:

Read (r): Permission to read the file or list the directory.
Write (w): Permission to modify the file or add/remove files in a directory.
Execute (x): Permission to execute a file or access a directory.
Permissions can be assigned to three types of users:

User (u): The owner of the file.
Group (g): The group associated with the file.
Others (o): All other users.
Permission Representation
Permissions can be represented in two ways:

Symbolic Notation:

r: Read
w: Write
x: Execute
-: No permission
Example: -rwxr-xr-- indicates:

User has read, write, and execute permissions.
Group has read and execute permissions.
Others have read permission.
Numeric (Octal) Notation:

Permissions can also be represented as a three-digit number.
Each permission is represented by a binary digit:
Read = 4
Write = 2
Execute = 1
Example: rwx = 4 + 2 + 1 = 7, r-x = 4 + 0 + 1 = 5, r-- = 4 + 0 + 0 = 4





chmod
—-------
The chmod command in Linux is used to change the permissions of files and directories. It stands for "change mode" and allows users to define who can read, write, or execute a file.



user  group  others
rwx	   r_x   rw_


chmod 000 devops.txt

cat devops.txt  → to see


chmod 444 devops.txt
    or
chmod +r devops.txt


chmod 766 devops.txt
     or 
chmod u+rwx,go+rw devops.txt


chmod 777 devops.txt
   or
chmod ugo+rwx devops.txt


chmod u+x file.txt  # Adds execute permission for the user
chmod g+w file.txt  # Adds write permission for the group


chmod o-r file.txt  # Removes read permission for others
chmod u-w file.txt  # Removes write permission for the user


chmod u=rwx,g=rx,o=r file.txt  # Sets permissions explicitly


chmod -v 644 file.txt --> what changes done it will show.




NOTE: user has write permission , But unable to open why? pls check the read permmission


Note : if you want you want to remove use [ - ]


IQ] what is the diff between 400 and ugo+r




Directories
—-----------
chmod 444 DevOps/ [Only parent dir permissions will change]
chmod ugo+x DevOps/


ls -lrth DevOps/

chmod -R 444 DevOps/ → Sub directories will change

 Q: 777 [numbers will override]
     ugo+rwx [ will not override]

chown  [ONLY ROOT USER can used]
======

The chown command in Linux is used to change the ownership of files and directories. The name stands for "change owner." This command allows you to specify a new owner (and optionally a new group) for one or more files or directories.


chown root devops.txt   → Run from Root

 sudo su -  —> pointing to root user home dir
 sudo su      —> pointing to the normal user only

what is the root user home directory?

/ —> parent dir or root dir
/home/root  —> in home normal users will store
/root  —> root user home directory


sudo chown root devops.txt
sudo chown -R root DevOps

chown root *.java




chgrp [only root users]
======

chgrp wheel devops.txt

cat /etc/group  —> all the groups available

chgrp -R wheel DevOps

chown ec2-user:ec2-user devops.txt --> both at a time change for file
chown ec2-user:ec2-user DevOps  --> both at a time change for dir





file [check the what type of file]
====
file demo.txt
file test.txt



diff command
============


echo 
—----
echo hello java
echo hello        java
echo "hello       java"
