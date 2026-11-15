TryHackMe — Easy Peasy

<details> 
<summary>Challenge Information</summary>Field| Details
Platform| TryHackMe
Room| Easy Peasy
Difficulty| Easy
Operating System| Linux
Status| Completed
Date Completed| November 4, 2026

</details>1. Introduction

Easy Peasy is a Linux-based CTF challenge focused on web enumeration, hidden information discovery, initial access, and privilege escalation.

The main objectives involved discovering hidden directories, identifying credentials, obtaining remote access, and exploiting a misconfigured scheduled task to escalate privileges.

This writeup documents my methodology, tools, findings, and technical learning throughout the challenge.

2. Reconnaissance

Objective

Identify open ports, running services, and their versions.

Tool

- Nmap

Commands

sudo nmap -sV -p- --open <TARGET_IP>

Additional service enumeration:

sudo nmap -sV -sC -p80,6498,65524 <TARGET_IP>

Findings

Three open ports were identified:

Port| Service| Version
80| HTTP (Nginx)| 1.16.1
6498| SSH (OpenSSH)| 7.6p1
65524| HTTP (Apache)| 2.4.43

Two web servers were running on different ports.

Both web services exposed a "robots.txt" file, making them interesting starting points for further enumeration.

3. Enumeration

3.1 Web Directory Enumeration

Tool: Gobuster

Objective: Discover hidden directories and web resources.

Commands:

gobuster dir -u http://<TARGET_IP> \
-w /usr/share/wordlists/dirb/common.txt

gobuster dir -u http://<TARGET_IP>:65524 \
-w /usr/share/wordlists/dirb/common.txt

A hidden directory named "/hidden" was discovered on port 80.

Further enumeration revealed another directory:

"/hidden/whatever"

Its HTML source contained encoded information.

Using CyberChef, I identified and decoded the encoded content, revealing the first flag.

3.2 HTML Source Inspection

I also inspected the HTML source of the Apache web server.

To filter relevant information, I used:

curl http://<TARGET_IP>:65524 | grep -i flag

This revealed another flag and an additional encoded clue.

Further investigation of the source code led to a hidden directory:

"/n0th1ng3ls3m4tt3r"

The directory name was obtained by identifying and decoding the encoding format suggested by the challenge.

3.3 robots.txt Investigation

Commands:

curl -s http://<TARGET_IP>/robots.txt

curl -s http://<TARGET_IP>:65524/robots.txt

The second "robots.txt" contained an unusual User-Agent entry and an access-related hint.

This was particularly interesting because it suggested that a specific User-Agent value was permitted to access the web resource.

By investigating the associated hash, I recovered the second flag.

3.4 Hidden Directory Investigation

I examined the discovered directory:

curl -s http://<TARGET_IP>:65524/n0th1ng3ls3m4tt3r

Its HTML source contained a password hash and a link to an image containing hidden information.

Password Hash Analysis

Tool: John the Ripper

The challenge provided a custom wordlist.

I identified the hash format and used John the Ripper to perform password cracking.

john --format=gost --wordlist=<WORDLIST> <HASH_FILE>

The recovered password was used as part of the subsequent investigation.

Steganography Analysis

Tool: Steghide

I downloaded the image and examined it for embedded information.

steghide --extract -sf binary.jpg

The extracted text contained binary-encoded information.

Using CyberChef, I decoded the binary content and discovered another username and its associated password.

This provided the credentials needed for the next stage.

4. Initial Access

Objective

Obtain remote access to the target machine.

Tool: SSH

The discovered credentials allowed successful authentication through the SSH service running on port 6498.

Command:

ssh <USERNAME>@<TARGET_IP> -p 6498

After establishing the connection, I investigated the user's home directory and located "user.txt".

The flag was encoded, so I used Ciphey to identify and decode its format.

This completed the initial access objective.

5. Privilege Escalation

Objective

Identify a vulnerability that could allow escalation from a standard user to root.

Vulnerability Identified

A scheduled task configured in "/etc/crontab" executed a script with root privileges.

The script was located at:

/var/www/.mysecretcronjob.sh

I inspected its permissions:

ls -lah /var/www/.mysecretcronjob.sh

The file was writable by the standard user.

This represented a significant security misconfiguration: a privileged scheduled task was executing a script that could be modified by an unprivileged account.

Exploitation

In the controlled CTF environment, I demonstrated the impact of this misconfiguration by modifying the scheduled script and establishing a reverse shell connection.

After receiving the connection, I obtained root-level access and located the final flag in the root user's directory.

Result: Successful privilege escalation from a standard user to root.

6. Tools Used

Tool| Purpose
Nmap| Network scanning and service enumeration
Gobuster| Hidden directory discovery
cURL| HTTP requests and web investigation
CyberChef| Encoding and decoding
John the Ripper| Password hash cracking
Steghide| Steganography
Ciphey| Automated cipher identification and decoding
SSH| Remote access
Bash| Shell interaction and privilege escalation
Netcat| Reverse shell listener

7. Key Lessons Learned

This challenge reinforced several important concepts:

- The importance of comprehensive port scanning.
- How hidden directories can expose sensitive information.
- Why source-code inspection is valuable during web enumeration.
- How encoding and hashing can appear in CTF investigations.
- How steganography can conceal information inside image files.
- The importance of checking file permissions and scheduled tasks.
- How insecure cron job configurations can lead to privilege escalation.

8. Security Recommendations

From a defensive perspective, this challenge demonstrates the importance of:

- Restricting write permissions on scripts executed by privileged accounts.
- Auditing scheduled tasks and cron configurations.
- Protecting sensitive credentials and password hashes.
- Reviewing exposed web resources and directory permissions.
- Monitoring unusual authentication and network activity.

9. Conclusion

Easy Peasy provided practical experience across multiple stages of a penetration testing assessment, from reconnaissance to privilege escalation.

The most significant finding was the insecure cron job configuration, which demonstrated how a seemingly minor permission issue could result in complete system compromise.

This challenge strengthened my understanding of Linux security, web enumeration, password analysis, and privilege escalation techniques.

Learning. Practicing. Improving.

---

This writeup documents work performed in an authorized educational environment on TryHackMe. Sensitive challenge answers and credentials have intentionally been omitted.
