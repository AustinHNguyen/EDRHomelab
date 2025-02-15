# EDRHomelab
Simulating both attack and endpoint detection and response on blue team homelab.

<h3>Project:</h3>
This project will be completed using our blue team homelab that was built as a testing and learning environment for these exact types of projects. 
The goal is to simulate an attack on a Windows 10 machine by using Sliver and defending it by using LimaCharlie which is an EDR solution. 


This project follows a guide by Eric Capuano: https://blog.ecapuano.com/p/so-you-want-to-be-a-soc-analyst-part
and a video by Gerald Auger: https://www.youtube.com/watch?v=oOzihldLz7U

<h3>Setup:</h3>
Since this project uses our homelab which was already built, I will only go over steps after installation of VMs.
On the Windows 10 machine, turn off Windows defender permanently by changing the group policy under Computer Config->Admin Templates->Windows Components->Microsoft Defender Antivirus and enable "Turn off Microsoft Defender Antivirus. A registry command must also be entered into cmd. (REG ADD "hklm\software\policies\microsoft\windows defender" /v DisableAntiSpyware /t REG_DWORD /d 1 /f)

Boot into safe mode and set the value of start services to 4. They are all under Computer\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\(Folder) (Replace Folder with Sense, WdBoot, WinDefend, WdNisDrv, WdNisSvc, WdFilter)

Leave safemode and install sysmon. Download SwiftOnSecurity's Sysmon config. Then run Sysmon.

Install LimaCharlie EDR on the Windows machine and create an organization:
![image](https://github.com/user-attachments/assets/29aff637-2469-4535-82b1-7c63c7cbdde3)

Create an artifact collection rule that collects Windows sysmon logs:
![image](https://github.com/user-attachments/assets/c2f1faca-f83f-4dec-94eb-cda51a7088f6)

We are done with the Windows machine for now. We can start on the attack machine. The guide uses Ubuntu, but since we are using our homelab, we already have a Kali machine set up.

<h3>Attack:</h3>
Download Sliver and launch it.

![image](https://github.com/user-attachments/assets/5c0a501f-6f76-4f4b-9df3-98ab0685beb9)

Now we can generate our first payload using this command:
![image](https://github.com/user-attachments/assets/c4100cf5-6158-4619-936d-2ba5c117ff7d)

After that is finished, we use python to create a temporary web server so that we can download the C2 payload to the Windows VM
Command: python3 -m http.server 80

Go to the WindowsVM and download the malware:
![image](https://github.com/user-attachments/assets/786c518a-cf5b-4548-89cc-f77a378be1dc)

It is a good idea to snapshot the Windows VM before executing the malware.

Go back to the attack VM and start both sliver and sliver http listener.

![image](https://github.com/user-attachments/assets/4f0b4f25-e48d-44a2-8f7d-78786bdf093e)

On the Windows VM, we can now execute the C2 payload and we should be able to see the session on the Sliver server.

![image](https://github.com/user-attachments/assets/168d4d5f-cd6a-4dc9-b856-12ee48a5a94d)

Now that we are in, we can use commands such as whoami and getprivs:
![image](https://github.com/user-attachments/assets/ffdd9bd6-2f83-4759-bd16-c431bec378d4)

By using ps -T, Sliver highlights any detected defensive tools in red and our own implant in green:

![image](https://github.com/user-attachments/assets/9e1334d3-200b-43ec-8d60-fa9642247915)

We can now check our sensors on the LimaCharlie web UI. The first tab to look at is the processes tab. Knowing which processes are normal will help figure out which processes are bad, but one way to find out is to check which processes are unsigned. By scrolling through the list, we can see that the implant is not signed, and it has an active network connection:

![image](https://github.com/user-attachments/assets/ac5aca1e-8adc-4e4d-b787-0f6c8798f875)

Let's try to attack the machine by stealing credentials. We can use procdump -n lsass.exe -s lsass.dmp to dump the lsass.exe process from memory.
This works when we have the SeDebugPriviledge. When we look at the timeline for LimaCharlie when searching for SENSITIVE_PROCESS_ACCESS we can look for suspicious sources:
![image](https://github.com/user-attachments/assets/68ac0c93-5afb-4704-bc71-ede0bd8b07bf)

Now that we know an attack has occured and we have detected it, we can make a detections and response rule to alert us and respond to it.
