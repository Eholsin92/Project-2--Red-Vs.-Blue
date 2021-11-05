# Project-2--Red-Vs.-Blue




Red V Blue Project Report 

Eric Holsinger

11/1/2021





Overview

For this project, I was tasked to act as an offensive security, Red Team, to exploit a vulnerable Capstone VM. I was then tasked to use Kibana to analyze logs taken during the Red Team attack. As I analyzed, I used the data to develop ideas for new alerts that can improve my monitoring as the Blue Team. Even though I already knew what I did to exploit the target, analyzing the logs is still valuable as it teaches me: what my attack looks like from a defender's perspective, how stealthy or detectable my tactics are, and which kinds of alarms and alerts SOC and IR professionals can set to spot attacks like mine while they occur, rather than after. Finally, I was tasked to communicate your findings in the form of a presentation to my staff. However, in a real engagement, my client will pay me not to break into their network, but to teach them how to protect it. This is why both written and verbal communication skills are vital in the cybersecurity field.

Findings
Day 1
Discover the IP address of the Linux web server.
	The IP addres of the Linux web server is 192.168.1.105. This was found through both NetDiscover and Nmap. 



 
Locate the hidden directory on the web server.
The hidden directory was found by adjusting the URL of the company’s webpage to gain access to the hidden directory. Within the directory was a document giving instructions on how to access the company’s webdav server as well as a password hash. 



Brute force the password for the hidden directory using the hydra command:
hydra -l ashton -P rockyou.txt -s 80 -f -vV 192.168.1.105 http-get /company_folders/secret_folder/

Break the hashed password with the Crack Station website or John the Ripper.
ryan:linux4u


Connect to the server via WebDav.
To connect to the webserver I first opened up a file manager window on the Linux Kali machine. In the network section I entered into the navigation bar: dav://192.168.1.105/webdav/. This connected me to the WebDav server. I then used Ryan’s cracked username and password to gain access to the server. 

Upload a PHP reverse shell payload.
I used msfvenom to create the payload. 

To execute this payload I first opened Metasploit. I used a standard multi/handler exploit so that I could easily set my custom payload and the listener host IP and listener port (4444). Having gained access to the WebDav folder I was easily able to drag and drop the php payload file into the WebDav folder. After the exploit was executed by simply clicking on the php file in the WebDav folder a Meterpreter session was opened gaining access to Ryan’s system. 



Execute payload Find and capture the flag.
After gaining access to the system with Meterpreter I created a shell. After that I simply moved to the root folder (cd /) and listed the contents (ls). I was able to find the flag in that folder, which I then read (cat flag.txt). 



Alternative Method: SSH
During the initial Nmap search I discovered that Port 22 on the target machine was open. After discovering Ryan’s password I decided to test if I was able to get into his system using SSH. To do that I used the command “ssh ryan@192.168.1.105”. This ended up working and I was able to gain access and find the flag through this alternative method as well. 




 
Day 2
Identify the offensive traffic.


Identify the traffic between your machine and the web machine:
When did the interaction occur?

What responses did the victim send back?

What data is concerning from the Blue Team perspective?
This data is concerning because it clearly is suspicious. With a large amount of error codes there is always a chance that is indicative of a brute force attack. The nature of the attack (trying different passwords over and over) will create a large amount of error codes as seen in the image above. The 200 - success codes could mean that is normal traffic working correctly or if it occurs after a brute force attack it could indicate the attack was a success and the attacker got into the system. 
Find the request for the hidden directory.


In your attack, you found a secret folder. Let's look at that interaction between these two machines.
How many requests were made to this directory? At what time and from which IP address(es)?


Which files were requested? What information did they contain?
Doc Files. The doc file in this case contained instructions on how to access the company’s webdav server. 

What kind of alarm would you set to detect this behavior in the future?
In the future it would be highly beneficial to set up an alert for traffic to the secret folder. The alert could be set for a very low number because the baseline of access to the folder normally is very low (as shown in the traffic before and after the attack; different companies would need to provide baselines based on their own traffic). This alert can be set up to email, text, or even shut down access if the threshold is met. 
Identify at least one way to harden the vulnerable machine that would mitigate this attack.
One way would be to limit traffic to the secret folder page, possibly by limiting based on IP. This would allow the person (in this case Ashton) to access the folder when he is at the proper location and block off any traffic coming from an outside IP. Encrypting the data inside the secret folder would also be a great way to protect the data within the folder. Another preventative measure would be to ensure all employees have complex passwords that regularly change. Because Ashton’s password was simple and happened to be contained in the infamous rockyou.txt it was relatively easy to gain access to his account. 
Identify the brute force attack.


After identifying the hidden directory, you used Hydra to brute-force the target server. Answer the following questions:
Can you identify packets specifically from Hydra?

This shows traffic to the secret folder (the same search as the above question). As you can see 99.6% of all the traffic came from Hydra. 
How many requests were made in the brute-force attack?

15,330 hits were made during the brute force attack. This includes the 2 hits that came after hydra finished its search and found the user’s password. 
How many requests had the attacker made before discovering the correct password in this one?

15,328 hits came from hydra which is how many attempts it made before discovering the correct password. 
What kind of alarm would you set to detect this behavior in the future and at what threshold(s)?
One alarm would be to cause an alert to go off with any kind of hydra traffic. The fact that it works so fast would mean another method, such as an IPS, might be needed to shut down that traffic as soon as it is detected. Also setting an alert for a certain number of log-in attempts to get into the secret folder would be beneficial. Since Ashton should be the only person logging into the folder a small number, such as 5 attempts, should be plenty to allow him to mess up his log-in a few times without inundating the SOC staff with alerts while still protecting against potentially malicious activity. 
Identify at least one way to harden the vulnerable machine that would mitigate this attack.
Once again there are several ways you could harden against the attack. One basic but potentially effective method would be to lock out an account for a certain amount of time after a set number of unsuccessful log-ins. This would prevent a brute force attack from happening because it would lock the account after a set number of attempts making future attempts impossible. Enabling 2 factor authentication on the log-in would also be another method to prevent the attack. This would make the hydra attack useless because even if it was able to find the password the 2nd factor would prevent further log-in. Once again making sure employees use complex and regularly changing passwords also can help prevent brute force attacks from working using well known password lists like rockyou.txt. 
Find the WebDav connection.


Use your dashboard to answer the following questions:
How many requests were made to this directory?

As shown there were 63 requests made to the webdav directory and 64 made to the shell.php file within the directory. 
Which file(s) were requested?
A file called passwd.dav was found in the folder. In the case of my traffic I already found the file on the webpage so I did not click on the file in the webdav directory so there was no recorded traffic from the webdav directory to the file. 
What kind of alarm would you set to detect such access in the future?
I would set an alarm that detected and alerted for any outside traffic to the webdav folder. Also depending on the baseline of how much the folder is used within the company an alert could also be set that would alert if the traffic to the folder reached a set threshold. 
Identify at least one way to harden the vulnerable machine that would mitigate this attack.
A mitigation technique for this attack would be to block any outside traffic from accessing the Webdav folder. It seems as if it is used for internal file sharing within the company so blocking traffic not coming from within the company would help harden it. Also enabling 2 factor authentication to log-in to the directory would be beneficial. And once again making sure employees use complex and changing passwords to protect themselves and the company and its assets. 
Identify the reverse shell and meterpreter traffic.


To finish off the attack, you uploaded a PHP reverse shell and started a meterpreter shell session. Answer the following questions:
Can you identify traffic from the meterpreter session?
Yes, there is traffic coming from the Kali source IP over port 4444, the port meterpreter session was established at the time of the attack. 

What kinds of alarms would you set to detect this behavior in the future?
Make sure to set up alerts for any traffic coming from port 4444 as the attack has shown it is a known port used for attacks. Also alert for any php file uploaded onto the system. 
Identify at least one way to harden the vulnerable machine that would mitigate this attack.
These hardening techniques are very similar to the last section. Don't allow uploading of files to the Webdav directory or severely restrict access to the directory. Also making sure to block traffic from port 4444. 



Summarization 

Network Topology
What are the addresses and relationships of the machines involved?

 
For this particular project all of the machines were located in the Azure cloud. Within that was the main host Windows machine. From the Hyber V Manager there was the Linux Kali machine, the Capstone web server target machine, and the ELK machine which logged the data from the Capstone machine into Kibana. 

Red Team
Security Misconfiguration - The log-in on the web server did not have a lock-out limit for log-in attempts which allowed the brute force attack to occur. Port 22 was also open to the public so it was very easy to SSH into the system once the password was cracked.  
WebDav Vulnerability - With the cracked password it was very simple to log-in to the company’s WebDav server, especially when paired with the log-in instructions found in the secret_folder. With access to this WebDav server it was a simple drag and drop to put in any file into the directory without restriction. This allowed for the malicious php file to be uploaded. 
Simple/Bad Passwords - The employees at the company seemed to have very simplistic usernames and passwords. The usernames were the employee’s first name. The passwords were easily cracked with the infamous rockyou.txt file, even the hashed passwords. This weak password policy turned out to be a major vulnerability. 
Blue Team
There was a large amount of evidence found of the attack. The amount of traffic on the system skyrocketed during the attack making it very obvious. The traffic to specific pages (especially the secret_folder page) also gave a great indication of what the attack was targeting. Finally a large amount of traffic from Port 4444 with which the meterpreter session was established also helped clue in on the attack. 
In the future traffic to the secret_folder must be intensely watched. Traffic to and from the WebDav directory also needs to be focused on, especially if any files are uploaded. Finally any large amount of traffic from a single IP needs to be watched very closely. 
Mitigation
Alarms need to be set at a number of places. Attempts to access the secret_folder need to have an alarm connected to it. An alert for excess log-in attempts also needs to be set. Any traffic to the WebDav folder, especially when files are uploaded, needs to have an alarm associated as well. For basic baselines and examples related to this project for these alarms see the Day 2 section above. 
A number of controls should be put in place on the target machine to help prevent attacks like this in the future. A limit on the amount of log-in attempts before the account is locked needs to be put in place right away. A password policy that requires a longer password with unique characters also needs to be enacted. Closing ports, such as Port 22, to outside traffic will help limit the attack space on the target machine. Encrypting data, especially any sensitive data (such as that in the secret_folder) should also be considered. Finally disallowing dragging and dropping any file without restriction in the WebDav server could help prevent similar attacks. This would ensure any potential malicious actor with access to the WebDav directory couldn't easily upload a dangerous script as was done with this project. 



