# Super Earth
### For Managed Democracy
"*The existence of high casualty missions implies the existence of low casualty missions, and we can take some solace in that*"

### General Information
Welcome to the **first CTF room** created by me :D<br>
I hope you will enjoy it as much as I enjoyed making it!

Link to the TryHackMe room: 

Attack machine IP: ????????????<br>
Target machine IP: ????????????

>As always, terminal output is cut down for brevity.

---
### Scanning the field
Let's begin. In order to find the objective, Helldivers have to scan the field and locate crucial assets:

```java
root@ip-10-113-94-131:~# nmap -sS 10.113.170.99
Host is up (0.000093s latency).
PORT   STATE SERVICE
21/tcp open  ftp
22/tcp open  ssh
```

Let's check the specifics. Add these specific ports to your scan and use script and version scan

```java
root@ip-10-113-94-131:~# nmap -sC -sV 10.113.170.99 -p21,22 -n -Pn

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.5
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| drwxr-xr-x    2 0        0            4096 Aug 25 15:09 fun
| -rw-r--r--    1 0        0          904912 Aug 25 15:09 helldivers.jpg
|_-rw-r--r--    1 0        0             274 Aug 25 15:09 malevelon_creek_orders.txt

22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.19 (Ubuntu Linux; protocol 2.0)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
root@ip-10-113-94-131:~# 
```
*Objective Located!*<br>
Clear as day, the Anonymous FTP login is **allowed** and there are files inside. That's our first clue.

For the newcomers, here is the explanation of the nmap flags:
- `-sC`: use default scripts; this way FTP login was automatically performed, with output shown in our terminal
- `-sV`: check service version; now we know that FTP server is on version 3.0.5 and OpenSSH on 9.6p1. These versions don't have known vulnerabilities that grant access to the system straight away (at least for now. Who knows what AI's gonna bring us?)
- `-p21,22`: scan only ports 21 and 22; script and version scans can take much more time if we don't provide specific ports
- `-n`: do not lookup reverse-DNS domain; totally optional, but a good practice
- `-Pn`: assume the server is up, don't send ICMP Echo; also a good practice if you know the server is up. It's more useful with Windows servers, but doesn't hurt to use here

---

### Main Objective: FTP
We already know it's open to anonymous connections. Connect to the FTP server and verify if it's really true.

pho1.png

Download all files to your Attack Machine. Use the `get` command. Changing directories and listing files works just the same as with Linux.
```
ftp> ls
drwxr-xr-x    2 0        0            4096 Aug 25 15:09 fun
-rw-r--r--    1 0        0          904912 Aug 25 15:09 helldivers.jpg
-rw-r--r--    1 0        0             274 Aug 25 15:09 malevelon_creek_orders.txt

ftp> cd fun
ftp> ls 
-rw-r--r--    1 0        0              58 Aug 25 15:09 play_me.txt

ftp> get play_me.txt
226 Transfer complete.

ftp> cd ..
250 Directory successfully changed.

ftp> get helldivers.jpg
226 Transfer complete.

ftp> get malevelon_creek_orders.txt
226 Transfer complete.

ftp> 
```
We have identified and retrieved three files:
- `play_me.txt` from the `fun` folder
- `helldivers.jpg`, a photo of the Helldivers team
- `malevelon_creek_orders.txt`, probably a clue to our task

*I'm a Helldiver, and I want to have fun!*
```
root@ip-10-113-94-131:~# cat play_me.txt 
more fun during our mission:
https://youtu.be/6VPn9jYm1QU
```
> Oh, so it's a fun addition to our missions. If you want to, you can blast it on your headphones :)

Helldivers.jpg? Should I put it in a frame and stare at it all day?!

pho2.png

> Seems fishy.. Why would there be a photo of our fellow comrades?

What do we have next..? Right! The orders!
```
root@ip-10-113-94-131:~# cat malevelon_creek_orders.txt 

         ===== Transmission from Super Earth Command =====

      This is mission control. 
      Helldiver, were you able to extract the sample 
                                   from the given resource?

    ============================================================
```
**Extract** the sample? **Given** resource? The only thing we have is the photo..

Maybe if we *walk* through that *bin*ary photo, the answer might come to us?

```
root@ip-10-113-94-131:~# binwalk helldivers.jpg 

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             JPEG image data, JFIF standard 1.01
904553        0xDCD69         Zip archive data, at least v2.0 to extract, compressed size: 171, uncompressed size: 238, name: uncommon_sample.txt
904890        0xDCEBA         End of Zip archive, footer length: 22
```
> Zip archive data, huh? Notice the text file name in Zip Description

All we have to do is to **extract** the resource. Just use the command:
```
binwalk -e helldivers.jpg --run-as=root
```
..and a folder with the sample has been created: `_helldivers.jpg.extracted`

pho3.png

The partial Helldivers communication has been received!<br>
Ah, so the Helldiver `john` was a participant on `Malevelon Creek`. Also, the order was to **SCAN** the **WHOLE** field. Maybe that's an another clue?

`Uncommon Sample Collected: THM{N0_P4In_NO_FR33D0M}`

---

### Scan the WHOLE field

```
root@ip-10-113-94-131:~# nmap -sS 10.113.170.99 -p- -n -Pn
Host is up (0.00011s latency).
PORT      STATE SERVICE
21/tcp    open  ftp
22/tcp    open  ssh
17800/tcp open  unknown
```
what is it
... smb doesn't work I have to reload the machine


### Sources
- Nmap
- FTP on Linux
- Binwalk
- SMB Client

