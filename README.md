## Continuous deployment using Jenking via Ansible
We have a webiste created on PHP. 
If we want to make any updates on contents, what we do on AWS was we need to create new launch configuration and new instances are needed to be created with the new contents. After that we need to terminate the intances created with old lauch configuration by adjusting ASG's deletion policy. 

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

## Deployment

The code to deploy contents will be

```sh
- name: "updating ASG"
  hosts: newasg
  become: true
  serial: 1
  vars:
    git_url: "https://github.com/gigingeorge/aws-elb-site.git"
    owner: "apache"
    group: "apache"
  tasks:
    - name: "downloading from git"
      git:
        repo: "{{git_url}}"
        dest: /var/myweb/
      register: git_status
    - name: "disabling health check"
      when: git_status.changed == true
      file:
        path: "/var/www/html/health.php"
        state: touch
        mode: 0000
    - name: Pause to offload from elb
      when: git_status.changed == true
      pause:
        seconds: 25
    - name: copying contents
      when: git_status.changed == true
      copy:
        src: /var/myweb/
        dest: /var/www/html/
        owner: "{{owner}}"
        group: "{{group}}"
        remote_src: true
    - name: "Enabling health check"
      when: git_status.changed == true
      file:
        path: "/var/www/html/health.php"
        state: touch
        mode: 0444
    - name: Pause to  load to elb
      when: git_status.changed == true
      pause:
        seconds: 25
```
The code is configured such that:

1. Whenever the update occurs, it should complete the update one intance at a time which is defined using 
```sh
serial: 1

The playbook will get executed only after completing the task on each instance. If not, if the contents are big in size, applying to every intance at a time will cause the website to be down. 
```

#### Process flow

1. I've created a health.php to perform health check for the load balancer. So before the update needs to be occurred, the intance should get excluded from load balancer. 
```sh
    - name: "disabling health check"
      when: git_status.changed == true
      file:
        path: "/var/www/html/health.php"
        state: touch
        mode: 0000
```

2. Once excluded, it need some time to complete the health check. So we need to pause the playbook for some seconds.
```sh
 - name: Pause to offload from elb
      when: git_status.changed == true
      pause:
        seconds: 25
```
3.  Now the instances are excluded from the load balancer and we can now perform the content update. 
```sh
    - name: copying contents
      when: git_status.changed == true
      copy:
        src: /var/myweb/
        dest: /var/www/html/
        owner: "{{owner}}"
        group: "{{group}}"
        remote_src: true
```
4. Once completed, we need to add the intance back to load balancer.
```sh
    - name: "Enabling health check"
      when: git_status.changed == true
      file:
        path: "/var/www/html/health.php"
        state: touch
        mode: 0444
    - name: Pause to  load to elb
      when: git_status.changed == true
      pause:
        seconds: 25
```

![alt text](https://i.ibb.co/420h0K5/Screenshot.png)


That's it. 



--------------------------------- _Gigin George_ ------------------------------------------------------
 
 
--------------------------------- _gigingkallumkal@gmail.com_ ----------------------------------------
 
 
--------------------------------------- _linkedin.com/in/gigin-george-648679142_ ---------------------
