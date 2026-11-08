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
    - *ping -c2 10.66.134.126* 
      **successful**
  -Start a ports scan to identify probable vulnerabilities and export results into a grapeable format easy to read and reutilize
    - sudo nmap -p- --open -sS --min-rate 5000 -n -Pn -vvv 10.66.134.126 -oG allportsBilling 
    - We discover that 4 ports are open the http/80, ssh/22, 3306/mysql and 5038/Aterisk Manager Interface
  -Start an aggresive scan to  ports to discover if information macthes and what servers are using
    - sudo nmap -p22,80,3306,5038 -sCV 10.66.134.126 -oN targetBilling
    - ssh is 8.2p1 on ubuntu, if I have true users I can try a brute force
    - http is on Apache on Ubuntu, but it don't has a title or version
    - mysql
    - Asterisk manager
  - I'm going to access to http in browser
    - There is a login page
  - Lets numerate web directories of our page url
    - gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://10.66.134.126/ -t 30
    - Found a mbilling dir, lets focus it
    - 
      
3.2 Privilege Escalation 
- Starting permissions: root
- Misconfiguration or weakness: server system
- How higher privileges were confirmed: whoami
- Security impact: Critic
- Actions performed:
  - Search for the 3rd flag navigating on directories
 
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
- Location or discovery method
  -Second flag done
3.5 Flag 3
- Location or discovery method: servers directory
- Technical reasoning: When we don´t see the responses, we can try to obtain all system control by using a privilege scalation when we are already inside the system

