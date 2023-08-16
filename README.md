# Red-Team-Simulation-1
A Red Team engagement that exposed Child and Parent Domain Controllers

### **Initial Access SCOPE of Engagement :**

## **[172.16.25.0/24](http://172.16.25.0/24) [ONLY 172.16.25.1 is out of scope]**

### Objective:

### Objective:

1. To pivot through a network by compromising a public facing web machine and tunnelling your traffic to access other machines in the network.
2. To reach the highest (**root/administrator**) level command execution.
3. To compromise the Child and Parent Domain

### Executing my .ovpn exam environment

```bash
openvpn CCRTA-Exam-TCP4-4443-exam_operator-config.ovpn
```
## My Attacker IP Address: 172.16.250.4
![Screenshot 2023-07-08 at 9 48 01 AM](https://github.com/JFPineda79/Red-Team-Simulation-1/assets/96193551/c362d61d-db0d-44ce-a491-354a3e131b86)

# Enumeration

Enumerating the given IP Ranges

```bash
nmap -sn 172.16.25.0/24 > ./Findings/nmap_172-16-25-0_24.txt
```

And resulted me to 3 IP Addresses

![Screenshot 2023-07-08 at 10 02 27 AM](https://github.com/JFPineda79/Red-Team-Simulation-1/assets/96193551/f4b1128b-60cf-4940-a21a-36e06540e8d6)

### Network Details

| External IP Address | Remarks | Description |
| --- | --- | --- |
| 172.16.25.1 | Out of Scope |  |
| 172.16.25.2 | 22 open ports | Production-Server |
| 172.16.25.3 | 4 open ports (with no port 80), but with RDP port open | child.redteam.corp/Employee-System |

Enumerating 172.16.25.2 using nmap scan, and had 22 open ports

```bash
nmap -A -sV -sT 172.16.25.3 > ./Findings/nmap_172-16-25-3.txt
```
## 172.16.25.2
![Screenshot 2023-07-08 at 10 08 45 AM](https://github.com/JFPineda79/Red-Team-Simulation-1/assets/96193551/0abb9949-5de9-45a0-a1a8-28b95dab850c)
![Screenshot 2023-07-08 at 10 09 23 AM](https://github.com/JFPineda79/Red-Team-Simulation-1/assets/96193551/9471722f-34b3-4bc3-bb7a-ecde9ccb7189)
![Screenshot 2023-07-08 at 10 09 51 AM](https://github.com/JFPineda79/Red-Team-Simulation-1/assets/96193551/d5229108-4ded-454b-8b1b-112133ff6e44)

## 172.16.25.3

```bash
nmap -A -sV -sT 172.16.25.3 > ./Findings/nmap_172-16-25-3.txt
```

We found 4 open ports and validates that this is windows machine where port 3389 
![Screenshot 2023-07-08 at 10 04 54 AM](https://github.com/JFPineda79/Red-Team-Simulation-1/assets/96193551/7d0f1ee4-c0da-4bac-bfe2-92f8661fc69d)

I will get back to this later, while I proceed on IP 172.16.25.2

## port 80 at 172.16.25.2

since port 80 is open, I look at http://172.16.25.2, a Registration page for Red Team Lab

![Screenshot 2023-06-28 at 9 32 04 AM](https://github.com/JFPineda79/Red-Team-Simulation-1/assets/96193551/671daaca-c974-4258-a7ae-257fbd4f1307)

I tried the registration but it give us an error 

![Screenshot 2023-06-28 at 10 13 38 AM](https://github.com/JFPineda79/Red-Team-Simulation-1/assets/96193551/344afbb0-1575-43d3-ae7b-6ee68400c194)

![Screenshot 2023-07-08 at 10 13 21 AM](https://github.com/JFPineda79/Red-Team-Simulation-1/assets/96193551/91e3e2ad-abd0-440f-9218-5d61302bcc68)

error after the registration

## vsftpd 2.3.4

Since I couldn’t get any information on port 80, I moved to the service running on port 21 which I believed vsftpd 2.3.4 has vulnerability.
![Screenshot 2023-07-08 at 10 14 41 AM](https://github.com/JFPineda79/Red-Team-Simulation-1/assets/96193551/725b9bde-e9bc-4b4f-b502-c931fad37585)

| Vulnerability | System | CVSS Version 3.x | CVSS version 2.0 |
| --- | --- | --- | --- |
| CVE-2011-2523 vsftpd 2.3.4 | 172.16.25.2 | 9.8 Critical | 10.0 High |

Using metasploit, We look on possible use of the  vsftpd 2.3.4 service vulnerability.

![Screenshot 2023-07-08 at 12 23 21 PM](https://github.com/JFPineda79/Red-Team-Simulation-1/assets/96193551/dd0ebfc7-2a0f-45c7-bb2b-87bf7d892530)

I found an exploit for vsftpd 2.3.4 which is a Backdoor Command Execution and can be used to the target machine. Selecting the module - exploit/unix/ftp/vsftpd_234_backdoor, and the setting up the following:

RHOSTS: 172.16.25.2

verbose: True

![Screenshot 2023-07-08 at 12 32 16 PM](https://github.com/JFPineda79/Red-Team-Simulation-1/assets/96193551/85d66a19-cd8e-4bdf-878c-2ca84995d926)
![Screenshot 2023-07-08 at 12 34 13 PM](https://github.com/JFPineda79/Red-Team-Simulation-1/assets/96193551/12289f49-0fef-4bfb-b606-8c930d4849ae)

Executing the exploit and a shell session was created, but this is not a interactive shell
![Screenshot 2023-07-08 at 12 35 08 PM](https://github.com/JFPineda79/Red-Team-Simulation-1/assets/96193551/e90ebfff-f2db-4298-af96-4ab0ca80cef6)

To have a interactive shell, I execute a terminal (tty) spawned via Python

```bash
python -c "import pty;pty.spawn('bin/bash')"
```

I got a root shell under the host-name Production-Server

## Production-Server
![Screenshot 2023-06-28 at 10 26 54 AM](https://github.com/JFPineda79/Red-Team-Simulation-1/assets/96193551/97d6cd96-e1cf-40ea-bbd1-7f6b50188946)

we got a root shell of Production-Server

From here I checked the /etc/passwd to check some interesting credentials

```bash
cat /etc/passwd
```
![Screenshot 2023-07-08 at 12 40 06 PM](https://github.com/JFPineda79/Red-Team-Simulation-1/assets/96193551/20e0ba04-b946-4db0-b5d4-e728f6129ff4)


I found a a familiar credential

```bash
msfadmin:x:1000:1000:msfadmin,,,:/home/msfadmin:/bin/bash
```

I look around to gather more interesting information. I found another user named “prod-admin”.

![Screenshot 2023-07-08 at 12 46 22 PM](https://github.com/JFPineda79/Red-Team-Simulation-1/assets/96193551/70ab05d3-f736-4627-89f7-e0faa4aa9423)


## prod-admin

I navigate to home root directory and found 5 users folders. And look to the prod-admin folder and found a file named “credential.txt”

![Screenshot 2023-07-08 at 12 48 49 PM](https://github.com/JFPineda79/Red-Team-Simulation-1/assets/96193551/6ee39fc7-9280-4af2-add2-2487c8519ab9)

```bash
cd prod-admin
ls
cat credential.txt
```
![Screenshot 2023-07-08 at 12 49 43 PM](https://github.com/JFPineda79/Red-Team-Simulation-1/assets/96193551/90889d99-bbe2-4146-ab62-676c30f779da)

2 interesting credentials

| User Name | Password |
| --- | --- |
| support | support@123 |
| prod-admin | Pr0d!@#$% |

First, I try to login using the 1st credential - support:support@123. It doesn’t work

```bash
ssh support@172.16.25.2
```

![Screenshot 2023-07-08 at 12 56 23 PM](https://github.com/JFPineda79/Red-Team-Simulation-1/assets/96193551/ef1d2b75-0691-4725-968a-80de57760c7c)

Next, I go with trying the 2nd credential - prod-admin:Pr0d!@#$%. It does work

```bash
ssh prod-admin@172.16.25.2
```
![Screenshot 2023-07-08 at 12 59 25 PM](https://github.com/JFPineda79/Red-Team-Simulation-1/assets/96193551/a9a7faf2-a6df-4f50-b16e-dece94b8ce59)

So far this is the summary of what I got from the root directory enumeration of the Production-Server.

| User’s Directory | Remarks |  |
| --- | --- | --- |
| ftp | nothing interesting |  |
| msfadmin | nothing interesting |  |
| prod-admin | found credential.txt | Support User Credential = support:support@123 and Prod-admin Credential = prod-admin:Pr0d!@#$% |
| service | nothing interesting |  |
| user | nothing interesting |  |

# Pivoting

## IP 10.10.10.5 : Production-Server

Moving on, I conduct an initial enumeration inside the compromised Production-Server

Run a network card enumeration and found its internal ip address. Do a ping test on it and it is active.

![Screenshot 2023-07-08 at 1 03 52 PM](https://github.com/JFPineda79/Red-Team-Simulation-1/assets/96193551/891ac41e-7f77-4887-aac0-243500fb2672)

Surprisingly nmap is working on the production server, I scanned the network to look for an ip range

```bash
nmap -sN 10.10.10.0/24
```
![Screenshot 2023-07-08 at 1 37 10 PM](https://github.com/JFPineda79/Red-Team-Simulation-1/assets/96193551/203038b8-7161-4a07-8640-1ee752993205)

![Screenshot 2023-07-08 at 1 37 33 PM](https://github.com/JFPineda79/Red-Team-Simulation-1/assets/96193551/7dfe5c22-bbcb-4615-b574-81ad19409c40)

Found an IP range 10.10.10.1, 10.10.10.2, 10.10.10.3 and 10.10.10.4

### Network Details

| External IP Address | Description |
| --- | --- |
| 172.16.25.1 | Out of Scope |
| 172.16.25.2 | Production-Server |
| 172.16.25.3 | child.redteam.corp/Employee-System |
| Internal IP Address | Description |
| 10.10.10.1 | Reserved IP of the network |
| 10.10.10.2 | we suspect this as the Domain Controller |
| 10.10.10.3 | unknown |
| 10.10.10.4 | unknown |
| 10.10.10.5 | The compromised Production-Server (Ubuntu 8.04) |

From our gathered IP ranges we moved on our first target which is 10.10.10.3

## IP 10.10.10.3

With the compromised Production-Server, I setup my proxychains at 1080 to be able to run commands directly from my machine without touching the Production-Server.

![Screenshot 2023-07-09 at 8 32 11 AM](https://github.com/JFPineda79/Red-Team-Simulation-1/assets/96193551/2c72a5df-cf4c-4a84-be4b-74b13ba76e62)

I run an nmap scan to 10.10.10.3 to find an open ports to attack with.

```bash
proxychains nmap -sV 10.10.10.3
```

![Screenshot 2023-07-09 at 7 51 11 AM](https://github.com/JFPineda79/Red-Team-Simulation-1/assets/96193551/3df08f03-cf8c-4f93-9825-773fcaf738b5)

Again I run some nmap scan to it

```bash
proxychains nmap -sC -A 10.10.10.3
```
![Screenshot 2023-07-09 at 8 00 12 AM](https://github.com/JFPineda79/Red-Team-Simulation-1/assets/96193551/7598a7ae-6ae0-4bfb-a90c-a42df934ee03)

Found 4 open ports with 2 high ports in it. Based on the protocol assigned to this 2 open ports, looks like these are web applications.

| Port | Description | Exploit |
| --- | --- | --- |
| 9090 | http / web application / Cockpit web service 162 - 188 | found some article related to its exploit |
| 10000 | http / web application / MiniServ 1.953 (Webmin httpd) | Unable to find any exploit on this version |

Before accessing these I setup a new proxy in firefox foxyproxy for port 1080

![Screenshot 2023-07-09 at 7 54 27 AM](https://github.com/JFPineda79/Red-Team-Simulation-1/assets/96193551/44e0e34f-1896-4f46-9cde-fafd99685e5d)

I will check what’s on these ports by navigating through the following urls:

```bash
http://10.10.10.3:9090
http://10.10.10.4.10000
```
![Screenshot 2023-07-09 at 7 58 49 AM](https://github.com/JFPineda79/Red-Team-Simulation-1/assets/96193551/ae327810-17cb-4730-9d9d-290bdadbf33b)

I found out that this url https://10.10.10.3:9090 is a server named Admin-System.
