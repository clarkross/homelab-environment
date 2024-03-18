# Homelab Environment

In order to generate my own projects and analysis, I thought it best to construct my own environment. My interests gravitate much more to blue team operations, but we will need to utilize red team tools to create traffic / information / etc.

*My Workstation Specs:*
* CPU: Intel i7-6700K (8) @ 4.200GHz
* Memory: 32 GB
* VM Storage: 2 TB spinning HD

Firstly, let's setup a standard Linux environment for general operations. Although having a completely sandboxed local environment is more inline with the project scope- a dedicated VM for other operations is useful. I fully intend to make use of all the VMs I'm about to create for personal research or labs.

# Creating General Operations VM

> As a personal choice, I have decided to go with Debian. It will provide us with a clean slate and we can install tools as needed. Many will choose Kali- as someone more interested in blue team operations, I find it easier to start with a clean slate.

### Installing

* https://www.debian.org/distrib/ - Contains download link.
	* Using the DVD installation method. https://cdimage.debian.org/debian-cd/current/amd64/bt-dvd/
	* Torrent option- as it's much faster.

<img src="https://i.postimg.cc/65DXmg5t/image.png">

* VirtualBox is the next step. I currently run Fedora 39 on my daily workstation and will add it to my repo list.

```bash
sudo dnf upgrade --refresh
sudo dnf install @development-tools
sudo wget http://download.virtualbox.org/virtualbox/rpm/fedora/virtualbox.repo -P /etc/yum.repos.d/
sudo dnf install VirtualBox-7.0
```

VirtualBox has been successfully installed, and now I will shift to creating my Debian installation. Below are the options / specifications I chose for the VM:

* Base Memory: 6034 MB
* Processors: 2
* Virtual Storage: 30 GB 

### Customizing The Debian Install

Once the VM has been installed, I install xfce4; did not realize the unattended install would opt for Gnome. Since we're running this through virtualization, it handles much better and I'd rather run XFCE (this is my personal primary desktop environment anyways). in terminal emulator:

```bash
su # Switching to root.
usermod -aG sudo clark # Adding myself as a super user.
su clark
# I uncheck DVD firmware in gnome-software. Debian won't download from unsigned repos by default, but I'd rather keep those security features.
sudo apt update # First time upgrade
sudo apt upgrade # Software didn't need upgrading (great!)
sudo tasksel # Select xfce & unselect gnome.
```

Now a logout, environment switch, and login. I will continue to maintain this VM as my passive blue team research/study OS.

### Other Virtual Machine Installs

* Windows 11 -- Had a Vbox machine copy on my backup drive.
* Parrot Security (Attack Box) -- Local Network
	* Similar to Kali, but is its own spin- wanted to give it a try instead- as I'm more used to Kali from previous experience. Ultimately, this doesn't matter- as tools are just as accessible on either distro.
	* https://www.parrotsec.org/
* Metasploitable3 (Vulnerable Machine) -- Local Network
	* Metasploitable3 is a vulnerable testing machine. We can isolate a network for Parrot OS and Metasploitable3- and analyze some attacks.
	* https://github.com/rapid7/metasploitable3
	* Vulnerabilities: https://github.com/rapid7/metasploitable3/wiki/Vulnerabilities

# Creating Isolated Internal Network

Within VirtualBox, we can make some adjustments to the Network tab via the Parrot Security VM settings. I select internal network and name this lablocal- making the same changes to Metasploitable3 settings.

<img src="https://i.postimg.cc/sgrcV1pj/image.png">

### VB DCHP Server Configuration

Although the VMs have been created and the network settings adjusted accordingly, the machines still will not talk to each other. In order to do so, we need to implement  a DHCP server to assign their IPs in the environment. VirtualBox has an option for this and is quite convenient. 

```bash
cd /user/share/virtualbox # This may not be needed, but generally is the instruction on Windows- just a different path on the Linux FS.
vboxmanage dhcpserver add --network=lablocal --server-ip=10.38.1.1 --lower-ip=10.38.1.110 --upper-ip=10.38.1.120 --netmask=255.255.255.0 --enable
```

# The Result

* Booted up both Metasploitable3 & Parrot OS to ensure they're on the same network.
* Parrot OS shows having the configured IP of ```10.38.1.111``` and since I assigned the IP range of ```10.38.1.110 - 10.38.1.120``` & booted Metasploitable3 first, it gained ```10.38.1.110``` from DHCP.
* A simple nmap scan of the first 1,000 ports reveals some of the vulnerabilities and enumerates the correct service information.

<img src="https://i.postimg.cc/LXywrL70/image.png">
