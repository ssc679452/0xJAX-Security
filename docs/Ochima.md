# Ochima Writeup
**Name:** Ochima  
**OS:** Linux    
**Difficulty:**  Intermediate  
**Complexity:**  Intermediate


## Phase 1: Reconnaissance  
  
To begin, we spawn in the machine and wait for it to assign an IP address. In this case, it appears we will be attacking `192.168.171.32` (Note: I came back to this machine, so later in the writeup you will also see the victim host as `192.168.114.32`)  
  
![Offsec GUI](assets/Ochima/enum-1.png)  
  
## Phase 2: Scanning  
  
Initiating our scanning stage, I run my enumeration script. We are able to find that there is an application running on port `8338`  
  
![Image of our nmap scan](assets/Ochima/enum-2.png) 

Opening my web browser, I navigate to this page and am met with a `Maltrail` logo in the upper left. I try a few basic credentials (admin:admin, admin:password, etc. . .) but none of these works. Following suite, I look up default credentials for Maltrail:  
  
![Maltrail login page](assets/Ochima/enum-3.png)  
  
![Googling default creds for maltrail](assets/Ochima/enum-4.png)  
  
Attempting to sign in with the default credentials (admin:changeme!) we are met with success and are able to get a glance at the version at the bottom of the page:  
  
![Maltrail version after login](assets/Ochima/enum-5.png)  
  
## Phase 3: Vulnerability verification & Exploitation
  
Using searchsploit, I check to see if it has any known vulnerabilities - to which I am not met with success.  
  
![Searchsploit output](assets/Ochima/enum-6.png)  
  
Turning our search over to Google - I do a quick search for `maltrail 0.52` and see a particularly interesting Github repository created by `joshchalabi`.  
  
![Image of our nmap scan](assets/Ochima/enum-7.png)  
  
![Image of our nmap scan](assets/Ochima/enum-8.png)  
  
I use git clone to download this and navigate to its root. Preparing it to run, I change its permission to allow it to be executed and open it for review. Reviewing the code, we can see that it is expecting input in the form `./exploit.sh target_url listening_host listening_port` where listening host and listening port is our nc listener on our attacker machine.  
  
![Googling mailtrail version](assets/Ochima/enum-9.png)  
  
![Image of our nmap scan](assets/Ochima/enum-10.png)  
  
Testing exectution before targeting the victim host.  
  
![Image of our nmap scan](assets/Ochima/enum-11.png)  
  
We plug in our victim machine IP, our attacker IP, and listening port - while opening our nc listener in a secondary tab.  
  
![Image of our nmap scan](assets/Ochima/enum-12.png)  
  
It seems we have a foothold on the victim machine!  
![Image of our nmap scan](assets/Ochima/enum-13.png)  

I opt to spawn a bash shell via Python3 (after determining the location of the Python3 binary).  
  
![Image of our nmap scan](assets/Ochima/enum-14.png)  
  
Before moving on, I make sure to adequately document our user flag as proof alongside the IP address.  
  
![Image of our nmap scan](assets/Ochima/enum-15.png)  
  
## Phase 4: Maintaining access & privilege escalation  
  
Next, I aim to get a more stable shell on the machine in case my current shell dies. To do this, I verified from our initial NMAP scan that SSH service was open and running. I navigate to the `snort` user's home directory and create the directory `.ssh`. I then generate a public / private keypair with `ssh-keygen` and echo the content of my public key into an `authorized_keys` file.  
  
![Image of our nmap scan](assets/Ochima/enum-16.png)  

I utilize the private key I had generated to SSH into the victim machine without issue. Additionally, I call the bash shell as I prefer it.  
![Image of our nmap scan](assets/Ochima/enum-17.png)  
  
![Image of our nmap scan](assets/Ochima/enum-18.png)  

Following this, I pull all of my linux enum tools into my `tool` directory, host a web server via python and pull them onto the victim machine using `wget`, since it is installed. I additionally make all of these tools executable within the folder I created in the `/tmp` directory.  
  
![Image of our nmap scan](assets/Ochima/enum-20.png)  
  
![Image of our nmap scan](assets/Ochima/enum-19.png)  

Beginning our enumeration for priv esc, I start with some low hanging fruit checks such as SUID's, seeing if we have any sudo -l perms, and PwnKit - but none of these yield any interesting findings. I thought that we might be able to do something with the `fusermount3` binary - but this was a dead end it seemed.  
  
[Image of our nmap scan](assets/Ochima/enum-21.png)  
  
![Image of our nmap scan](assets/Ochima/enum-22.png)  
  
![Image of our nmap scan](assets/Ochima/enum-23.png)  
  
![Image of our nmap scan](assets/Ochima/enum-24.png)  

After running `lse.sh -l 2` I see that there is a cron job being run as root calling a script located at `/var/backups/etc_Backup.sh`.

![Image of our nmap scan](assets/Ochima/enum-25.png)  
  
I check and see if we are able to actually write to this file. Jackpot! We can. A root shell is within our grasp. I navigate to this directory and see if there is anything else interesting in it and then proceed to make a backup of the file - so we have something to revert back to if we mess the original up.  
  
![Image of our nmap scan](assets/Ochima/enum-26.png)  
  
![Image of our nmap scan](assets/Ochima/enum-27.png)  

At first, I attempted a named pipe reverse shell - with no success. Looking back on this, I believe that my lack of success here was due to using port 443.  
  
![Image of our nmap scan](assets/Ochima/enum-28.png)  

Utilizing `frizb's MSF-Venom-Cheatsheet`, I opted to use the BASH shell reverve shell, then uploaded to the victim machine.  
  
![Image of our nmap scan](assets/Ochima/enum-29.png)  
  
![Image of our nmap scan](assets/Ochima/enum-30.png)  
  
![Image of our nmap scan](assets/Ochima/enum-31.png)  
  
I set it so that the file is able to be executed and added a line to the `etc_Backup.sh` calling our reverse shell and `sudo rlwrap nc -lvnp 80` open in another tab . . .  
  
![Image of our nmap scan](assets/Ochima/enum-32.png)  
  
![Image of our nmap scan](assets/Ochima/enum-33.png)  
  
We're in as root!  
  
![Image of our nmap scan](assets/Ochima/enum-34.png)  

Let's make it a bit more stable by again adding our public SSH key to the `authorized_keys` file in `root` home and sign in with SSH.  
  
![Image of our nmap scan](assets/Ochima/enum-35.png)  

![Image of our nmap scan](assets/Ochima/enum-35.5.png)  
  
I call BASH again and nab the proof flag along with adequate proof that we have owned this machine.  
  
![Image of our nmap scan](assets/Ochima/enum-36.png)  
  
## Phase 5: Reporting & Documentation  
  
Looking back on this host, there are several addressable misconfigurations that allowed us to fully compromise the victim machine. Initially, the use of default credentials on the Maltrail application allowed us to log in, identify the version, and determine it was vulnerable to remote code execution. After gaining a foothold, we were able to further enumnerate and discover a script was running in the context of root that we were able to write to, allowing us to inject our own commands that additionally executed in the context of root. The system admin could remediate these issues in a few different ways. Updating default credentials and updating the application to a version that is not vulnerable would be a great start. Additionally, implementing a form of AV and locking down permissions on files running in the context of root would help in preventing our privilege escalation vector.  
    



