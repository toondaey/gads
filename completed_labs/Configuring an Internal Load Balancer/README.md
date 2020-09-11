## Translation from console to cloud

To use the gcloud command from your computer, you must first of all be authenticated using:

```bash
gcloud auth login
```

The command above redirects you to your browser where you complete the login.

If using **cloud shell**, the above is unnecessary.

### Configure internal traffic and health check firewall rules.

**Create the firewall rule to allow traffic from any sources in the 10.10.0.0/16 range**

```bash
gcloud compute --project=[PROJECT] firewall-rules create fw-allow-lb-access --direction=INGRESS --priority=1000 --network=my-internal-app --action=ALLOW --rules=all --source-ranges=10.10.0.0/16 --target-tags=backend-service
```

**Create the health check rule**

```bash
gcloud compute --project=[PROJECT] firewall-rules create fw-allow-health-checks --direction=INGRESS --priority=1000 --network=my-internal-app --action=ALLOW --rules=tcp:80 --source-ranges=130.211.0.0/22,35.191.0.0/16 --target-tags=backend-service
```

### Create a NAT configuration using Cloud Router

**Create Router**

```bash
gcloud compute routers nats create [NAME] --router=[ROUTER] --region=[REGION] --nat-external-ip-pool=[IP_ADDRESS]
```

**Create Cloud router instance**

```bash
gcloud compute routers nats create [NAME] --router=[ROUTER] --region=[REGION] --nat-external-ip-pool=[IP_ADDRESS]
```

### Configure instance templates and create instance groups

**Configure the instance templates**

```bash
gcloud beta compute --project=[PROJECT] instance-templates create instance-template-1 --machine-type=e2-medium --subnet=projects/qwiklabs-gcp-03-7f5b8d2803a7/regions/us-central1/subnetworks/subnet-a --no-address --metadata=startup-script-url=gs://cloud-training/gcpnet/ilb/startup.sh --maintenance-policy=MIGRATE --service-account=569919412468-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --region=us-central1 --tags=backend-service --image=debian-10-buster-v20200902 --image-project=debian-cloud --boot-disk-size=10GB --boot-disk-type=pd-standard --boot-disk-device-name=instance-template-1 --no-shielded-secure-boot --no-shielded-vtpm --no-shielded-integrity-monitoring --reservation-affinity=any

gcloud beta compute --project=[PROJECT] instance-templates create instance-template-2 --machine-type=e2-medium --subnet=projects/qwiklabs-gcp-03-7f5b8d2803a7/regions/us-central1/subnetworks/subnet-b --no-address --metadata=startup-script-url=gs://cloud-training/gcpnet/ilb/startup.sh --maintenance-policy=MIGRATE --service-account=569919412468-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --region=us-central1 --tags=backend-service --image=debian-10-buster-v20200902 --image-project=debian-cloud --boot-disk-size=10GB --boot-disk-type=pd-standard --boot-disk-device-name=instance-template-2 --no-shielded-secure-boot --no-shielded-vtpm --no-shielded-integrity-monitoring --reservation-affinity=any
```

**Create the managed instance groups**

Instance group 1

```bash
gcloud compute --project=[PROJECT] instance-groups managed create instance-group-1 --base-instance-name=instance-group-1 --template=instance-template-1 --size=1 --zone=us-central1-a

gcloud beta compute --project [PROJECT] instance-groups managed set-autoscaling "instance-group-1" --zone "us-central1-a" --cool-down-period "45" --max-num-replicas "5" --min-num-replicas "1" --target-cpu-utilization "0.8" --mode "on"
```

Instance group 2

```bash
gcloud compute --project=[PROJECT] instance-groups managed create instance-group-2 --base-instance-name=instance-group-2 --template=instance-template-2 --size=1 --zone=us-central1-b

gcloud beta compute --project [PROJECT] instance-groups managed set-autoscaling "instance-group-2" --zone "us-central1-b" --cool-down-period "45" --max-num-replicas "5" --min-num-replicas "1" --target-cpu-utilization "0.8" --mode "on"
```

**Verify backend service**

Create Utility VM

```bash
gcloud beta compute --project=[PROJECT] instances create utility-vm --zone=us-central1-f --machine-type=f1-micro --subnet=subnet-a --private-network-ip=10.10.20.50 --no-address --maintenance-policy=MIGRATE --service-account=569919412468-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --image=debian-10-buster-v20200902 --image-project=debian-cloud --boot-disk-size=10GB --boot-disk-type=pd-standard --boot-disk-device-name=utility-vm --no-shielded-secure-boot --no-shielded-vtpm --no-shielded-integrity-monitoring --reservation-affinity=any
```

### Configure the internal load balancer

Create health check
### Configure a health check firewall rule

```bash
gcloud compute --project=[PROJECT] firewall-rules create my-ilb-health-check --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:80
```

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