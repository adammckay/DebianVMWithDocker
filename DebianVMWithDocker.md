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

All other defaults are fine at this point.  **You may find the 20GB drive space limiting so set tht to something you can handle.** You can start your VM.

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

By default, VirtualBox has your VM using NAT for networking. This means that your VM is behind a virtual network and not really on your 'real' network. You will want bridged networking so that your VM gets its own IP address from your router.

Log in to Debian as vbox.

```bash
# This will show your IP Address
ip address
```

You will see IP addresses for two interfaces. One is your 'lo' which is your loopback interface. It's 127.0.0.1. Nothing to worry about there.

The other interface is your virtual ethernet. It will probably have an IP that starts with 10. That is a virtual address that your VM is getting from VirtualBoxe's virtual networking.

Shut down your VM.

```bash
# This will shut down your VM gracefully after about a minute.
sudo shutdown
```

On your host computer, open VirtualBox Manager.

Go to Network►Adapter 1 Tab

- Set 'Attached to:' to 'Bridged Apapter'.

- Set 'Name:' to the card on your physical machine that is on your network. (Ethernet card if your Host computer is using ethernet, Wifi if the Host is using Wifi)

Start you Debian VM back up and log in as vbox.

```bash
# This will show your IP Address
ip address
```

You should now see an IP that starts with 192 (or whatever range your router uses). If so, your VM is now a machine connected to your router. For all intents and purposes, it is seen as a 'real' computer by everything on your network.

If possible, you should configure your router so that your Debian VM has a static entry.  This means that it will always get the same IP address from the router.  Otherwise, you may run into situations where the IP changes unexpectedly.  How to do this is out of the scope of this guide.  Look at your router's config page or search online for "{your router model} static entry".

## SSH

SSH is the protocol used to remote to a lot of Linux computers. The reason that you want to use SSH to access your VM via Windows Terminal is that you can copy / paste commands into it instead of typing them into your running VM.

You’ll be on your host machine (I'm assuming Windows) for a bit during this part.  You should also have Windows Terminal and Windows SSH Client installed.

Open Windows Terminal. You will be at a powershell prompt.

```powershell
# This is the Powershell command to start an SSH session.
ssh vbox@debian
```

You are now remoted into your VM. In fact, you don’t even need to really visit the Virtualbox Interface too much anymore.  You can also copy / paste commands more easily this way.

## Headless Virtual Machine

It is easier to run your Debian VM headless and to use SSH to interact with it.

Shut down your Debian VM.

```bash
# This will shut down your VM gracefully after about a minute.
sudo shutdown
```

On your host computer, open the VirtualBox Manager.

Highlight your VM in VirtualBox and go to Machine►Create Shortcut on Desktop.

Right click that new shortcut and go to Properties. You should see a command line in there that looks like this*:

```
"C:\Program Files\Oracle\VirtualBox\VirtualBoxVM.exe" --comment "Debian12" --startvm "{b9904e9e-0333-4464-98e2-63427c0a445c}"
```

Change the command to look like this*:

```
"C:\Program Files\Oracle\VirtualBox\VBoxManage.exe" startvm "{b9904e9e-0333-4464-98e2-63427c0a445c}" --type headless
```

*The jumbled GUID string {b9904e9e-0333-4464-98e2-63427c0a445c} is unique to your computer. 

You changed the .exe to VBoxManage, you removed the '--comment "Debian12"'. You removed the two dashes before startvm. You added '--type headless'.

Save your shortcut. The nice icon will disappear. Fix it if you want to by repointing the icon to 'C:\Program Files\Oracle\VirtualBox\VirtualBoxVM.exe'. Make sure it works by running the shortcut one time. You can tell it worked by running VirtualBox Manager which will report that your VM is running even though you can't see it. You should be able to SSH into your VM.

On your host machine, hit WinKey+R to open a the Run dialog. Type in: “shell:startup” and hit ENTER. That should open your startup folder. Copy your newly created Debian shortcut into that folder. Now when you start your computer, Debian will start up invisibly. From now on you would get to it via the various SSH Clients that you have access to.

## Neofetch and Micro

From Windows Terminal SSH to your Debian VM

```powershell
ssh vbox@debian
```

You are going to install the neofetch and micro packages.  Neofetch is for cool factor.  Micro is a text editor. Any time a tutorial tells you to use 'vi' or 'nano', you can use 'micro' instead. It has more intuitive inner workings like CTRL-S to save, CTRL-Q to quit, and ALT-G to list available commands.

```bash
# This will install neofetch and micro.
sudo apt-get install micro neofetch
```

Set Neofetch to pop on login:

```bash
# This will edit your .batchrc file using the Micro text editor.
micro .bashrc
```

Scroll to the bottom of that file. On the last line / new line type (case-sensitive):

```ini
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

Samba is the protocol used by lots of OSes to share files. Windows has it by default and just refers to it as 'File Sharing'.

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
```ini
read only = no
create mask = 0775
directory mask = 0775
# This line is optional if you want to hide dot (.) directories and files.
veto files = /.*/
```

CTRL-S to save, CTRL-Q to quit Micro.

Samba needs to restart after you edit the configuration.

```bash
# This will restart the smbd (Samba) service.
sudo systemctl restart smbd.service
```

On your Host computer, you should now be able to open Windows Explorer (file explorer) and go to:
```
\\debian
```

It will ask for your vbox user/pass. 

You will see a 'vbox' folder. Right click it►Map network drive. 

Pick a letter (D for Debian? S for Samba? Whatever!). Click Finish. Now you have a drive in windows mapped to the home folder of your Debian VM.

## Updating Debian

From Windows Terminal, SSH to your Debian VM to log in.

```powershell
ssh vbox@debian
```

If you want to update your whole entire system, the command is:

```bash
sudo apt-get update
```

When you run that, you are updating EVERYTHING on the system. Unlike Windows, Debian's apt-get package manager keeps track of everything for you via its dependency trees.

If you want to see all installed programs:

```bash
sudo apt list --installed
```

## Docker

You want to get the official latest Docker running on your Debian system. Docker keeps its own repositories and they keep them updated much better than anything that might be in the Debian repositories.

The instructions in this section came from the Docker site: [https://docs.docker.com/engine/install/debian](https://docs.docker.com/engine/install/debian)

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

Now let's go get into Portainer. On your Host machine, go to this website:

[https://debian:9443](https://debian:9443)

You will see the browser warning page because even though you are using https in the address, there isn't really a certificate backing the site. Click through the browser warnings to continue.

Notice above the port 9443 where you specified 

```yaml
-p 9443:9443
```

That is telling docker that if it sees traffic on 9443, to forward to 9443 on your machine. You cannot change the port on the right side because that's the port set in the docker container. But you can make the port on the left side whatever you want. So if you wanted to use 10001, you could do that. Then the address would be:

[https://debian:10001](https://debian:10001)

**Do not use common ports like 80,443, etc. or you will quickly run into conflicts.**

Portainer will make you set up a password. After you do that, you can click Get Started. Now you will see the Environments. There is only one Environment listed there so just click it. You have now made it to the dashboard of your Docker setup!

## Backing Up Your Debian VM

This is a great time to back up your VM. There are two ways to do backups and you should regularly do both.

1. If you have an external hard drive, just back up the corresponding subdirectory of VirtualBox VMs:
   
   C:\Users\Your Windows Username\VirtualBox VMs\Debian12
   
   The most important file in there is the .vdi file (Virtual Disk Image) but back up that whole folder to an external drive if you have one.

2. In Virtualbox, highlight your Debian VM, go to Machine►Tools►Snapshots.
   
   Take a new Snapshot called Debian-CleanInstall. Now you can revert back to this any time you need to undo a huge mistake.

Keep in mind that every time you do a snapshot you are cloning the .vdi file which is already > 2GB. So Option 1 is the better backup for your drive space and Option 2 is the better backup for a quick UNDO. Use Option 1 on the regular, use Option 2 before you do things that might break your system. So right before you do a major update, you can create a snapshot called “Debian-BeforeUpdate”. If the update works out, you can delete that snapshot later.

**It is important to note that when you delete a snapshot, your entire VM can become corrupted if VirtualBox manager is End-Tasked during the lengthy delete process. This is another reason that you should regularly back your Debian VM up using Option 1.**

## MeTube

MeTube is a YouTube downloader. We'll use it as an example of how to get more containers running in Docker.

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

- Title: metube (◄case sensitive - must be lowercase!)

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

Click the Create The Template button. If you get errors, you have to make sure that there are two spaces before every indentation (not tabs). So 'metube:' has two spaces before it. 'image:' has 4 spaces before it. etc. etc.

This template is in the YAML (.yml, .yaml) format. YAML cares a whole lot about spacing. The Portainer interface will check your YAML for validity and give errors if it sees something wrong.

Now highlight the Template you just made and click 'Deploy the Stack'.

It will run slow because it's downloading the image. Eventually it will give you a message of success or failure. If successful, it will take you to the stacks list to show you that it's running.

Keep in mind here that 'Stacks' is an invention of Portainer. It is not really a Docker concept. If you want to see the Docker containers running, you can click 'Containers' in Portainer's left menu.

Notice that port you used in the stack template? 8081. In your Windows browser go to:

[http://debian:8081](http://debian:8081) (take note you are not using https)

You should see the MeTube interface. Copy / Paste a link to a youtube video in there and hit Add.

Now remember this part of the config?

```yaml
/home/vbox/MeTube:/downloads
```

The folder on the left is YOUR folder. The folder on the right is the folder inside the docker container. Just like with ports, you can choose any folder that you want on the left, the one on the right is not changeable by you.

From Windows Terminal, SSH to your Debian VM to log in.

```powershell
ssh vbox@debian
```

Run this command to list your files/folders.

```bash
# List files and folders in your current working directory.
ls -la
```

You should see the 'MeTube' folder you created earlier.

```bash
# Change current working directory to 'MeTube'.
cd MeTube
# List files and folders in your current working directory.
ls -la
```

You should see the file that you just downloaded from YouTube.

The quick way to do the same thing would have been:

```bash
# List files in a specific directory ( ~ shortcuts to your home folder)
ls -la ~/MeTube
```

Back on your Host machine, open your Debian: drive that you mapped.  You should also see the file there.

## PiHole

PiHole is a DNS Sinkhole.  It will intercept ads and block them.  Additionally, it is useful for easily creating your own DNS records.  PiHole makes use of DNSMasq for some of its DNS handling. 

**At this point, it is also assumed that your router will always give your Debian VM the same IP address.  If there's any chance that it will not get the same IP, configure static routing in your router admin for your Debian VM.**

From Windows Terminal, SSH to your Debian VM to log in.

```powershell
ssh vbox@debian
```

We're going to create folders for PiHole and DNSMasq.

```bash
# Create a folder for PiHole
mkdir .pihole
# Create a folder for DNSMasq 
mkdir .dnsmasq
```

Notice the dot (.) before the folder name.  That will make the directory hidden.  You can always run the following to show all files in your home folder:

```bash
# Show all files in the current working directory.
ls -la
```

PiHole does not allow you to configure wildcard domains but DNSMasq does.  So now you will create a configuration file for DNSMasq.

```bash
# Create a configuration file for DNSMasq
# Remember to substitue 192.168.x.xx with the actual address of your Debian VM.
echo address=/.debian/192.168.x.xx > .dnsmasq/07-debian.conf
```

On your Host machine, log in to the Portainer site:

[https://debian:9443](https://debian:9443)

On the left navigation, choose Custom Templates (a sub-menu of App Templates). Click the button for 'Add Custom Template'

Set the following values:

- Title: pihole (◄case sensitive - must be lowercase!)

- Description: PiHole

Scroll down to the web editor and paste this in:

```yaml
version: "3"

# More info at https://github.com/pi-hole/docker-pi-hole/ and https://docs.pi-hole.net/
services:
  pihole:
    container_name: pihole
    image: pihole/pihole:latest
    # For DHCP it is recommended to remove these ports and instead add: network_mode: "host"
    ports:
      - "53:53/tcp"
      - "53:53/udp"
      #- "67:67/udp" # Only required if you are using Pi-hole as your DHCP server
      - "8080:80/tcp"
      - "44343:443/tcp"
    environment:
      TZ: 'America/New_York'
      WEBPASSWORD: 'vbox'
    # Volumes store your data between container upgrades
    volumes:
      - '/home/vbox/.pihole:/etc/pihole'
      - '/home/vbox/.dnsmasq:/etc/dnsmasq.d'
    #   https://github.com/pi-hole/docker-pi-hole#note-on-capabilities
    cap_add:
      - NET_ADMIN # Required if you are using Pi-hole as your DHCP server, else not needed
    restart: unless-stopped
```

Notice that we set the WEBPASSWORD environment variable to 'vbox'. Choose any admin password that you want. Also notice that we are using ports 8080 and 44343 here because we don't want to use the regular 80/443 ports.  We will use port 53 however because that is the port for DNS resolution to work correctly.

Click the Create The Template button.  

Let's see if PiHole is working.  On your Host machine, go to:

[http://debian:8080/admin](http://debian/admin)

If that worked, you will be able to log in to PiHole with the password that you set in your stack (YAML) configuration.

If you click the 'Dashboard' menu item, you will see that PiHole reports some basic information about Total queries and Domains on Adlists.  The Total queries will be zero (0).  That is because nothing is using PiHole as a DNS Server yet.

Now you need to set your Host machine's DNS to use PiHole.  Follow these instructions (Windows 10/11 is lower down) to do that.  The IP that you use will be the IP of your Debian VM.  If you want to configure an alternate DNS, you can use CloudFlare's DNS resolver at 1.1.1.1.  This way, if PiHole has an issue, your Host machine will still get DNS from CloudFlare's DNS Server.

[https://www.windowscentral.com/how-change-your-pcs-dns-settings-windows-10](https://www.windowscentral.com/how-change-your-pcs-dns-settings-windows-10)

Now your Host machine is using PiHole as it's DNS Server.  If you start to browse to different websites, you will see that PiHole's dashboard is now showing activity in the count of Total queries.  You can even click PiHole's 'Query Log' menu item and see if the log corresponds to where you are browsing.  This also means that PiHole is actively blocking things for you.

Because we set the wildcard domain to 'debian' via the DNSMasq configuration file, any subdomain of 'debian' will now point to your PiHole installation on port 80. So that means that (for right now) these will all work:

[http://pihole.debian:8080/admin](http://pihole.debian:8080/admin)

[http://whatever.debian:8080/admin](http://whatever.debian:8080/admin)

[http://yougettheideanow.debian:8080/admin](http://yougettheideanow.debian:8080/admin)

This will come in handy when you set up a Proxy Manager.

There is one more thing to do.  We must point the Debian VM to use its own PiHole container for DNS.  Right now, your Host machine is the only thing using PiHole - it will be a lot easier when your Debian VM uses it as well.

There are two ways to do this.  **You should only choose one.**

1. Set your Host machine's DNS back to Automatic.  Then go to your Router's configuration and set the DNS Server on the router to be 192.168.x.xx where 192.168.x.xx is the IP Address of your Debian VM.  To play it safe, use a second entry of 1.1.1.1 for Cloudflare's public DNS.  This option will cover **every** device on your home network and is the recommended.

2. You can use Debian's resolvconf package to set your DNS on your Debian VM.
   
   ```bash
   # This will install resolvconf which will allow Debian to manage its resolvconf file which is written at bootup.
   sudo apt-get install resolvconf
   # This will open your interfaces file in the Micro editor.
   sudo micro /etc/network/interfaces
   ```
   
   In the interfaces file, add this line to your primary interface. IP Addresses are separated by spaces.
   
   ```ini
   dns-nameservers 127.0.0.1 1.1.1.1
   ```
   
   If you go with option 2, you will need to manually set DNS for all devices on your network or you will need to simply point them to the IP / Port # of each container.

## NginxProxyManager

Nginx Proxy Manager will act as a reverse proxy and allow you to configure subdomains on the fly.  This part of the guide assumes you have set up PiHole correctly and that you have a DNSMasq wildcard domain called 'debian'.

From Windows Terminal, SSH to your Debian VM to log in.

```powershell
ssh vbox@debian
```

We're going to create folders for Nginx Proxy Manager.

```bash
# Ceate a folder for Nginx Proxy Manager
mkdir .nginx
```

On your Host machine, log in to the Portainer site:

[https://debian:9443](https://debian:9443)

On the left navigation, choose Custom Templates (a sub-menu of App Templates). Click the button for 'Add Custom Template'

Set the following values:

- Title: nginx(◄case sensitive - must be lowercase!)

- Description: Nginx Proxy Manager

Scroll down to the web editor and paste this in:

```yaml
version: "3"

services:
  nginx:
    container_name: nginx
    image: jlesage/nginx-proxy-manager:latest
    environment:
      - TZ=America/New_York
    ports:
      - 10001:8181
      - 80:8080
      - 443:4443
    volumes:
      - /home/vbox/.nginx:/config:rw
    network_mode: bridge
    restart: unless-stopped
    #default credentials: admin@example.com / changeme
```

On your host machine, you should now be able to access the Nginx Proxy Manager Admin Panel at this address:

[http://debian:10001](http://debian:10001)

Log in with the default credentials:

- User: admin@example.com

- Pass: changeme

Nginx Proxy Manager will prompt you to enter new Admin credentials.

Click the link to 'Dashboard' in the menu.

Click 'Proxy Hosts'

You will come to this page often.  This is where you will point Nginx Proxy Manager to your various Docker containers.  You will now set up the containers that you have created so far.

Click the 'Add Proxy Host' button.

Enter the following:

- Domain Names: nginx.debian (There is a quirk in NPM where you MUST mouse click the Domain Name after you type it in.)

- Scheme: http (This is because nginx uses http by default.)

- Forward Hostname / IP: 192.168.x.xx (This should be the IP Address of your Docker VM.)

- Forward Port: 10001

- Block Common Exploits: Checked (This is usually a good idea.)

- WebSockets Support: Checked (This is usually a good idea.)

Test this out on your Host machine by visiting:

[http://nginx.debian](http://nginx.debian)

Click the 'Add Proxy Host' button.

Enter the following:

- Domain Names: portainer.debian (There is a quirk in NPM where you MUST mouse click the Domain Name after you type it in.)

- Scheme: https (This is because portainer uses https by default.)

- Forward Hostname / IP: 192.168.x.xx (This should be the IP Address of your Docker VM.)

- Forward Port: 9443

- Block Common Exploits: Checked (This is usually a good idea.)

- WebSockets Support: Checked (This is usually a good idea.)

Test this out on your Host machine by visiting:

[http://portainer.debian](http://portainer.debian) (Notice you are not getting certificate errors any more on http.)

Click the 'Add Proxy Host' button.

Enter the following:

- Domain Names: metube.debian (There is a quirk in NPM where you MUST mouse click the Domain Name after you type it in.)

- Scheme: http (This is because metube uses http by default.)

- Forward Hostname / IP: 192.168.x.xx (This should be the IP Address of your Docker VM.)

- Forward Port: 8081

- Block Common Exploits: Checked (This is usually a good idea.)

- WebSockets Support: Checked (This is usually a good idea.)

Test this out on your Host machine by visiting:

[http://metube.debian](http://metube.debian)

Click the 'Add Proxy Host' button.

Enter the following:

- Domain Names: pihole.debian (There is a quirk in NPM where you MUST mouse click the Domain Name after you type it in.)

- Scheme: http (This is because pihole uses http by default.)

- Forward Hostname / IP: 192.168.x.xx (This should be the IP Address of your Docker VM.)

- Forward Port: 8080

- Block Common Exploits: Checked (This is usually a good idea.)

- WebSockets Support: Checked (This is usually a good idea.)

Test this out on your Host machine by visiting:

[http://pihole.debian/admin](http://pihole.debian/admin) (Note PiHole uses a default page at /admin.)

You can also assign multiple domain pointers to the same site.  For example, if you find yourself going to PiHole a lot, you can always additionally point 'p.debian' to PiHole or 'n.debain' to Nginx Proxy Manager.  The possibilities are endless.

One additional note.  Nginx Proxy Manager is serving all of your sites over http.   It is not within the scope of this guide to get an SSL certificate to force your websites to use https.  If you are interested in doing that, you will (probably) need to buy a domain name and get an SSL certificate from a real CA (Certificate Authority).

At this point, you should be fairly well versed in creating a Docker Container and pointing Nginx Proxy Manager to it.

## Heimdall

Heimdall is a nice dashboard for all of your services.  At this point you have four different Containers running in Docker on various ports.  You are going to want to organize all of this somehow or you will probably lose track of all of the port/url combinations you are creating.

From Windows Terminal, SSH to your Debian VM to log in.

```powershell
ssh vbox@debian
```

We're going to create folders for Heimdall.

```bash
# Create a folder for Heimdall
mkdir .heimdall
```

On your Host machine, log in to the Portainer site:

[http://portainer.debian](http://portainer.debian) 

Remember, it used to be [https://debian:9443](https://debian:9443) but you have Nginx Proxy Manager making life easy now.

On the left navigation, choose Custom Templates (a sub-menu of App Templates). Click the button for 'Add Custom Template'

Set the following values:

- Title: heimdall(◄case sensitive - must be lowercase!)

- Description: Heimdall

Scroll down to the web editor and paste this in:

```yaml
version: "3"

services:
  heimdall:
    image: lscr.io/linuxserver/heimdall:latest
    container_name: heimdall
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/New_York
    volumes:
      - /home/vbox/.heimdall:/config
    ports:
      - 10800:80
      - 10443:443
    restart: unless-stopped
```

Notice the ports used - you are just making ports up at this point.  There is no rhyme or reason to them as long as they don't conflict with ports already in use.  As a general rule of thumb stay between 10K and 65K.

On your host machine, you should now be able to access the Nginx Proxy Manager Admin Panel at this address:

[http://nginx.debian](http://nginx.debian)

Click the 'Add Proxy Host' button.

Enter the following:

- Domain Names: heimdall.debian (There is a quirk in NPM where you MUST mouse click the Domain Name after you type it in.)

- Scheme: http (This is because heimdall uses http by default.)

- Forward Hostname / IP: 192.168.x.xx (This should be the IP Address of your Docker VM.)

- Forward Port: 10800

- Block Common Exploits: Checked (This is usually a good idea.)

- WebSockets Support: Checked (This is usually a good idea.)

Go straight to Heimdall on your Host machine.

[http://heimdall.debian](http://heimdall.debian)

Heimdall's interface will be empty.  There will be a link to add an application or you can click the button for 'Application List' to add Applications.

Here are the Applications you should add:

Portainer

- Application Name: Portainer

- Application Type: Portainer

- Colour: #161b1f

- URL: [http://portainer.debian](http://portainer.debian)

- Config: Enable

- User/Pass: (Supply your Portainer Login / Pass)

- Skip TLS: Enable

Nginx Proxy Manager

- Application Name: Nginx Proxy Manager

- Application Type: Nginx Proxy Manager

- Colour: #161b1f

- URL: [http://nginx.debian](http://nginx.debian)

- Config: Enable

- User/Pass: (Supply your Nginx Proxy Manager User / Pass)

MeTube

- Application Name: MeTube

- Application Type: youtube-dl

- Colour: #161b1f

- URL: [http://metube.debian](http://metube.debian)

PiHole

- Application Name: Pi-Hole

- Application Type: Pi-hole

- Colour: #161b1f

- URL: [http://pihole.debian](http://pihole.debian)

- Config: Enable

- API Key: (Go to [http://pihole.debian](http://pihole.debian), Click Settings►API Tab►Click 'Show API Token'►Click 'Yes, show API token'►Copy the long string that is the 'Raw API Token'►Come back to Heimdall and paste that token into the 'API Key' field.

## Basic Docker Workflow

From this point forward, you should be pretty proficient at navigating the various pieces of the Debian VM Docker setup.  The rest of the guide will be less detailed about adding things because you've done it several times now.

The basic workflow of adding new things is:

1. Identify a Docker Image you want to run.

2. Obtain a working Docker Compose (YAML).

3. Don't forget to create a dot (.) folder in /home/vbox/ to store the container data.

4. Create a Stack in Portainer using the Custom Templates feature.

5. Take note of the port mappings for the Container.

6. Use Nginx Proxy Manager to create a Proxy Host to your new Container.

7. Use Heimdall to add your new Container to the Dashboard.

## Glances

Glances will show you all of the activity of your Debian VM.

Here is the YAML:
```yaml
version: "3"

services:
  glances:
    container_name: glances
    image: nicolargo/glances:latest
    pid: host
    environment:
      - GLANCES_OPT=-w
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
    ports:
      - 61208-61209:61208-61209
    restart: unless-stopped
    #port 61208 is the web port
```

## Jellyfin
Jellyfin is an excellent Media server.  There are readily available Jellyfin Client apps for most SmartTV platforms.

You will need to create 3 folders in your /home folder: '.jellyfin' (for configuration), 'Movies', and 'Television'.

```yaml
version: "3"

volumes:
  vol_jellyfin:

services:
  jellyfin:
    container_name: jellyfin
    image: linuxserver/jellyfin:latest
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/New_York
    volumes:
      - /home/vbox/.jellfyin:/config
      - /home/vbox/Movies:/data/movies
      - /home/vbox/Television:/data/tvshows
    ports:
      - 8096:8096
    restart: unless-stopped
```

## Dynamic DNS

(Forthcoming...)

## Wireguard

(Forthcoming...)


## Docker Resources

There are tons of great sites out there to get more in-depth information or to go find Docker Images.  This guide was largely based on many of these resources.

In no particular order, they are:

[LinuxServer.io](https://fleet.linuxserver.io) - They have produced a fleet of top-notch Docker Images.

[Docker Hub](https://hub.docker.com) - Probably the largest collection of Images.

[r/SelfHosted](https://www.reddit.com/r/selfhosted) - A great Sub-Reddit with tons of Self Hosting information.

[Awesome Self Hosted](https://github.com/awesome-selfhosted/awesome-selfhosted) - A massive list of Self Hosted applications.

