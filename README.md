# MicroK8s Local Deployment Guide & Manifests

A companion repository for setting up a local MicroK8s cluster on Debian-based systems, specifically tailored to handle environments with non-standard user home directories. 

This setup is particularly useful if you are developing on a corporate Linux workstation (such as those at Google) where enterprise configurations often mount user directories in custom paths like `/usr/local/google/home/`, causing standard Snap installations to fail.

---

## Repository Contents

This repository contains the declarative configuration files needed to spin up a local load balancer and deploy a sample web application. 

| File | Purpose | Is Required? |
|---|---|---|
| **`metallb-pool.yaml`** | Defines the local IP address range for MetalLB to assign to services. | Yes |
| **`namespace.yaml`** | Creates a dedicated `development` namespace to isolate these resources. | Recommended |
| **`deployment.yaml`** | Deploys a sample Nginx web server across two replica pods. | Yes |
| **`service.yaml`** | Exposes the Nginx pods to your local network using a LoadBalancer. | Yes |

---

## Prerequisites

Before applying these manifests, ensure your local environment is configured:

- **Debian-based OS** with `snapd` installed.
- **MicroK8s** installed via classic confinement.
- **Custom Home Directory Mapped** (if applicable) using `sudo snap set system homedirs=...`.
- **MetalLB Addon Enabled** via `microk8s enable metallb`.

> **Pro-Tip:** Make sure the IP range you configure in `metallb-pool.yaml` sits **outside** of your local router's DHCP pool to prevent IP address conflicts on your network.

---

## Quick Start Guide

Follow these steps to deploy the application to your local cluster.

### 1. Configure the Load Balancer

First, apply the MetalLB address pool. This tells your cluster which local IP addresses it is allowed to hand out.

```bash
kubectl apply -f metallb-pool.yaml
```

### 2. Set Up the Namespace

Create the isolated environment for the application.

```bash
kubectl create namespace development
# Alternatively, if you have a namespace.yaml:
# kubectl apply -f namespace.yaml
```

### 3. Deploy the Application

Roll out the Nginx web server pods and the service that routes traffic to them.

```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

---

## Verifying the Deployment

Once the manifests are applied, you can verify that MetalLB successfully assigned an external IP address to your service.

```bash
kubectl get services -n development
```

You should see an output similar to this, where `EXTERNAL-IP` is populated from your configured pool:

```text
NAME             TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)        AGE
mysite-service   LoadBalancer   10.152.183.42   172.28.62.192   80:32145/TCP   2m
```

Open a web browser and navigate to that `EXTERNAL-IP` (e.g., `[http://172.28.62.192](http://172.28.62.192)`). You should be greeted by your running web application!

---

## Teardown and Cleanup

If you are just experimenting and want to free up your system resources and local IP addresses, you can easily delete the applied configurations.

```bash
kubectl delete -f service.yaml
kubectl delete -f deployment.yaml
kubectl delete namespace development
kubectl delete -f metallb-pool.yaml
```