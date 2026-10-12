# Build and Secure Networks in Google Cloud: Challenge Lab (GSP322)

![Google Cloud](https://img.shields.io/badge/Google_Cloud-4285F4?logo=googlecloud&logoColor=white)
![Lab](https://img.shields.io/badge/Lab-GSP322-34A853)
![Topic](https://img.shields.io/badge/Topic-Network_Security-red)

Reference solution for the Google Cloud Skills Boost challenge lab **Build and Secure Networks in Google Cloud** (GSP322).

## 📖 Overview

The lab gives you a VPC with an overly permissive firewall. You lock it down so that:

- Only the **bastion** host accepts SSH from outside, and only through **Identity-Aware Proxy (IAP)**.
- The **juice-shop** web server accepts HTTP from anywhere.
- The **juice-shop** server accepts SSH only from the management subnet.

## 🧰 Services used

VPC networks · Firewall rules with network tags · Compute Engine · Identity-Aware Proxy (IAP) TCP forwarding

## ✅ Prerequisites

Open **Cloud Shell** and set the zone and network from your lab:

```bash
export ZONE="us-central1-b"
export NETWORK="acme-vpc"
export MGMT_SUBNET_RANGE="192.168.10.0/24"
```

## 🚀 Solution

### Task 1: Remove the overly permissive rule

```bash
gcloud compute firewall-rules list --filter="network:$NETWORK"
gcloud compute firewall-rules delete open-access --quiet
```

### Task 2: Start the bastion host

```bash
gcloud compute instances start bastion --zone $ZONE
```

### Task 3: Allow SSH to the bastion from IAP only

`35.235.240.0/20` is the source range Google uses for IAP TCP forwarding.

```bash
gcloud compute firewall-rules create ssh-ingress \
  --network $NETWORK \
  --allow tcp:22 \
  --source-ranges 35.235.240.0/20 \
  --target-tags ssh-ingress

gcloud compute instances add-tags bastion --tags ssh-ingress --zone $ZONE
```

### Task 4: Allow HTTP to juice-shop

```bash
gcloud compute firewall-rules create http-ingress \
  --network $NETWORK \
  --allow tcp:80 \
  --source-ranges 0.0.0.0/0 \
  --target-tags http-ingress

gcloud compute instances add-tags juice-shop --tags http-ingress --zone $ZONE
```

### Task 5: Allow SSH to juice-shop from the management subnet

```bash
gcloud compute firewall-rules create internal-ssh-ingress \
  --network $NETWORK \
  --allow tcp:22 \
  --source-ranges $MGMT_SUBNET_RANGE \
  --target-tags internal-ssh-ingress

gcloud compute instances add-tags juice-shop --tags internal-ssh-ingress --zone $ZONE
```

### Task 6: SSH to juice-shop through the bastion

```bash
gcloud compute ssh bastion --zone $ZONE --tunnel-through-iap

# From inside the bastion:
ssh <juice-shop-internal-ip>
```

## 🔐 Resulting access model

| Rule | Port | Source | Target tag |
|---|---|---|---|
| `ssh-ingress` | 22 | IAP range `35.235.240.0/20` | bastion |
| `http-ingress` | 80 | `0.0.0.0/0` | juice-shop |
| `internal-ssh-ingress` | 22 | management subnet | juice-shop |

---

## ⚠️ Notes

- Resource names, zones, regions and ports are generated per lab session. Always use the values shown in **your** lab instructions.
- These steps were written when the lab was live. Google revises labs over time, so the current version may differ.

## 📌 Disclaimer

This repository is a study reference. Attempt the lab on your own first, then compare approaches. It is not affiliated with or endorsed by Google.

## 👤 Author

**Girish Sharma** · Cloud & DevOps Engineer · [GitHub](https://github.com/codewithgarry) · [LinkedIn](https://www.linkedin.com/in/codewithgarry/)
