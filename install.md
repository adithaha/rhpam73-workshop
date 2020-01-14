Prerequisites
---
- JDK 8 installed
- Modern browser

Installation
---
RHPAM 7.5.1 will be installed on JBoss EAP 7.2
Install JBoss EAP 7.2
- Create folder for RHPAM73, remember no space in directory
- unzip jboss-eap-7.2.0.zip
Install RHPAM 7.5.1
- run rhpam-installer-7.5.1.jar
- set installation path to JBoss EAP 7.2 directory <jboss-eap-7.2>
- Enable Business Central and Process Server
- Username: rhpamAdmin
- Password: admin123!@#
- Next
- Perform Default Configuration
- Finish

Start and Stop RHPAM 7.3
---
- go to cmd/terminal
- Start RHPAM 7.3  
  $ cd <jboss-eap>/bin  
  $ standalone.bat -c standalone-full.xml  
  Open browser: http://localhost:8080/business-central
- Stop RHPAM 7.3
  $ ctrl-c
  
