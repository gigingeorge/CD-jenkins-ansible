## Continuous deployment using Jenking via Ansible
We have a webiste created on PHP. 
If we want to make any updates on contents, what we do on AWS was we need to create new launch configuration and new instances are needed to be created with the new contents. After that we need to terminate the intances created with old lauch configuration by adjusting ASG's deletion policy which is cost comsuming

Here we are doing continues deployment using ansible to add the new contents to the existing intances on ASG with the help of Jenkins. 



I created an ansible playbook to create the following
1. Classic load balancer
2. Auto scaling group
3. launch configuration
other basic requirements for the above 
> ##### Requirements:
 >  Jenking and git must be installed on the same machine where ansible installed

#### Jenkins configuration
1. Install Ansible plugin 
2. Manage Jenkins -> amange plugins -> avilable -> ansible -> install without restart -> 
  Restart Jenkins when installation is complete and no jobs are running
3. Setting ansible Location : 
   Manage Jenkins -> Global Tool Configuration  >> under Ansible, mention the ansible execution directory location

##### Creating Job

1. Create an item > source code management > select Git > paste our github URL
2. On Build > Invoke ansible playbook > paste the path where our anible playbook is located

Done with Jenkins

##### Configuring Git

Go to our repository  > settings > webhooks <Add the jenkins machine IP>/<something>-webhook

> Note: The webhook link should be added so that it should be like <IP>/work-webhook

