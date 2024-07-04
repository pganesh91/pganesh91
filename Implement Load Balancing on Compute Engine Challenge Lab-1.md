### **1. First Task**
gcloud config set compute/region `your <REGION>`
 
gcloud config set compute/zone `your <ZONE>`
 
gcloud compute instances create `your <INSTANCENAME>` --machine-type=e2-micro
 

### **Creating the startup.sh file [To create an Instance Template code]**
```
cat << EOF > startup.sh
#! /bin/bash
apt-get update
apt-get install -y nginx
service nginx start
sed -i -- 's/nginx/Google Cloud Platform - '"\$HOSTNAME"'/' /var/www/html/index.nginx-debian.html
EOF
```
### **2. Second Task Here onwards**
### **2.1 Creating Instance from **Template(startup.sh)****
`gcloud compute instance-templates create lb-backend-template --region=<REGION> --network=default --subnet=default --tags=allow-health-check --machine-type=e2-medium --image-family=debian-11 --image-project=debian-cloud --metadata-from-file startup-script=**startup.sh**`
### **2.2 Creating a target pool  before creating Managed Instance Group****
`gcloud compute target-pools create nginx-pool`

### **2.3 creating a managed instance group of 2 nginx web servers**** 
`gcloud compute instance-groups managed create lb-backend-group --base-instance-name nginx --template=lb-backend-template --size=2 --target-pool nginx-pool --zone=<REGION>`
### **2.4 Creating a firewall( _Lab provided Filename_) for tcp:80**** 
`gcloud compute firewall-rules create **<grant-tcp-rule-954>** --allow tcp:80`
### **2.5 Allowing TCP connection through the firewall**** 
`gcloud compute firewall-rules create permit-tcp-rule-522 --network=default --action=allow --direction=ingress --source ranges=130.211.0.0/22,35.191.0.0/16 --target-tags=allow-health-check --rules=tcp:80`
### **2.6 Creating an external IPV4 address**** 
`gcloud compute addresses create lb-ipv4-1 --ip-version=IPV4 --global`
### **2.7 Health check for port 80**** 
`gcloud compute health-checks create http http-basic-check --port 80`
### **2.8 Forwarding Rule to the nginx-pool**** 
`gcloud compute forwarding-rules create nginx-lb --region  <REGION>  --ports=80 --target-pool nginx-pool`
### **2.9 Creating a basic http check**** 
`gcloud compute http-health-checks create http-basic-check`
### **2.10 Defining the ports to http:80 to the managed group**** 
`gcloud compute instance-groups managed set-named-ports lb-backend-group --named-ports http:80`
### **2.11 # create a backend service and attach the managed instance group**** 
Step-1:
`gcloud compute backend-services create web-backend-service --protocol=HTTP --port-name=http --health-checks=http-basic-check --global`
Step-2:
`gcloud compute backend-services add-backend web-backend-service --instance-group=lb-backend-group --instance-group-zone=<ZONE> --global`
### **2.12 # Create a url map and target the HTTP proxy**** 
Step-1:
`gcloud compute url-maps create web-map-http --default-service web-backend-service`
Step-2:
`gcloud compute target-http-proxies create http-lb-proxy --url-map web-map-http`
### **2.13 # Finally creating Forwarding rule to the proxy**** 
`gcloud compute forwarding-rules create http-content-rule --address=lb-ipv4-1 --global --target-http-proxy=http-lb-proxy --ports=80`
### **2.14 # Checking forwarding rules list**** 
`gcloud compute forwarding-rules list`
