# IRAutomation-AWS
Rapid Automation Incident Response on AWS workload

1.Instruction 	
Security teams need to quickly detect, quickly recognize incidents, and respond to prevent spread through
notifications of many breach threats occurring in AWS workloads

  1.1 Necessity 

The security team should use a number of 3rd party solutions in with AWS security services to quickly perform the detection, isolation, and analysis procedures in case of an AWS workload incident.
However, since the analysis and response procedures are not automated after the detection stage, there are difficulties in the analysis and response.

The incident response from the previous security team had the following issues.

①.	Incident notification
It is difficult for the security team to quickly recognize the detection history after an incident.
Dissemination of incident details is not easy.

②.	Incident Analysis
As incident analysis is not automated, there is a delay in manual analysis.
Incorrect incident analysis integrity occurs while manual incident analysis

③.	Analysis Data Collect
Incident analysis data is not organized.
There is no environment in which detailed analysis data can be stored and analyzed as needed.

④.	Multi Account
There is no process for incident response when using multiple accounts.
There is no guide to the analysis procedure in the security team.

In order to solve the above issues, this solution automates the detection and rapid recognition of intrusion threats in multi account situations. Through this, the security team updates and recognizes the current situation. Perform analysis automation. The analyzed result is saved in the S3 Bucket through history management and managed in the security team account.

The automated process consists of AWS Lambda Functions in the security team account, and performs isolations, take a snapshot, and forensic for memory dumps through the assume roll in the target account.

The command is executed in the incident account through Systems Manager SSM, and after taking a snapshot to the security team account, a forensic instance required for incident analysis is configured and attached.

Afterwards, it is configured to automatically go through a predefined forensic step-by-step procedure. Such as memory forensics, disk dump, anomaly file search, history search, and when completed forensic steps, result export to the S3 bucket of the security team account.

The progress issue history is sent to Slack notifications, and the entire process of the progress in real time is provided by configuring a AWS Step Function.


2. ARCITECTURE SOLUTION

![image](https://user-images.githubusercontent.com/10083600/120257574-71e64c80-c2cb-11eb-9ec6-40cba1984af8.png)

3. Pre-requisties
 Activate AWS GuardDuty on your AWS account
 Have a Slack channel ready. Alerts will be sent to that channel
 Download the YAML templates and two Lambda functions ZIP code and save them into one of your S3 bucket
  - Lambda ZIP file is for the function sending auto scaling notifications to the Slack channel
  - the other one contains the code of all the incident response and forensic Lambda functions
* Create SSH Key Pairs for your EC2 instances (In the __EC2 console__, go to __Network & Security__ > __Key Pairs__). 
* If you choose to enable VPC Flow Logs to S3, have a bucket ready for it
* Prepare a S3 bucket where the outputs of the forensic analysis will be stored.


2.2.1	Deploy Target Account
Target Account is the account that can be incident on in the OU configuration
Security personnel perform automated forensics on target accounts for detection and forensics in the event of an incident.

 

 
Picture II.	GuardDuty Enable

The following configures the STS Assume Role of the Target Account. 
Picture III.	Trust Relationship Edit
 
Picture IV.	sts Assume Role Edit

Next, add permissions for automating EC2 incident analysis in the event of an incident. 
Picture V.	Permission config base

Add the permission as below. 
Picture VI.	Insert Permission Edit
2.2.2	Deploy Security Account
Security Account is the GaurdDuty Master account in the OU configuration.
Security personnel perform automated forensics on target accounts for detection and forensics in the event of an incident.

 

 
Picture VII.	GuardDuty Enable

Next, upload the code to the S3 Bucket for Lambda configuration and deploy.
 
Picture VIII.	Lambda Function Code Deploy

Here is the code I uploaded: 
Picture IX.	Lambda Code

After uploading the code, copy the S3 URL of the YAML file to configure CloudFormation Stack. 
Picture X.	CloudFormation Stack 
Next, after the Stack is configured, configure the automation to be performed through STS Assume role the target account.  
Picture XI.	Security Account STS Assume Role

After the code is configured, check the Lambda Function to see if the stepping code is applied. 
Picture XII.	Lambda Functions

Among the steps for response after the automation configuration, the Isolation step automates the movement of the target account to a separate security group other than the service section for network isolation when an incident occurs.

Enter the quarantine SG of the Target Account by modifying the Environment in step 4 in the Lambda Function as shown below.
 
 
Picture XIII.	Environment Config

 
3. Architecture Detail  
3.1 Detect
The first step is the Finding process. Here's what's done:

•	GuardDuty Finding – High Severity, EC2 Findings 18
•	GuardDuty Finding in case of EC2 Instance based incident
•	Slack Channel notification or SNS topic notification through CloudWatch Event after GuardDuty Finding

 
Picture XIV.	Step Detect

3.2 Isolation
The next step is to prevent the spread through network isolation immediately after an incident occurs. Make sure the Security Group is quarantined.

•	Incident Moved from EC2 Instance Production Environment Security Group to Quarantine Security Group

 
Picture XV.	Step Isolation
3.3 Response
The post-isolation stage proceeds to the stage for analysis.
3.3.1 Memory Dump

 The first step is to do a real-time memory dump on the incident EC2 Instance.

•	Memory Dump – Based on Real Data
•	Perform a memory dump before saving snapshots after quarantine
•	Perform Memory Dump through Lime and save /forensic/ folder
•	Adjustable memory dump steps based on real-time data needs

 
Picture XVI.	Step Response - Memory

3.3.2 Create Snapshot & Forensic instance

 After the memory dump, a snapshot is taken as a step for analysis in the security account, and then the snapshot is shared with the Security account to prepare for analysis.

•	EBS Snapshot – Incident Instance
•	Snapshot attached for detailed analysis after performing instance snapshot
•	Performed after Isolation Instance progress
•	EBS Volume Attached to Forensic Instance

 
Picture XVII.	Step Response – Create Snapshot
3.4 Analysis
In the analysis stage, the investigation items required for actual incident analysis are configured in advance and automated to respond in case of an incident.

3.3.1 Memory Analysis

•	Forensic Automation of Captured Memory
•	It is a configuration to analyze dumped memory and review application according to requirements

 
Picture XVIII.	Step Analysis - Memory

3.3.2 Create Snapshot & Forensic instance

•	Proceed with the Snapshot Volume analysis procedure
•	Configure Automation Script to analyze the EC2 Instance target where the incident occurred
•	Process, Network, var/log, Find shell, Behavior File

 
Picture XIX.	Step Analysis – Snapshot Forensic
![image](https://user-images.githubusercontent.com/10083600/120257852-f3d67580-c2cb-11eb-9fac-41453c7c4afe.png)
