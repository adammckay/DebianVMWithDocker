# Debian VM with Docker

The purpose of this guide is to create a working Debian 12 Virtual Machine that runs Docker.  There will be instructions for some basic workability of the VM and creation of various Docker containers to serve as examples.  The guide is not detailed to the precision of walking a complete novice through every single click.  It assumes some proficiency working with computers. 

**Remember that when working with Linux, everything is going to be case-sensitive.**



## Required Software

You will need several different things to follow this guide.



VirtualBox is the application to host Virtual Machines.

[Virtualbox](https://www.virtualbox.org)



Debian 12 is the Linux distribution that we will use as our virtual operating system.

[Debian 12](https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/debian-12.5.0-amd64-netinst.iso)



If you are running a newer version of Windows, try to install Windows Terminal.  It will be easier to copy / paste commands into it.

[Windows Terminal](https://apps.microsoft.com/detail/9N0DX20HK701?hl=en-US&gl=US)



If you are running Windows, you will need Microsoft's SSH Client.

[Windows SSH Client](https://learn.microsoft.com/en-us/windows-server/administration/openssh/openssh_install_firstuse?tabs=gui)



Optionally, you can get an SSH Client for your phone as well.

[Android Juice SSH Client](https://play.google.com/store/apps/details?id=com.sonelli.juicessh&hl=en_US&gl=US&pli=1)

[iPhone Termius SSH Client](https://apps.apple.com/us/app/termius-terminal-ssh-client/id549039908)



## Debian VM

Create a new VM in VirtualBox.  It is very important to uncheck the unattended install option.  If you don't, you will get many features that will overcomplicate your setup. 

Go to VirtualBox and click 'New' to create a new VM.  On the wizard screen, choose:

- Name: Debian

- Folder: (default is fine)

- ISO Image: Choose your Debian 12 ISO file that you downloaded earlier.

- Type: Linux

- Version: Debian 12 Bookworm (64-bit)

- Skip Unattended Installation

It is very important to uncheck the unattended install option.  If you don't, you will get many features that will overcomplicate your setup.

All other defaults are fine at this point.  You can start your VM.

When Debian boots up, choose the 'Install' option (graphical install is fine but unnecessary).

Going with the defaults of Debian Setup are fine.  The rest of this guide will assume that you used the values below.  If you didn't, just replace your own values throughout the guide.

Throughout setup use these values:

- Hostname: debian

- Domain: WORKGROUP

- Username: vbox

During package management selection:

- Uncheck Debian Desktop

- Uncheck Gnome

- Check SSH Server

After installation, your VM will shut down.  You can start it back up to make sure that it works.  Log in as vbox.



## Debian Commands

A quick note about commands - It is OK to copy and run bash comments which start with #. If you run comments, nothing happens but you actually will see them in your bash history. Bash history can be viewed with the the following command:

```bash
# Use the spacebar to advance page by page. To quit, hit 'q'.
more $HISTFILE
```



## sudo

Sudo (Super User Do) will allow you to run administrative commands from the a user account.

Log in to Debian as vbox.

```bash
# Elevate to the Super User (a.k.a root) account.
su
# Install sudo.
apt-get install sudo
# Add the vbox user to the group that can run sudo.
/sbin/adduser vbox sudo
# Exit the SU session.
exit
# Exit the vbox session.
exit 
```

From this point on, you can run commands starting with 'sudo' which will tell Debian to run the command as root.  Debian will prompt for credentials periodically when sudo is used.

If you try to run a command and you forgot to use sudo, you can do:

```bash
# This will re-issue the last command as sudo.
sudo !!
```

 

## Virtual Box Bridged Networking

By default, VirtualBox has your VM using NAT for networking.  This means that your VM is behind a virtual network and not really on your 'real' network.  You will want bridged networking so that your VM gets its own IP address from your router.

Log in to Debian as vbox.

```bash
# This will show your IP Address
ip address
```

You will see IP addresses for two interfaces.  One is your 'lo' which is your loopback interface.  It's 127.0.0.1.  Nothing to worry about there.

The other interface is your virtual ethernet.  It will probably have an IP that starts with 10.  That is a virtual address that your VM is getting from VirtualBoxe's virtual networking.

Shut down your VM.

```bash
# This will shut down your VM gracefully after about a minute.
sudo shutdown
```

On your host computer, open VirtualBox Manager.

Go to Network-->Adapter 1 Tab

- Set 'Attached to:' to 'Bridged Apapter'.

- Set 'Name:' to the card on your physical machine that is on your network. (Ethernet card if your Host computer is using ethernet, Wifi if the Host is using Wifi)

Start you Debian VM back up and log in as vbox.

```bash
# This will show your IP Address
ip address
```

You should now see an IP that starts with 192 (or whatever range your router uses).  If so, your VM is now a machine connected to your router.  For all intents and purposes, it is seen as a 'real' computer by everything on your network.



## SSH

SSH is the protocol used to remote to a lot of Linux computers.  The reason that you want to use SSH to access your VM via Windows Terminal is that you can copy / paste commands into it instead of typing them into your running VM.

You’ll be on your host machine (I'm assuming Windows) for a bit during this part.  You should also have Windows Terminal and Windows SSH Client installed.

Open Windows Terminal.  You will be at a powershell prompt.

```powershell
# This is the Powershell command to start an SSH session.
ssh vbox@debian
```

You are now remoted into your VM.  In fact, you don’t even need to really visit the Virtualbox Interface too much anymore.  You can also copy / paste commands more easily this way.



## Headless Virtual Machine

It is easier to run your Debian VM headless and to use SSH to interact with it.

Shut down your Debian VM.

```bash
# This will shut down your VM gracefully after about a minute.
sudo shutdown
```

On your host computer, open the VirtualBox Manager.

Highlight your VM in VirtualBox and go to Machine-->Create Shortcut on Desktop.

Right click that new shortcut and go to Properties.  You should see a command line in there that looks like this*:

```powershell
"C:\Program Files\Oracle\VirtualBox\VirtualBoxVM.exe" --comment "Debian12" --startvm "{b9904e9e-0333-4464-98e2-63427c0a445c}"
```

Change the command to look like this*:

```powershell
"C:\Program Files\Oracle\VirtualBox\VBoxManage.exe" startvm "{b9904e9e-0333-4464-98e2-63427c0a445c}" --type headless
```

*The jumbled GUID string {b9904e9e-0333-4464-98e2-63427c0a445c} is unique to your computer. 

You changed the .exe to VBoxManage, you removed the '--comment "Debian12"'.  You removed the two dashes before startvm.  You added '--type headless'.

Save your shortcut.  The nice icon will disappear.  Fix it if you want to by repointing the icon to 'C:\Program Files\Oracle\VirtualBox\VirtualBoxVM.exe'.  Make sure it works by running the shortcut one time.  You can tell it worked by running VirtualBox Manager which will report that your VM is running even though you can't see it.  You should be able to SSH into your VM.

On your host machine, hit WinKey+R to open a the Run dialog.  Type in: “shell:startup” and hit ENTER.  That should open your startup folder.  Copy your newly created Debian shortcut into that folder.  Now when you start your computer, Debian will start up invisibly.  From now on you would get to it via the various SSH Clients that you have access to.



## Neofetch and Micro

From Windows Terminal SSH to your Debian VM

```powershell
ssh vbox@debian
```

You are going to install the neofetch and micro packages.  Neofetch is for cool factor.  Micro is a text editor.  Any time a tutorial tells you to use 'vi' or 'nano', you can use 'micro' instead.  It has more intuitive inner workings like CTRL-S to save, CTRL-Q to quit, and ALT-G to list available commands.

```bash
# This will install neofetch and micro.
sudo apt-get install micro neofetch
```

Set Neofetch to pop on login:

```bash
# This will edit your .batchrc file using the Micro text editor.
micro .bashrc
```

Scroll to the bottom of that file.  On the last line / new line type (case-sensitive):

```bash
neofetch
```

CTRL-S to save, CTRL-Q to quit Micro.

```bash
# Exit the vbox session.
exit 
```

From Windows Terminal, SSH to your Debian VM to log in.

```powershell
ssh vbox@debian
```

You should see a nice presentation of your system’s basic info when you log in.



## Samba

Samba is the protocol used by lots of OSes to share files.  Windows has it by default and just refers to it as 'File Sharing'.

From Windows Terminal, SSH to your Debian VM to log in.

```powershell
ssh vbox@debian
```

```bash
# Install Samba
sudo apt-get install samba
# Set the samba password for the vbox account.
# It is probably a good idea to re-use the login password for vbox here.
sudo smbpasswd -a vbox
```

Now you need to edit your smb.conf file.

```bash
# Open smb.conf in the Micro editor.
sudo micro /etc/samba/smb.conf
```

In the file, find the section called [Homes].

In the [Homes] section, set the following values :

- read only = no

- create mask = 0775

- directory mask = 0775

CTRL-S to save, CTRL-Q to quit Micro.

Samba needs to restart after you edit the configuration.

```bash
# This will restart the smbd (Samba) service.
sudo systemctl restart smbd.service
```

On your Host computer, you should now be able to open Windows Explorer (file explorer) and go to:

\\\debian

It will ask for your vbox user/pass.  

You will see a 'vbox' folder. Right click it-->Map network drive. 

Pick a letter (D for Debian? S for Samba?  Whatever!).  Click Finish.  Now you have a drive in windows mapped to the home folder of your Debian VM.



## Updating Debian

From Windows Terminal, SSH to your Debian VM to log in.

```powershell
ssh vbox@debian
```

If you want to update your whole entire system, the command is:

```bash
sudo apt-get update
```

When you run that, you are updating EVERYTHING on the system.  Unlike Windows, Debian's apt-get package manager keeps track of everything for you via its dependency trees.



If you want to see all installed programs:

```bash
sudo apt list --installed
```

 

## Docker

You want to get the official latest Docker running on your Debian system.  Docker keeps its own repositories and they keep them updated much better than anything that might be in the Debian repositories.

The instructions in this section came from the Docker site: https://docs.docker.com/engine/install/debian/

From Windows Terminal, SSH to your Debian VM to log in.

```powershell
ssh vbox@debian
```

Now you will run the commands to set up the Docker repositories and install Docker.

```bash
# Add Docker's official GPG key.
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources. (this is a large command)
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Update your repos.
sudo apt-get update

# Install Docker
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

 

## Portainer-CE

Think of Portainer as a GUI for Docker. Portainer-CE (Community Edition) is the free version.

From Windows Terminal, SSH to your Debian VM to log in.

```powershell
ssh vbox@debian
```

If you look at the Portainer documentation, Portainer can be set up in a one-line command:

```bash
# Download the Portainer Image and run the Portainer Container. (this is a large command)
sudo docker run -d -p 8000:8000 -p 9443:9443 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:latest
```

You can now see what containers are active in Docker.

```bash
# See the active containers in Docker.  CTRL-C to Quit.
sudo docker stats
```

Now let's go get into Portainer.  On your Host machine, go to this website:

[https://debian:9443](https://debian:9443)

You will see the browser warning page because even though you are using https in the address, there isn't really a certificate backing the site.  Click through the browser warnings to continue.

Notice above the port 9443 where you specified 

```yaml
-p 9443:9443
```

That is telling docker that if it sees traffic on 9443, to forward to 9443 on your machine.  You cannot change the port on the right side because that's the port set in the docker container.  But you can make the port on the left side whatever you want.  So if you wanted to use 10001, you could do that.  Then the address would be:

[https://debian:10001](https://debian:10001)

**Do not use common ports like 80,443, etc. or you will quickly run into conflicts.**

Portainer will make you set up a password.  After you do that, you can click Get Started.  Now you will see the Environments.  There is only one Environment listed there so just click it.  You have now made it to the dashboard of your Docker setup!



## Backing Up Your Debian VM

This is a great time to back up your VM.  There are two ways to do backups and you should regularly do both.

1. If you have an external hard drive, just back up the corresponding subdirectory of VirtualBox VMs:
   
   C:\Users<Your Windows Username>\VirtualBox VMs\Debian12
   
   The most important file in there is the .vdi file (Virtual Disk Image) but back up that whole folder to an external drive if you have one.

2. In Virtualbox, highlight your Debian VM, go to Machine-->Tools-->Snapshots.
   
   Take a new Snapshot called Debian-CleanInstall.  Now you can revert back to this any time you need to undo a huge mistake.

Keep in mind that every time you do a snapshot you are cloning the .vdi file which is already > 2GB.  So Option 1 is the better backup for your drive space and Option 2 is the better backup for a quick UNDO.  Use Option 1 on the regular, use Option 2 before you do things that might break your system.  So right before you do a major update, you can create a snapshot called “Debian-BeforeUpdate”. If the update works out, you can delete that snapshot later.

**It is important to note that when you delete a snapshot, your entire VM can become corrupted if VirtualBox manager is End-Tasked during the lengthy delete process. This is another reason that you should regularly back your Debian VM up using Option 1.**



## MeTube

MeTube is a YouTube downloader.  We'll use it as an example of how to get more containers running in Docker.

From Windows Terminal, SSH to your Debian VM to log in.

```powershell
ssh vbox@debian
```

You need a folder for MeTube to download to.  You can go to your mapped Debian drive on your host machine and create a folder called 'MeTube' (case-sensitive).  You can also run this command:

```bash
# Create a directory for MeTube to download files.
mkdir MeTube
```

You will now build your own stack template.  Go back to the portainer site on your Host machine:

[https://debian:9443](https://debian:9443)

On the left navigation, choose Custom Templates (a sub-menu of App Templates).  Click the button for 'Add Custom Template'

Set the following values:

- Title: metube (<--case sensitive - must be lowercase!)

- Description: MeTube

Scroll down to the web editor and paste this in:

```yaml
version: "3"

services:

  metube:
    image: alexta69/metube:latest
    container_name: metube
    user: 1000:1000
    volumes:
    - /home/vbox/MeTube:/downloads
    ports:
    - 8081:8081
    restart: unless-stopped
```

Click the Create The Template button.  If you get errors, you have to make sure that there are two spaces before every indentation (not tabs).  So 'metube:' has two spaces before it.  'image:' has 4 spaces before it.  etc. etc.

This template is in the YAML (.yml, .yaml) format.  YAML cares a whole lot about spacing.  The Portainer interface will check your YAML for validity and give errors if it sees something wrong.

Now highlight the Template you just made and click 'Deploy the Stack'.

It will run slow because it's downloading the image.  Eventually it will give you a message of success or failure.  If successful, it will take you to the stacks list to show you that it's running.

Keep in mind here that 'Stacks' is an invention of Portainer.  It is not really a Docker concept.  If you want to see the Docker containers running, you can click 'Containers' in Portainer's left menu.

Notice that port you used in the stack template? 8081.  In your Windows browser go to:

[http://debian:8081](http://debian:8081)  (take note you are not using https)

You should see the MeTube interface.  Copy / Paste a link to a youtube video in there and hit Add.

Now remember this part of the config?

```yaml
/home/vbox/MeTube:/downloads
```

The folder on the left is YOUR folder.  The folder on the right is the folder inside the docker container.  Just like with ports, you can choose any folder that you want on the left, the one on the right is not changeable by you.

From Windows Terminal, SSH to your Debian VM to log in.

```powershell
ssh vbox@debian
```

Run this command to list your files/folders.

```bash
# List files and folders in your current working directory.
ls -al
```

You should see the 'MeTube' folder you created earlier.

```bash
# Change current working directory to 'MeTube'.
cd MeTube
# List files and folders in your current working directory.
ls -al
```

You should see the file that you just downloaded from YouTube.

The quick way to do the same thing would have been:

```bash
# List files in a specific directory ( ~ shortcuts to your home folder)
ls -al ~/MeTube
```

Back on your Host machine, open your Debian: drive that you mapped.  You should also see the file there.
