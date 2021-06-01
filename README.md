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

4. Deplooy
Security Account is the GaurdDuty Master account in the OU configuration.
Security personnel perform automated forensics on target accounts for detection and forensics in the event of an incident.

 

 
![image](https://user-images.githubusercontent.com/10083600/120258104-69424600-c2cc-11eb-9b07-9ab7e5cea247.png)
