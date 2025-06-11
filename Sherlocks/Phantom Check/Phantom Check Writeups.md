# Scenario
This is a very easy sherlock that demostrates some of the common virtualization detection techniques used by attackers. Is suspected that the threat actor carried out **anti-virtualization checks to avoid detection in sandboxed enviroments**, so we have to **analyze the event logs** and **identify the specific techniques** used for virtualization detection. It also require evidence of the registry checks or processes the attacker executed to perform these checks.

## Skills Learnt
We'll gain the ability to create detection rules by identifying specific WMI queries, comparing processes for virtual machine detection, and analyzing registry keys or file paths associated with virtual environments.

## Initial Analysis
We have provided two `.evtx` files to examine the incident logs. In these files, we observe only Windows PowerShell event logs, indicating that PowerShell was used.

# Questions
---
1. **Which WMI class did the attacker use to retrieve model and manufacturer information for virtualization detection?**
   
 As mentioned above, PowerShell was used, so we can assume that the attacker used the `Get-WmiObject` command to retrieve information, as it is one of the most common ways to do so through WMI classes in PowerShell.

Therefore, we can use the **Find** tool to search for this in the event log. As shown in the images below, the `Win32_ComputerSystem` class is used to obtain model and manufacturer details.

![image](https://github.com/user-attachments/assets/712e97ed-abe1-4fcd-abde-3ce994aea90e)

![image](https://github.com/user-attachments/assets/edbf252b-83d4-4caf-85dc-cd1d04336373)

**ANSWER `Win32_ComputerSystem`**

2. **Which WMI query did the attacker execute to retrieve the current temperature value
of the machine?**

Following the same approach, we searched for `Get-WmiObject` and found the query `SELECT * FROM MSAcpi_ThermalZoneTemperature`, which is used to retrieve the current temperature value, as shown in the image below.

![image](https://github.com/user-attachments/assets/dc4e7550-320f-4a32-9f0a-d44e2f8d4485)

**ANSWER: `SELECT * FROM MSAcpi_ThermalZoneTemperature`**

3. **The attacker loaded a PowerShell script to detect virtualization. What is the function name of the script?**

