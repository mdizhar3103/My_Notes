Vulnerability Management Process
--------------------------------

```bash

+--> Plan -> Scan -> Prioritize -> Report -> Remdiate -> Verfiy --+
|                                                                 |
+---<-------------------<----------------------------<------------+

```

Qualys is installed in internal Network and scans all network and send data to Qualys Cloud

Qualys agent is installed on each servers which monitor them and send data to Qualys Cloud

**External Scanner** is pre-configured and ready to use.
- Provides External Risk View (that a hacker on internet have)
- We can open firewall and allow external scanner to scan internal servers. (Not recommended way to do, because it require insecure firewall rule)
- This can be used to scan external servers such as servers in DMZ.

### Internal Scanner
- Deployed in internal Networks
- Can be a VM or an appliance
- Benefits is that it allows network segregation (no need for insecure firewall allowing all traffic), but require only one rule to allow communication between internal scanner and to Qualys Cloud.
- Can have multiple internal server. (Allow to do scan multiple network at same time)

### Cloud Agents
- Should be installed in specific computers like person laptops and computers.
- Allows mobility without using visibility
- Provide real time visibility 

### Qualys Console
- Login to Qualys Community Edition (you will get an url, username and password to access Qualys Lab)
- After Login, go to Scans tab, then click on Appliance and click on Virtual Scanner Appliance, then click on download image only option.
- Download based on you choice (Default choose standard option)
- Once Image is downloaded close the panel and go to virtual appliance section and click on I have image option
- Give a name to virtual scanner, click on next and note the personalization code, and enter the code in VM you created via importing vdi image.
- Now then click on activation and all goes well click on done.


### Scan Types

|Unauthenticated Scan | Authenticated Scan|
|---------------------|-------------------|
|Also called Black Box Testing or remote scan | White Box Testing or internal scan |
|Scan the server from external point of view | Requires credentials to scan servers from internal point of view | 
| Identify and assess externally avaiable services | Check installed software versions and products |
| Depending upon the policy, may miss open services | Detect internal and external vulnerabilities |
|  | Gives more accurate results | 
| Scanning might be slow | Faster Scans |

### Scan Option Profiles
- Provides the flexibility, to choose the scanning methods depending upon the usecase
- Defines how the scan will be performed, which ports to scan, how to probe ports, intensity of scans, etc

### Scanned Vulnerabilities Types
- Confirmed (Services are vulnerable)
- Potential (Services/app might be vulnerable)

### WebScans
- Web Application Vulnerability Scanning
- Web Pages security flaws like SQL injection, XSS, etc.
- Identify Missing Patches
