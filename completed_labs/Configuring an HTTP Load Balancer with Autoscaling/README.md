## Translation from console to cloud

To use the gcloud command from your computer, you must first of all be authenticated using:

```bash
gcloud auth login
```

The command above redirects you to your browser where you complete the login.

If using **cloud shell**, the above is unnecessary.

### Configure a health check firewall rule

```bash
gcloud compute --project=[PROJECT] firewall-rules create fw-allow-health-checks --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:80 --source-ranges=130.211.0.0/22,35.191.0.0/16 --target-tags=allow-health-checks
```

### Create a NAT configuration using Cloud Router

```bash
gcloud compute routers create [NAME]
```

```bash
gcloud compute routers nats create [NAME] --router=[ROUTER] --region=[REGION] --nat-external-ip-pool=[IP_ADDRESS]
```

### Create a custom image for a web server

```bash
gcloud beta compute --project=[PROJECT] instances create webserver --zone=us-central1-a --machine-type=f1-micro --subnet=default --no-address --maintenance-policy=MIGRATE --service-account=644407591100-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --tags=allow-health-checks --image=debian-9-stretch-v20200902 --image-project=debian-cloud --boot-disk-size=10GB --no-boot-disk-auto-delete --boot-disk-type=pd-standard --boot-disk-device-name=webserver --reservation-affinity=any
```

**Customize the VM**

```bash
sudo apt-get update
sudo apt-get install -y apache2
## start server
sudo service apache2 start
## test service
curl localhost
## set apache to start with instance
sudo update-rc.d apache2 enable
```

**Create custom image**

```bash
gcloud compute images create mywebserver --project=[PROJECT] --source-disk=webserver --source-disk-zone=us-central1-a --storage-location=us
```

### Configure an instance template and create instance groups

**Configure instance templates**

```bash
gcloud beta compute --project=[PROJECT] instance-templates create mywebserver-template --machine-type=f1-micro --network=projects/qwiklabs-gcp-07fefb4555ae827d/global/networks/default --no-address --maintenance-policy=MIGRATE --service-account=644407591100-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --tags=allow-health-checks --image=mywebserver --image-project=qwiklabs-gcp-07fefb4555ae827d --boot-disk-size=10GB --boot-disk-type=pd-standard --boot-disk-device-name=mywebserver-template --reservation-affinity=any
```

**Create the managed instance groups**
```bash
gcloud compute --project=[PROJECT] health-checks create tcp "http-health-check" --timeout "5" --check-interval "10" --unhealthy-threshold "3" --healthy-threshold "2" --port "80"

gcloud beta compute --project=[PROJECT] instance-groups managed create us-central1-mig --base-instance-name=us-central1-mig --template=mywebserver-template --size=1 --zones=us-central1-b,us-central1-c,us-central1-f --instance-redistribution-type=PROACTIVE --health-check=http-health-check --initial-delay=60

gcloud beta compute --project [PROJECT] instance-groups managed set-autoscaling "us-central1-mig" --region "us-central1" --cool-down-period "60" --max-num-replicas "2" --min-num-replicas "1" --target-load-balancing-utilization "0.8" --mode "on"
```

### Configure the HTTP load balancer

```bash
gcloud compute backend-services create [BACKEND_SERVICE_NAME] \
--enable-logging \
--logging-sample-rate=1 \
--connection-draining-timeout=300 \
--connection-drain-on-failover \
--health-checks=http-health-check

gcloud compute backend-services add-backend [BACKEND_SERVICE_NAME] \
--instance-group=us-central1-mig \
--health-checks=http-health-check \
--balancing-mode=RATE \
--max-rate=50 \
--capacity-scaler=1.0
```

For stress testing, create an image closer to any of the managed instance group and run the Apache http server benchmarking tool