## KOTH Writeup
# Cal Poly Swfit KOTH - Write-Up
---

## 1. Overview on the competition

This KOTH competition was a purple team competition set up by the amazing Cal Poly Swift where compeitors would log remote into a kali machine and attack 6 machines, gain root, plant their flag, and defend it from other hackers trying to plan theirs.

Due to us not having direct access to the envrionment we had to use ssh tunnelling to access things like the websites, set up blood hound, and more.



<img width="680" height="801" alt="image" src="https://github.com/user-attachments/assets/5127f953-ef93-4e30-b15e-df02a568e3a3" />

This was the competition environment, each machine had a scored service which was either WinRM (port 5895/5896) or SSH (port 22). This came up to 6 total scored services. 

 At the start, i went ahead and tried to gain access to a linux machine at first but after the first hour i went onto Windows so ill go over my attack path for windows first.

---

##  Windows

To start off with windows i did an nmap scan to see the domain name and what ports were open. 

   
    [nmap -sC -sV -T5 [Ip Address]

<img width="1227" height="938" alt="Screenshot 2025-12-01 000733" src="https://github.com/user-attachments/assets/f31fec3b-c64e-416d-8f33-67847c29d8ed" />


So in this screenshot, i was able to see that

3.  **Observation:** [Describe what you found.]

