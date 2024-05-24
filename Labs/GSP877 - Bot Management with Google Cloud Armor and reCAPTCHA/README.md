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

Project ID: `qwiklabs-gcp-03-f8adc4abc8c5`
Region: `europe-west1`
Zone: `europe-west1-c`

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

Follow the instructions from the lab.

Startup Script:
```
#! /bin/bash
sudo apt-get update
sudo apt-get install apache2 -y
sudo a2ensite default-ssl
sudo a2enmod ssl
sudo su
vm_hostname="$(curl -H "Metadata-Flavor:Google" \
http://metadata.google.internal/computeMetadata/v1/instance/name)"
echo "Page served from: $vm_hostname" | \
tee /var/www/html/index.html
```

Wait for the instance template to be created.

#### Create the managed instance group

Follow the instructions from the lab or run the following command:

```
gcloud beta compute instance-groups managed create lb-backend-example --project=qwiklabs-gcp-03-f8adc4abc8c5 --base-instance-name=lb-backend-example --template=projects/qwiklabs-gcp-03-f8adc4abc8c5/global/instanceTemplates/lb-backend-template --size=1 --zone=europe-west1-c --default-action-on-vm-failure=repair --no-force-update-on-repair --standby-policy-mode=manual --list-managed-instances-results=PAGELESS && gcloud beta compute instance-groups managed set-autoscaling lb-backend-example --project=qwiklabs-gcp-03-f8adc4abc8c5 --zone=europe-west1-c --mode=off --min-num-replicas=1 --max-num-replicas=1 --target-cpu-utilization=0.6 --cool-down-period=60
```

Wait for the instance group to be created.

#### Add a named port to the instance group

For your instance group, in Cloud Shell, define an HTTP service and map a port name to the relevant port:

```
gcloud compute instance-groups set-named-ports lb-backend-example \
--named-ports http:80 \
--zone us-east1-d
```

### Task 3. Configure the HTTP Load Balancer

Follow the instructions from the lab.

### Task 4. Create and deploy reCAPTCHA session token and challenge-page site key

#### Create reCAPTCHA session token and WAF challenge-page site key

1. Create the reCAPTCHA session token site key and enable the WAF feature for the key:

```
gcloud recaptcha keys create --display-name=test-key-name \
  --web --allow-all-domains --integration-type=score --testing-score=0.5 \
  --waf-feature=session-token --waf-service=ca
```

2. Create the reCAPTCHA WAF challenge-page site key and enable the WAF feature for the key. You can use the reCAPTCHA challenge page feature to redirect incoming requests to reCAPTCHA Enterprise to determine whether each request is potentially fraudulent or legitimate. Later, you associate this key with the Cloud Armor security policy to enable the manual challenge. This lab refers to this key as `CHALLENGE-PAGE-KEY` in the later steps.

```
gcloud recaptcha keys create --display-name=challenge-page-key \
--web --allow-all-domains --integration-type=INVISIBLE \
--waf-feature=challenge-page --waf-service=ca
```

3. Navigate to Navigation menu (Navigation menu icon) > Security > reCAPTCHA Enterprise. You should see the keys you created in the Enterprise Keys list.

Capture the key id to be used in the next section.



#### Implement reCAPTCHA session token site key

1. Navigate to Navigation menu (Navigation menu icon) > Compute Engine > VM Instances. Locate the VM in your instance group and SSH to it.

2. Go to the webserver root directory and change user to root:
```
cd /var/www/html/
sudo su
```
3. Update the landing `index.html` page and embed the reCAPTCHA session token site key. The session token site key (that you recorded earlier) is set in the head section of your landing page as below:
```
src="https://www.google.com/recaptcha/enterprise.js?render=<SESSION_TOKEN_SITE_KEY>&waf=session" async defer>
```
Remember to replace `<SESSION_TOKEN_SITE_KEY>` with the site token before you run the following command:
```bash
echo '<!doctype html><html><head><title>ReCAPTCHA Session Token</title><script src="https://www.google.com/recaptcha/enterprise.js?render=<SESSION_TOKEN_SITE_KEY>&waf=session" async defer></script></head><body><h1>Main Page</h1><p><a href="/good-score.html">Visit allowed link</a></p><p><a href="/bad-score.html">Visit blocked link</a></p><p><a href="/median-score.html">Visit redirect link</a></p></body></html>' > index.html
```

4. Create three other sample pages to test out the bot management policies:

- good-score.html
```
echo '<!DOCTYPE html><html><head><meta http-equiv="Content-Type" content="text/html; charset=windows-1252"></head><body><h1>Congrats! You have a good score!!</h1></body></html>' > good-score.html
```

- bad-score.html
```
echo '<!DOCTYPE html><html><head><meta http-equiv="Content-Type" content="text/html; charset=windows-1252"></head><body><h1>Sorry, You have a bad score!</h1></body></html>' > bad-score.html
```

- median-score.html
```
echo '<!DOCTYPE html><html><head><meta http-equiv="Content-Type" content="text/html; charset=windows-1252"></head><body><h1>You have a median score that we need a second verification.</h1></body></html>' > median-score.html
```

Validate that you are able to access all the webpages by opening them in your browser. Be sure to replace [LB_IP_v4] with the IPv4 address of the load balancer:

1. Open http://[LB_IP_v4]/index.html. You verify that the reCAPTCHA implementation is working when you see "protected by reCAPTCHA" at the bottom right corner of the page:

2. Click into each of the links.
   
3. Validate you are able to access all the pages.

### Task 5. Create Cloud Armor security policy rules for Bot Management

1. In Cloud Shell, create security policy via gcloud:
```
gcloud compute security-policies create recaptcha-policy \
    --description "policy for bot management"
```

2. To use reCAPTCHA Enterprise manual challenge to distinguish between human and automated clients, associate the reCAPTCHA WAF challenge site key (`CHALLENGE-PAGE-KEY`) you previously created for a manual challenge with the security policy. In the following script, remember to replace `"CHALLENGE-PAGE-KEY"` with the key you previously created:
```
gcloud compute security-policies update recaptcha-policy \
  --recaptcha-redirect-site-key "CHALLENGE-PAGE-KEY"
```

3. Add a bot management rule to allow traffic if the url path matches good-score.html and has a score greater than 0.4:
```
gcloud compute security-policies rules create 2000 \
    --security-policy recaptcha-policy\
    --expression "request.path.matches('good-score.html') &&    token.recaptcha_session.score > 0.4"\
    --action allow
```

4. Add a bot management rule to deny traffic if the url path matches bad-score.html and has a score less than 0.6:
```
gcloud compute security-policies rules create 3000 \
    --security-policy recaptcha-policy\
    --expression "request.path.matches('bad-score.html') && token.recaptcha_session.score < 0.6"\
    --action "deny-403"
```

5. Add a bot management rule to redirect traffic to Google reCAPTCHA if the url path matches median-score.html and has a score equal to 0.5:
```
gcloud compute security-policies rules create 1000 \
    --security-policy recaptcha-policy\
    --expression "request.path.matches('median-score.html') && token.recaptcha_session.score == 0.5"\
    --action redirect \
    --redirect-type google-recaptcha
```

6. Attach the security policy to the backend service `http-backend`:
```
gcloud compute backend-services update http-backend \
    --security-policy recaptcha-policy --global
```

7. In the console, navigate to Navigation menu > Network Security > Cloud Armor policies.

8. Click `recaptcha-policy`.

### Task 6. Validate Bot Management with Cloud Armor

Follow the instructions from the lab.

### Congratulations!




