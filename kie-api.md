Prerequisites
---
- RHPAM installed
- Modern browser
- REST client (postman, etc)

Start and Stop RHPAM
---
- go to cmd/terminal
- Start RHPAM  
  $ cd <jboss-eap>/bin  
  $ standalone.bat -c standalone-full.xml  
  Open browser: http://localhost:8080/business-central
- Stop RHPAM
  $ ctrl-c
  
  
Prepare user
---
$ cd bin  
$ add-user.bat -a -u approver1 -p approver1 -g user,approver  
$ add-user.bat -a -u manager1 -p manager1 -g user,manager  
  
Deploy sample application
---


Access Mortgage process via API
---
container id: mortgage-process_1.0.0-SNAPSHOT  
process id: Mortgage_Process.MortgageApprovalProcess

### START PROCESS (admin)
1. start process  
review the body request
```
https://raw.githubusercontent.com/adithaha/rhpam73-workshop/master/start.json
```
Use any REST client you have, note process instance id response
```
URL: http://localhost:8080/kie-server/services/rest/server/containers/mortgage-process_1.0.0-SNAPSHOT/processes/Mortgage_Process.MortgageApprovalProcess/instances
Method: POST
Basic Auth: rhpamAdmin:admin123!@#
Header: 
- accept: application/json
- content-type: application/json
Body: copy from https://raw.githubusercontent.com/adithaha/rhpam73-workshop/master/start.json
```
Curl command
```
curl -X POST -u 'rhpamAdmin:admin123!@#' --data @start.json "http://localhost:8080/kie-server/services/rest/server/containers/mortgage-process_1.0.0-SNAPSHOT/processes/Mortgage_Process.MortgageApprovalProcess/instances" -H  "accept: application/json" -H  "content-type: application/json" 
```

### QUALIFY TASK (approver)
2. get task inbox, choose task id  
Note task id that related to same process instance id in #1
```
URL: http://localhost:8080/kie-server/services/rest/server/queries/tasks/instances/pot-owners?page=0&pageSize=10&sortOrder=true
Method: GET
Basic Auth: approver1:approver1
Header: 
- accept: application/json
- content-type: application/json
```
Curl command
```
curl -X GET -u 'approver1:approver1' "http://localhost:8080/kie-server/services/rest/server/queries/tasks/instances/pot-owners?page=0&pageSize=10&sortOrder=true" -H  "accept: application/json"
```

3. review task data input  
Review task data with id from #2, replace <task-id> from url
```
URL: http://localhost:8080/kie-server/services/rest/server/containers/mortgage-process_1.0.0-SNAPSHOT/tasks/<task-id>/contents/input
Method: GET
Basic Auth: approver1:approver1
Header: 
- accept: application/json
- content-type: application/json
```
Curl command
```
curl -X GET -u 'approver1:approver1' "http://localhost:8080/kie-server/services/rest/server/containers/mortgage-process_1.0.0-SNAPSHOT/tasks/<task-id>/contents/input" -H  "accept: application/json"
```
  
4. claim task id
```
URL: http://localhost:8080/kie-server/services/rest/server/containers/mortgage-process_1.0.0-SNAPSHOT/tasks/<task-id>/states/claimed
Method: PUT
Basic Auth: approver1:approver1
Header: 
- accept: application/json
- content-type: application/json
```
Curl command
```
curl -X PUT -u 'approver1:approver1' "http://localhost:8080/kie-server/services/rest/server/containers/mortgage-process_1.0.0-SNAPSHOT/tasks/19/states/claimed" -H  "accept: application/json"
```

5. start task id
```
URL: http://localhost:8080/kie-server/services/rest/server/containers/mortgage-process_1.0.0-SNAPSHOT/tasks/<task-id>/states/started
Method: PUT
Basic Auth: approver1:approver1
Header: 
- accept: application/json
- content-type: application/json
```
Curl command
```
curl -X PUT -u 'approver1:approver1' "http://localhost:8080/kie-server/services/rest/server/containers/mortgage-process_1.0.0-SNAPSHOT/tasks/<task-id>/states/started" -H  "accept: application/json"
```

6. complete task id
review the body request
```
https://raw.githubusercontent.com/adithaha/rhpam73-workshop/master/qualify.json
```
Use any REST client you have, note process instance id response
```
URL: http://localhost:8080/kie-server/services/rest/server/containers/mortgage-process_1.0.0-SNAPSHOT/tasks/<task-id>/states/completed
Method: PUT
Basic Auth: approver1:approver1
Header: 
- accept: application/json
- content-type: application/json
Body: copy from https://raw.githubusercontent.com/adithaha/rhpam73-workshop/master/qualify.json
```
Curl command
```
curl -X PUT -u 'approver1:approver1' --data @qualify.json "http://localhost:8080/kie-server/services/rest/server/containers/mortgage-process_1.0.0-SNAPSHOT/tasks/19/states/completed" -H  "accept: application/json" -H "content-type: application/json"
```

### FINAL APPROVAL TASK (manager)
6. get task inbox, choose task id
curl -X GET -u 'manager1:manager1' "http://localhost:8080/kie-server/services/rest/server/queries/tasks/instances/pot-owners?page=0&pageSize=10&sortOrder=true" -H  "accept: application/json"

7. review task data input
curl -X GET -u 'manager1:manager1' "http://localhost:8080/kie-server/services/rest/server/containers/mortgage-process_1.0.0-SNAPSHOT/tasks/20/contents/input" -H  "accept: application/json"

8. claim task id
curl -X PUT -u 'manager1:manager1' "http://localhost:8080/kie-server/services/rest/server/containers/mortgage-process_1.0.0-SNAPSHOT/tasks/20/states/claimed" -H  "accept: application/json"

9. start task id
curl -X PUT -u 'manager1:manager1' "http://localhost:8080/kie-server/services/rest/server/containers/mortgage-process_1.0.0-SNAPSHOT/tasks/20/states/started" -H  "accept: application/json"

10. complete task id
curl -X PUT -u 'manager1:manager1' "http://localhost:8080/kie-server/services/rest/server/containers/mortgage-process_1.0.0-SNAPSHOT/tasks/20/states/completed" -H  "accept: application/json" -H "content-type: application/json"


— REVIEW COMPLETED PROCESS INSTANCE (admin) —
11. review completed process instance, process-instance-state=2 means already completed
curl -X GET -u 'rhpamAdmin:admin123!@#' "http://localhost:8080/kie-server/services/rest/server/containers/mortgage-process_1.0.0-SNAPSHOT/processes/instances/35" -H  "accept: application/json"
