**================ VEEAM-Backup-Recovery-jobs ================**

This template use the VEEAM Backup & Replication PowerShell Cmdlets to discover and manage VEEAM Backup jobs, Veeam BackupSync, Veeam Tape Job, Veeam Endpoint Backup Jobs, All Repositories and Veeam Services.

Work with Veeam backup & replication V7 to V9.5<br />
Work with Zabbix 3.X<br />
French & English translation for the Template

Explanation of how it works :<br />
The "Result Export Xml Veeam Xml" element sends a powershell command to the host to create an xml of the result of the Get-VBRBackupSession and Get-VBRJob commands.<br />
Then, each request imports the xml to retrieve the information.<br />
Why? Because the execution of this command can take between 30 seconds and more than 3 minutes (depending on the history and the number of tasks) and I end up with several scripts running for a certain time and the execution is in timeout.
The result of the Job is send by Zabbix Sender.<br /><br />

**-------- Items --------**

  - Number of running jobs<br />
  - Result Export Xml Veeam<br />

**-------- Discovery --------**

**1. Veeam Jobs :** 
  - Execution status for each jobs
  - Type for each jobs
  - Number of virtual machine in each jobs
  - Size included in each jobs
  - Size excluded in each jobs
  - Result of each jobs (ZabbixTrapper)
  - Result task ZabbixSender of each jobs
  - Next run time of each jobs

**2. Veeam Tape Jobs :**
  - Execution status for each jobs
  - Result of each jobs (ZabbixTrapper)
  - Result task ZabbixSender of each jobs
  - Next run time of each jobs

**3. Veeam Repository :**<br />
  - Remaining space in repository for each repo<br />
  - Total space in repository for each repo<br />
<br />

**4. Veeam Jobs Endpoint Backup:**<br />
  - Execution status for each jobs<br />
  - Result of each jobs (ZabbixTrapper)<br />
  - Result task ZabbixSender of each jobs<br />
  - Next run time of each jobs<br /><br />


**-------- Triggers --------**<br />
[WARNING] => Export XML Veeam Error <br />

-------- Discovery Veeam Jobs --------<br />
[HIGH] => Job has FAILED <br />
[AVERAGE] => Job has completed with warning  
[HIGH] => Job is still running (8 hours)<br />
[WARNING] => Backup Veeam data recovery problem

-------- Discovery Veeam Tape Jobs --------<br />
[HIGH] => Job has FAILED <br />
[AVERAGE] => Job has completed with warning<br />
[HIGH] => Job is still running (8 hours)<br />
[INFORMATION] => No data recovery for 24 hours<br />

-------- Discovery Veeam Repository --------<br />
[HIGH] => Less than 2Gb remaining on the repository


-------- Discovery Veeam Services --------<br />
[AVERAGE] => Veeam Service is down for each services<br />
<br />

-------- Discovery Veeam Jobs Endpoint Agent --------<br />
[HIGH] => Job has FAILED <br />
[AVERAGE] => Job has completed with warning<br />
[HIGH] => Job is still running (8 hours)<br />
[INFORMATION] => No data recovery for 24 hours<br />
<br>

**-------- Setup --------**

1. Install the Zabbix agent on your host
2. Copy zabbix_vbr_job.ps1 in the directory : "C:\Program Files\Zabbix Agent\scripts\" (create folder if not exist)
3. Add the following line to your Zabbix agent configuration file.<br />
EnableRemoteCommands=1 <br />
UnsafeUserParameters=1 <br />
ServerActive="IP or DNS Zabbix Server"<br />
Alias=service.discovery.veeam:service.discovery<br />
UserParameter=vbr[*],powershell -NoProfile -ExecutionPolicy Bypass -File "C:\Program Files\Zabbix Agent\scripts\zabbix_vbr_job.ps1" "$1" "$2"
4. In Zabbix : Administration, General, Regular Expression : Add a new regular expression :<br /> 
Name : "Veeam"    ;     Expression type : "**TRUE**"     ;     	Expression : "Veeam.\*"<br />
And modify regular expression "Windows service startup states for discovery" : Add : <br />
Name : "Veeam" ; Expression type : "**FALSE**" ; Expression : "Veeam.\*"<br />
5. Import TemplateVEEAM-BACKUPtrapper.xml file into Zabbix. 
6. Associate "Template VEEAM-BACKUP trapper" to the host.
