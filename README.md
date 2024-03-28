# Agent Tesla - Malware Analysis Report

Deda.Cloud Cyber Security Team • 28 Mar 2024


*Cybersecurity BlueTeam, Deda.Cloud*

# **Introduction**
Agent Tesla is a .Net-based Remote Access Trojan (RAT) and data stealer for gaining initial access that is often used for Malware-As-A-Service (MaaS). In this criminal business model, threat actors known as initial access brokers (IAB) outsource their specialized skills for exploiting corporate networks to affiliate criminal groups. As first-stage malware, Agent Tesla provides remote access to a compromised system that is then used to download more sophisticated second-stage tools, including ransomware. 


🌐 [AGICOM: AgentTesla intensifica la sua presenza in Italia: il ruolo cruciale degli allegati PDF](https://cert-agid.gov.it/news/agenttesla-intensifica-la-sua-presenza-in-italia-il-ruolo-cruciale-degli-allegati-pdf/)


# **Key points**

In this document, we will take a deep dive into steps the malware takes to execute the payload through these layers, as well as the various techniques it employs to hinder and slow down the analysis process.

Some of these significant features include:

- Use of AES256 encryption in javascript using "RijindaelManaged" .NET encryption module
- Use of PNG images to hide base64 encoded .NET PEs
- Use of **fileless loader** in the infection chain helps the IoC's threat to be undiscovered.
- Acquisition and exfiltration of credentials from web browsers, applications and extensive system enumeration related to hardware capabilities and current user activities
- **Process Hollowing** has been implemented in the final stages of the loader, wherein the code of RegAsm.exe is replaced with the payload associated with Tesla.

# **1st stage**
The initial javascript payload is started from a document. In this specific case, an excel document created a wscript process executing the following javascript code:
![images/ev_1.png](images/ev_1.png)

In this case, an URL is istantiated in the variable "restribar" to retrieve more javascript code, to be executed in memory using the "eval" function. The downloaded script will perform the decryption of an hardcoded encrypted value, which would be a Powershell script.

# **2nd stage**

The payload is encoded in base64 and obfuscated thanks to RijindaelManaged, a .NET cryptography module. All the configuration for the cryptor, along with the key, is stored in the code.
![images/ev_2.png](images/ev_2.png)
![images/ev_3.png](images/ev_3.png)
After the decryption process, the Powershell script is executed through Wscript.exe


# **3rd stage**

The purpose of the Powershell script is to download an image, find the placeholders <BASE64_START> and <BASE64_END>, and decode the base64 value embedded in the images. This would be a valid .NET DLL, which would be loaded in-memory thanks to Powershell Reflection
![images/ev_4.png](images/ev_4.png)
The method loaded and executed from the DLL receives four arguments, which will be used to perform operations later.
![images/ev_5.png](images/ev_5.png)

# **4th stage**
The second argument is a numeric value used as a flag, in this case to decide wether to install a persistence. Then, data is downloaded from the valid url obtained by reversing the first argument. This data is a reversed base64, and if decoded, it can be confirmed that it is a valid .NET PE file. The routine Tools.Ande performs a process hollowing operation, by creating the suspended process *C:\\Windows\\Microsoft.NET\\Framework\\v4.0.30319\\RegAsm.exe*
![images/ev_6.png](images/ev_6.png) 
The start routine receives the third and fourth arguments, a path and a filename. All the javascript files in the run-directory of the executable are copied in the *HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Run* (T1547.001)
![images/ev_7.png](images/ev_7.png)
The process hollowing is performed in a classic way
![images/ev_8.png](images/ev_8.png)


# **5th stage**
 
## Keylogging
![images/ev_13.png](images/ev_13.png)
## Microsoft vault password stealing
![images/ev_14.png](images/ev_14.png)
## Application credential stealing
Full list of applications here
![images/ev_15.png](images/ev_15.png)
## System enumeration
![images/ev_10.png](images/ev_10.png)
![images/ev_11.png](images/ev_11.png)
![images/ev_12.png](images/ev_12.png)


# **Configuration**
The malware has the capability of exfiltration via SMTP. Moreover, the configuration hints at a persistence mechanism as well.
![images/ev_9.png](images/ev_9.png)

# **IOCs**

- stage 1
*3007400ade8d88cb583d9eeabc1cd9d4d6f5d814e595aad226d46f04b3b7ffe3*

- stage 2 
*7390f1a9170a68b7155c60e2cf7c7d1879b60dc5e07d03d1a340af7c12d8efbf*

- stage 3 image
*53e8b36becd5610d9a7a8d9a78b713224ce04f3740b3461564dbd5c364c5dc35*

- stage 3 powershell
*5d186be07869bf5ddd08bf9f5b32b163235b7904aa9f26f1a7da9c5f4befff10*

- stage 4 loader
*39fb28ee2f1596faede940c5774b4b216f311718d51f334c4c11a63eb4c078337*

- stage 5 agent
*dc4e0712ab9dca9a6c457a83cfe0d30128d9b85c2aa8d7ed0d977da335dc3b80*

# **Domains**
- stage 2 url: https://paste[.]ee/d/fq6Ny
- image url: https://uploaddeimagens[.]com.br/images/004/731/991/original/new_image.jpg?1707144482
- agent url: (reversed base64encode) http://93.123.39.145/14.txt
	

