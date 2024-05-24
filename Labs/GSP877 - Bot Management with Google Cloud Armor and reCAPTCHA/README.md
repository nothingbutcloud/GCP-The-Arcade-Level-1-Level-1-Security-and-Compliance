# [GSP877 - Bot Management with Google Cloud Armor and reCAPTCHA](https://www.cloudskillsboost.google/games/5058/labs/33038)

## Overview
Google Cloud HTTP(S) load balancing is deployed at the edge of Google's network in Google points of presence (POP) around the world. User traffic directed to an HTTP(S) load balancer enters the POP closest to the user and is then load balanced over Google's global network to the closest backend that has sufficient capacity available.

Cloud Armor is Google's distributed denial of service and web application firewall (WAF) detection system. Cloud Armor is tightly coupled with the Google Cloud HTTP Load Balancer and safeguards applications of Google Cloud customers from attacks from the internet.

[reCAPTCHA Enterprise](https://cloud.google.com/recaptcha-enterprise/docs) is a service that builds on the reCAPTCHA API and protects your site from spam and abuse using advanced risk analysis techniques to tell humans and bots apart. Cloud Armor Bot Management provides an end-to-end solution integrating reCAPTCHA Enterprise bot detection and scoring with enforcement by Cloud Armor at the edge of the network to protect downstream applications.

In this lab, you configure an HTTP Load Balancer with a backend, as shown in the diagram below. You set up a reCAPTCHA session token site key and embed it in your website. You also set up redirection to reCAPTCHA Enterprise manual challenge. You then configure a Cloud Armor bot management policy to see how bot detection protects your application from malicious bot traffic.

![HTTP Load Balancer configuration diagram](GSP877-image-1.png)

## What you'll learn

In this lab, you learn how to:

- Set up a HTTP Load Balancer with appropriate health checks
- Create a reCAPTCHA WAF challenge-page site key and associated it with Cloud Armor security policy
- Create a reCAPTCHA session token site key and install it on your web pages
- Create a Cloud Armor bot management policy
- Validate that the bot management policy is handling traffic based on the rules configured

## Solution

Project ID: `qwiklabs-gcp-03-b3e9c12e8cde`
Region: `us-east1`
Zone: `us-east1-d`

### Setup

In Cloud Shell, set up your project ID:

```
export PROJECT_ID=$(gcloud config get-value project)
echo $PROJECT_ID
gcloud config set project $PROJECT_ID
```

Enable APIs:

```
gcloud services enable compute.googleapis.com
gcloud services enable logging.googleapis.com
gcloud services enable monitoring.googleapis.com
gcloud services enable recaptchaenterprise.googleapis.com
```

### Task 1. Configure firewall rules to allow HTTP and SSH traffic to backends

#### Create a firewall rule to allow HTTP traffic to the backends

To create a firewall rule to allow HTTP traffic to the backends, use the following command:

```
gcloud compute firewall-rules create default-allow-health-check --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:80 --source-ranges=130.211.0.0/22,35.191.0.0/16 --target-tags=allow-health-check
```

Create a firewall rule to allow SSH-ing into the instances:

```
gcloud compute firewall-rules create allow-ssh --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:22 --source-ranges=0.0.0.0/0 --target-tags=allow-health-check
```

### Task 2. Configure instance templates and create managed instance groups

#### Configure the instance templates

Run the following command:

```
gcloud compute instance-templates create lb-backend-template --project=qwiklabs-gcp-03-b3e9c12e8cde --machine-type=n1-standard-1 --network-interface=network-tier=PREMIUM,subnet=default --metadata=startup-script=\#\!\ /bin/bash$'\n'sudo\ apt-get\ update$'\n'sudo\ apt-get\ install\ apache2\ -y$'\n'sudo\ a2ensite\ default-ssl$'\n'sudo\ a2enmod\ ssl$'\n'sudo\ su$'\n'vm_hostname=\"\$\(curl\ -H\ \"Metadata-Flavor:Google\"\ \\$'\n'http://metadata.google.internal/computeMetadata/v1/instance/name\)\"$'\n'echo\ \"Page\ served\ from:\ \$vm_hostname\"\ \|\ \\$'\n'tee\ /var/www/html/index.html,enable-oslogin=true --maintenance-policy=MIGRATE --provisioning-model=STANDARD --service-account=401603670420-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --region=us-east1 --tags=allow-health-check,http-server,https-server,lb-health-check --create-disk=auto-delete=yes,boot=yes,device-name=lb-backend-template,image=projects/debian-cloud/global/images/debian-12-bookworm-v20240515,mode=rw,size=10,type=pd-balanced --no-shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring --reservation-affinity=any
```

Wait for the instance template to be created.

#### Create the managed instance group

```
gcloud beta compute instance-groups managed create lb-backend-example --project=qwiklabs-gcp-03-b3e9c12e8cde --base-instance-name=lb-backend-example --template=projects/qwiklabs-gcp-03-b3e9c12e8cde/global/instanceTemplates/lb-backend-template --size=1 --zone=us-east1-d --default-action-on-vm-failure=repair --no-force-update-on-repair --standby-policy-mode=manual --list-managed-instances-results=PAGELESS
```

Wait for the instance group to be created.

```
gcloud beta compute instance-groups managed set-autoscaling lb-backend-example --project=qwiklabs-gcp-03-b3e9c12e8cde --zone=us-east1-d --mode=off --min-num-replicas=1 --max-num-replicas=1 --target-cpu-utilization=0.6 --cool-down-period=60
```

Wait for autoscaler to be created.

#### Add a named port to the instance group

For your instance group, in Cloud Shell, define an HTTP service and map a port name to the relevant port:

```
gcloud compute instance-groups set-named-ports lb-backend-example \
--named-ports http:80 \
--zone us-east1-d
```


###


###


###



