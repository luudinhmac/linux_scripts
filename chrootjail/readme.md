# Restrict SSH User Access to Certain Directory Using Chrooted Jail
## Step 1: Create SSH Chroot Jail
```sh
mkdir -p /home/test/
```
For an interactive session, this requires at least a shell, commonly sh, and basic /dev nodes such as null, zero, stdin, stdout, stderr, and tty devices:
```sh
 ls -l /dev/{null,zero,stdin,stdout,stderr,random,tty}
```
create the /dev files as follows using the mknod command. In the command below, the -m flag is used to specify the file permissions bits, c means character file and the two numbers are major and minor numbers that the files point to.

```sh
mkdir -p /home/test/dev/  
cd /home/test/dev/
mknod -m 666 null c 1 3
mknod -m 666 tty c 5 0
mknod -m 666 zero c 1 5
mknod -m 666 random c 1 8
```
set the appropriate permission on the chroot jail. Note that the chroot jail and its subdirectories and subfiles must be owned by the root user, and not writable by any normal user or group:
```sh
chown root:root /home/test
chmod 0755 /home/test
ls -ld /home/test
```
## Step 2: Setup Interactive Shell for SSH Chroot Jail
create the bin directory and then copy the /bin/bash files into the bin directory
```bash
mkdir -p /home/test/bin
cp -v /bin/bash /home/test/bin/
```
identify the bash required for shared libs, as below, and copy them into the lib directory:

```sh
ldd /bin/bash
mkdir -p /home/test/lib64
cp -v /lib64/{libtinfo.so.5,libdl.so.2,libc.so.6,ld-linux-x86-64.so.2} /home/test/lib64/
```


## Step 3: Create and Configure SSH User
```sh
useradd mac
passwd mac
```
Create the chroot jail general configurations directory, /home/test/etc and copy the updated account files (/etc/passwd and /etc/group) into this directory:
```sh
mkdir /home/test/etc
cp -vf /etc/{passwd,group} /home/test/etc/
```
## Step 4: Configure SSH to Use Chroot Jail
```sh
vi /etc/ssh/sshd_config
```
and add/modify the lines below in the file.

```sh
# define username to apply chroot jail to
Match User tecmint
# specify chroot jail
ChrootDirectory /home/test
```
```sh
systemctl restart sshd
```

## Step 5: Testing SSH with Chroot Jail
```sh
ssh mac@localhost
```

## Step 6. Create SSH User’s Home Directory and Add Linux Commands
```sh
mkdir -p /home/test/home/mac
chown -R mac:mac /home/test/home/mac
chmod -R 0700 /home/test/home/mac
```
install a few user commands such as ls, date, and mkdir in the bin directory
```sh
cp -v /bin/ls /home/test/bin/
cp -v /bin/date /home/test/bin/
cp -v /bin/mkdir /home/test/bin/
```
check the shared libraries for the commands above and move them into the chrooted jail libraries directory:
```sh
ldd /bin/ls
cp -v /lib64/{libselinux.so.1,libcap.so.2,libacl.so.1,libc.so.6,libpcre.so.1,libdl.so.2,ld-linux-x86-64.so.2,libattr.so.1,libpthread.so.0} /home/test/lib64/
```
## Step 7. Testing SFTP with Chroot Jail
Add the line below in the /etc/ssh/sshd_config file
```sh
# Enable sftp to chrooted jail
ForceCommand internal-sftp
```
```sh
systemctl restart sshd
```
test using SSH, and you’ll get the following error:
```sh
root@mac:/home/mac/Desktop/test# ssh mac@10.0.0.25
mac@10.0.0.25's password: 
This service allows sftp connections only.
Connection to 10.0.0.25 closed.
```
Try using SFTP as follows:
```sh
sftp mac@10.0.0.25
```
```
root@mac:/home/mac/Desktop/test# sftp mac@10.0.0.25
mac@10.0.0.25's password: 
Connected to 10.0.0.25.
sftp> ls
a  
sftp> mkdir uploads
sftp> ls -l
drwxr-xr-x    2 mac      mac             6 Mar  8 02:06 a
drwxr-xr-x    2 mac      mac             6 Mar  8 07:49 uploads
```

# Create chrootjail with script
 ## execute command to create chrootjail
[chrootjail.sh](https://github.com/luudinhmac/linux_scripts/blob/master/chrootjail/chrootjail.sh)
./chroot.sh <path-to-command1> <path-to-command2> ... <path-to-command-n>

 example:
```sh
 ./chroot.sh /bin/{ls,cat,echo,rm,bash} /usr/bin/vi /etc/hosts
 ```
 ## Create group chrootjail
 ```sh
 sudo groupadd chrootjail
 ```
 ```sh
 cat <<EOF | tee >> /etc/ssh/sshd_config
 Match group chrootjail
            ChrootDirectory /var/chroot/
 EOF
 
systemctl restart sshd

# add user test to group chrootjail
sudo adduser test chrootjail
```
## test chrootjail
```
ssh test@localhost
```
