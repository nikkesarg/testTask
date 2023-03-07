## Devops Trial Task
# First Part 
You will need to install the following services using Ansible (preferred) or Terraform:
* a complete Gitlab installation
* a functional Grafana/Prometheus installation
* setup access to Gitlab and Grafana using Nginx  

# Second Part
Redo the exercises of the first part, but install it in Kubernetes (You can use minikube)  
**Bonuses:** Installation in Kubernetes using Helm  
**IMPORTANT:** When you finish, please provide the following:  
* access to the installation (for instance, https://13.35.46.34:8000/gitlab)
* SSH access to the server
* access to your source code (files you used for parts 2 & 3). You can provide a ZIP archive for the source code. If you don’t have any server instance to work on, you can provide a list of commands/configuration to be run to achieve the results.

**Other notes:**
* Ideally, use CentOS / dnf as this is our environment at Kameleoon.
* If you can use domain names for Gitlab and Grafana, that's better as well (i.e. https://www.mydomain.com/gitlab or https://www.mydomain.com/zabbix better rather than https://13.35.46.34:8000/gitlab).
* This can take you up to 6 hours and we don't expect you to finish all of it, so please focus on what you can do within these timeframe.


-------
# Solution

Since I do not have a personal cloud account(AWS/Azure/GCS) and the use of corporate resources is not possible for security reasons, the debugged solutions for this test task are provided below.  
All solutions were based on the principle of using ready-made solutions instead of inventing a bicycle.

## 1. The solution based on ansible playbooks
1. The solution of the first part of the task on ansible is available in the `ansible` folder.  
  *  All required services are raised in containers in order to avoid clutter on the host VM.  
  *  To deploy it, you need to install ansible + ansible-galaxy on the control vm(in any way convenient for you) and docker-compos with all dependencies
   `pip3 install docker-compose`  
  * The target VM (where the required containers will be deployed) can also act as a control VM.
  * the solution uses roles from the galaxy collection:
     - alexislessard.docker-prometheus
     - bsmeding.nginx_docker
     - yabusygin.gitlab

since these roles have not been updated for a long time, they has been debugged and improved by myself (for example, in the last grafana, the datasore task structure has changed. The default user has been switched from root to user, etc.)
Well-adjusted roles are also attached to the solution. The playbook is installed using the classic command:
`ansible-playbook -i inventories/servers  playbooks/deploy-gitlab.yml`

**because it is impossible to provide a dedicated host for this test task**, to access grafana and gitlab sites after deployment, you can write domain aliases in /etc/hosts like this:
```
127.0.1.1     grafana.example.com gitlab.example.com
```

For the same reason, **LetsEncrypt is not activated** to issue ssl-certificates and no forward http to  httpps.

## The solution based on helm charts  
The solution using helm charts, within the framework of this task, essentially boils down to the use of custom files with variables. You can use the suggested `prometheus_values.yaml` file to install the prometheus stack
  to install the gitlab server from the chart - the main variables are set from the launch line. Here are examples of chart deployments:
```
Install Prometheus Stack
* helm upgrade --install prometheus prometheus-community/kube-prometheus-stack --values prometheus_values.yaml  -n prometheus  

#Install Gitlab Server
* helm upgrade --install gitlab gitlab/gitlab --timeout 600s  --set global.hosts.domain=gitlab.example.com \
  --set global.hosts.externalIP=10.10.10.10   --set certmanager-issuer.email=me@gitlab.example.com \
  --set postgresql.image.tag=13.6.0
```

It is assumed that an ingress controller is already installed in the cluster in accordance with the type of cluster (EKS / AKS / GKE / on-prem), so the appropriate annotations are set in the charts

