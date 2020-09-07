## Translation from console to cloud

To use the gcloud command from your computer, you must first of all be authenticated using:

```bash
gcloud auth login
```

The command above redirects you to your browser where you complete the login.

### Create the VPN gateways and tunnels

**Reserve two static IP addresses**

```bash
gcloud compute addresses create vpn-1-static-ip --project=[PROJECT] --region=us-central1
gcloud compute addresses create vpn-1-static-ip --project=[PROJECT] --region=europe-west1
```

**Create the vpn-1 gateway and tunnel1to2**

```bash
gcloud compute --project "qwiklabs-gcp-03-5165366c2925" target-vpn-gateways create "vpn-1" --region "us-central1" --network "vpn-network-1"

gcloud compute --project "qwiklabs-gcp-03-5165366c2925" forwarding-rules create "vpn-1-rule-esp" --region "us-central1" --address "34.121.163.93" --ip-protocol "ESP" --target-vpn-gateway "vpn-1"

gcloud compute --project "qwiklabs-gcp-03-5165366c2925" forwarding-rules create "vpn-1-rule-udp500" --region "us-central1" --address "34.121.163.93" --ip-protocol "UDP" --ports "500" --target-vpn-gateway "vpn-1"

gcloud compute --project "qwiklabs-gcp-03-5165366c2925" forwarding-rules create "vpn-1-rule-udp4500" --region "us-central1" --address "34.121.163.93" --ip-protocol "UDP" --ports "4500" --target-vpn-gateway "vpn-1"

gcloud compute --project "qwiklabs-gcp-03-5165366c2925" vpn-tunnels create "tunnel1to2" --region "us-central1" --peer-address "34.78.91.125" --shared-secret "gcprocks" --ike-version "2" --local-traffic-selector "0.0.0.0/0" --target-vpn-gateway "vpn-1"

gcloud compute --project "qwiklabs-gcp-03-5165366c2925" routes create "tunnel1to2-route-1" --network "vpn-network-1" --next-hop-vpn-tunnel "tunnel1to2" --next-hop-vpn-tunnel-region "us-central1" --destination-range "10.1.3.0/24"
```

**Create the vpn-2 gateway and tunnel2to1**

```bash
gcloud compute --project "qwiklabs-gcp-03-5165366c2925" target-vpn-gateways create "vpn-2" --region "europe-west1" --network "vpn-network-2"

gcloud compute --project "qwiklabs-gcp-03-5165366c2925" forwarding-rules create "vpn-2-rule-esp" --region "europe-west1" --address "34.78.91.125" --ip-protocol "ESP" --target-vpn-gateway "vpn-2"

gcloud compute --project "qwiklabs-gcp-03-5165366c2925" forwarding-rules create "vpn-2-rule-udp500" --region "europe-west1" --address "34.78.91.125" --ip-protocol "UDP" --ports "500" --target-vpn-gateway "vpn-2"

gcloud compute --project "qwiklabs-gcp-03-5165366c2925" forwarding-rules create "vpn-2-rule-udp4500" --region "europe-west1" --address "34.78.91.125" --ip-protocol "UDP" --ports "4500" --target-vpn-gateway "vpn-2"

gcloud compute --project "qwiklabs-gcp-03-5165366c2925" vpn-tunnels create "tunnel2to1" --region "europe-west1" --peer-address "34.121.163.93" --shared-secret "gcprocks" --ike-version "2" --local-traffic-selector "0.0.0.0/0" --target-vpn-gateway "vpn-2"

gcloud compute --project "qwiklabs-gcp-03-5165366c2925" routes create "tunnel2to1-route-1" --network "vpn-network-2" --next-hop-vpn-tunnel "tunnel2to1" --next-hop-vpn-tunnel-region "europe-west1" --destination-range "10.5.4.0/24"
```

We can then test and see that VMs from both regions can ping (contact) each other using their respective internal IP addresses.