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

Install LimaCharlie EDR on the Windows machine.
