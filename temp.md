Prerequisites
---
- JDK 8 installed
- Modern browser

Installation
---
RHPAM 7.3 will be installed on JBoss EAP 7.2
Install JBoss EAP 7.2
- Create folder for RHPAM73, remember no space in directory
- unzip jboss-eap-7.2.0.zip
Install RHPAM 7.3
- run rhpam-installer-7.3.0.jar
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
- Stop RHPAM 7.3
  $ ctrl-c
  
Create Project
---
login-project-addproject
- Name: SKBPPh22Emas
- Group ID: id.go.pajak.workflow
- Artifact ID: SKBPPh22Emas
- Version: 1.0.0-SNAPSHOT
- Add


Create Data Object
---
WP
---
Add Asset - Data Object
- Name: WP
- Package: id.go.pajak.workflow.skbpph22emas
- OK
  + Add Field
    - Id: npwp
    - Label: NPWP
    - Type: String
    - Create and continue
    - Id: namaWP
    - Label: Nama WP
    - Type: String 
    - Create and continue
    - Id: alamatWP
    - Label: Alamat WP
    - Type: String
    - Create
    - Save

Permohonan
---
SKBPPh22Emas
Add Asset - Data Object
- Name: Permohonan
- Package: id.go.pajak.workflow.skbpph22emas
- OK
  + Add Field
    - Id: no
    - Label: Nomor Permohonan
    - Type: String
    - Create and continue
    - Id: tanggal
    - Label: Tanggal Permohonan
    - Type: Date
    - Create and continue
    - Id: wp
    - Label: Wajib Pajak
    - Type: id.go.pajak.workflow.skbpph22emas.WP
    - Create
    - Save

Keputusan
---
SKBPPh22Emas
Add Asset - Data Object
- Name: Keputusan
- Package: id.go.pajak.workflow.skbpph22emas
- OK
  + Add Field
    - Id: docNo
    - Label: Nomor Dokumen
    - Type: String
    - Create and continue
    - Id: tanggal
    - Label: Tanggal Keputusan
    - Type: Date
    - Create and continue
    - Id: tanggal
    - Label: Tanggal Berakhir
    - Type: Date
    - Create and continue
    - Id: terima
    - Label: Diterima
    - Type: Boolean
    - Create and continue
    - Id: alasan
    - Label: Alasan
    - Type: String
    - Create
    - Save


Create Business Process
---
Add Asset - Business Process
- Business Process: SKBPPh22EmasProcess
- Package: id.go.pajak.workflow.skbpph22emas
- OK
  - Business Process - Properties - Process Data
    - + 
    - Name: permohonan
    - Type: Permohonan
    - + 
    - Name: keputusan
    - Type: Keputusan
    - + 
    - Name: setujuKasiWaskon
    - Type: Boolean
    - + 
    - Name: setujuKepalaKantor
    - Type: Boolean
  - Green circle - Create Parallel - Convert into Exclusive
  - X Parallel - Create Task - Convert into User - Properties
    - Name: Membuat/Menolak SKB 22 Impor Emas
    - Implementation/Execution
      - Task Name: ProsesSKB22ImporEmas
      - Groups - Name: Approver
      - Assignment
        - Data Input and Assignment
          + Add
          - Name: permohonan
          - Data Type: Permohonan
          - source: permohonan
        - Data Output and Assignment
          + Add
          - Name: keputusan
          - Data Type: Keputusan
          - source: keputusan
        - Save
  - Membuat/Menolak SKB 22 Impor Emas - Create Task - Convert into User - Properties
    - Name: Persetujuan Kasi Waskon
    - Implementation/Execution
      - Task Name: PersetujuanKasiWaskon
      - Groups - Name: Approver
      - Assignment
        - Data Input and Assignment
          + Add
          - Name: permohonan
          - Data Type: Permohonan
          - source: permohonan
          + Add
          - Name: keputusan
          - Data Type: Keputusan
          - source: keputusan
        - Data Output and Assignment
          + Add
          - Name: setujuKasiWaskon
          - Data Type: Boolean
          - source: setujuKasiWaskon
        - Save
  - Persetujuan Kasl Waskon - Create Parallel - Convert into Exclusive - Create Sequence Flow - to first X Parallel
    - Organize arrow if needed
    - Arrow - Properties
      - Name: Tidak
      - Implementation/Execution
        - Condition
        - Process Variable: setujuKasiWaskon
        - Condition: is false
  - Second X Parallel - Create Task - Convert into User - Select arrow
    - Organize arrow if needed
    - Arrow - Properties
      - Name: Ya
      - Implementation/Execution
        - Condition
        - Process Variable: setujuKasiWaskon
        - Condition: is true  
  - Select User Task - Properties
    - Name: Persetujuan Kepala Kantor
    - Implementation/Execution
      - Task Name: PersetujuanKepalaKantor
      - Groups - Name: Approver
      - Assignment
        - Data Input and Assignment
          + Add
          - Name: permohonan
          - Data Type: Permohonan
          - source: permohonan
          + Add
          - Name: keputusan
          - Data Type: Keputusan
          - source: keputusan
        - Data Output and Assignment
          + Add
          - Name: setujuKepalaKantor
          - Data Type: Boolean
          - source: setujuKepalaKantor
        - Save
  - Persetujuan Kepala Kantor - Create Parallel - Convert into Exclusive - Create Sequence Flow - to first X Parallel
    - Organize arrow if needed
    - Arrow - Properties
      - Name: Tidak
      - Implementation/Execution
        - Condition
        - Process Variable: setujuKepalaKantor
        - Condition: is false
  - Third X Parallel - Create Task - Convert into Script - Select arrow
    - Organize arrow if needed
    - Arrow - Properties
      - Name: Ya
      - Implementation/Execution
        - Condition
        - Process Variable: setujuKepalaKantor
        - Condition: is true  
  - Select Script Task - Properties
    - Name: Mencetak SKB 22 impor emas/Penolakan SKB 22 impor emas
    - Implementation/Execution
      - Task Name: PersetujuanKepalaKantor
      - Assignment
        - Script
          - System.out.println(permohonan.getWp().getNpwp());
            System.out.println(keputusan.getTerima());
          - java
  - Mencetak SKB 22 impor emas/Penolakan SKB 22 impor emas - Create End
  - Validate
  - Save


Generate Forms
---
- Form Generation - Generate All Forms
- Refresh Object Explorer
- Review forms
  - Forms - SKBPPh22Emas.ProsesSKB22ImporEmas-taskform
    - Remove Keputusan Form
    - Remove Setuju Kasi Waskon
    - Remove Setuju Kepala Kantor
    - Save
  - Forms - Permohonan
    - Tanggal Permohonan - Edit
      - Uncheck Show Time
      - Check Required
      - + OK
    - Save
  - Forms - WP
    - Nama WP - Edit
      - Check Required
      - + OK
    - NPWP - Edit
      - Check Required
      - + OK
    - Alamat WP - Edit
      - Check Required
      - + OK
    - Save
  - Forms - Keputusan
    - Nomor Dokumen - Edit
      - Check Required
      - + OK
    - Tanggal Keputusan - Edit
      - Uncheck Show Time
      - Check Required
      - + OK
    - Tanggal Berakhir - Edit
      - Uncheck Show Time
      - + OK
    - Diterima - Edit
      - Check Required
      - + OK
    - Save

Deploy
---
- Go to SKBPPh22Emas - Deploy

Create Approver User
---
- Go to cmd/terminal
  $ cd <jboss-eap>/bin
  $ add-user.bat
    type of user: b
    Username: user1
    Password: user1
    Are you sure you want to use the password entered yes/no? yes
    Re-enter Password: user1
    Groups: user,approver
    Is this correct: yes
    slave host: no

Trying business process
---
- Login to business central as user1 http://localhost:8080/business-central
- Manage - Process Definition
- SKBPPh22EmasProcess - New Process Instance
  - Initial Form -> Fill initial input
  - Membuat/Menolak SKB 22 Impor Emas -> Menu - Track - Task Inbox
    - task Membuat/Menolak SKB 22 Impor Emas
    - Claim
    - Start
    - Fill Keputusan form
    - Complete
  - Persetujuan Kasi Waskon -> Menu - Track - Task Inbox
    - task Persetujuan Kasi Waskon
    - Claim
    - Start
    - Check Menyetujui Kasi Waskon
    - Complete
  - Persetujuan Kepala Kantor -> Menu - Track - Task Inbox
    - task Persetujuan Kepala Kantor
    - Claim
    - Start
    - Check Menyetujui Kasi Waskon
    - Complete
- View Completed Process Instance
  - Menu - Process Instances
  - Check Completed status
  - Completed SKBPPhEmasProcess - View Tasks
  - You should be able to see all completed tasks

Accessing RHPAM API
---
- Open swagger definition at browser http://localhost:8080/kie-server/docs/#/
- Getting container id using REST API
  - KIE Server and KIE containers - /servers/containers - try it out
  - Execute
  - Note down container id, ie. SKBPPh22Emas_1.0.0-SNAPSHOT
- Getting process id using REST API
  - Process queries - /server/queries/containers/{containerId}/processes/definitions - try it out
  - container id: <container-id> (ie. SKBPPh22Emas_1.0.0-SNAPSHOT)
  - Execute
  - Note down process id, ie. SKBPPhEmas.SKBPPhEmasProcess
- Starting process instance using REST API
  - Proces instances - server/containers/{containerId}/processes/{processId}/instances
  - container id: <container-id> (ie. SKBPPh22Emas_1.0.0-SNAPSHOT)
  - process id: <process-id> (ie. SKBPPhEmas.SKBPPhEmasProcess)
  - body:
    {
        "permohonan": {
            "Permohonan": {
                "no": "111",
                "tanggal": "2019-04-18",
                "wp": {
                     "namaWP": "aditya",
                     "npwp": "123",
                     "alamatWP": "jakarta"
                } 
            }
        }
    }
  - Execute
- Check if process instance is created on business central web
  - Menu - Manage - Process Instances
   
