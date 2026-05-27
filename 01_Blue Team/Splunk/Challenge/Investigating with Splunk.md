SOC Analyst **Johny** has observed some anomalous behaviours in the logs of a few windows machines. It looks like the adversary has access to some of these machines and successfully created some backdoor. His manager has asked him to pull those logs from suspected hosts and ingest them into Splunk for quick investigation. Our task as SOC Analyst is to examine the logs and identify the anomalies.

To learn more about Splunk and how to investigate the logs, look at the rooms [splunk101](https://tryhackme.com/room/splunk101) and [splunk201](https://tryhackme.com/room/splunk201).

Room Machine

Before moving forward, deploy the machine. When you deploy the machine, it will be assigned an IP **Machine IP**: `10.128.132.220`. You can visit this IP from the VPN or the Attackbox. The machine will take up to 3-5 minutes to start. All the required logs are ingested in the index **main.**

### Questions

#### How many events were collected and Ingested in the index **main**?  

```splunk
index=main
```

**Answer** : 12256

#### On one of the infected hosts, the adversary was successful in creating a backdoor user. What is the new username?  

```splunk
index=main host=server EventID=4720
```

![[investigating_splunk_newuser.png]]

**Answer** : A1berto

#### On the same host, a registry key was also updated regarding the new backdoor user. What is the full path of that registry key?

```splunk
index=main Hostname="Micheal.Beaven" EventID="12"
```

![[investigating_splunk_targetobject.png]]

**Answer** : HKLM\SAM\SAM\Domains\Account\Users\Names\A1berto

#### Examine the logs and identify the user that the adversary was trying to impersonate.  

![[investigating_splunk_alberto.png]]

**Answer** : Alberto

#### What is the command used to add a backdoor user from a remote computer?  

```splunk
index=main EventID="1"
```

![[investigating_splunk_commandline.png]]

**Answer** : C:\windows\System32\Wbem\WMIC.exe" /node:WORKSTATION6 process call create "net user /add A1berto paw0rd1

#### How many times was the login attempt from the backdoor user observed during the investigation?  

I search too much before geting the answer

**Answer** : 0

#### What is the name of the infected host on which suspicious Powershell commands were executed?

![[investigating_splunk_james.brown.png]]

**Answer** : James.Browne

#### PowerShell logging is enabled on this device. How many events were logged for the malicious PowerShell execution?

![[investigating_splunk_eventid4103.png]]

**Answer** : 79
#### An encoded Powershell script from the infected host initiated a web request. What is the full URL?

With previous filter there is a base 64 command so we can decode it :

![[investigating_splunk_base64.png]]

we can see there is an other base64 string, so let's see

![[investigating_splunk_cyberchef.png]]

here is our url. we juste need to defang it

**Answer** : hxxp[://]10[.]10[.]10[.]5

