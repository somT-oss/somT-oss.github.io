---
title: "K8s Auto-Scaler"
weight: 1
tech: ["Go", "Kubernetes", "Prometheus"]
github: "https://github.com/yourusername/k8s-autoscaler"
live: ""
summary: "A custom Kubernetes controller that scales deployments based on Prometheus metrics beyond what HPA supports out of the box."
---

## Overview

HPA is great until it isn't. This project started when we needed to scale on a custom business metric (queue depth from an internal service) that the default Horizontal Pod Autoscaler doesn't support.

The result is a lightweight controller written in Go that watches Prometheus for any metric you configure and drives scaling decisions based on it.

## How It Works

1. The controller runs as a deployment in-cluster
2. It scrapes a configured Prometheus query on an interval
3. If the metric crosses a threshold, it patches the target deployment's replica count
4. Cooldown windows prevent flapping

## What I Learned

- Writing Kubernetes controllers with `controller-runtime`  
- The surprisingly tricky parts of leader election  
- Why you should always implement cooldowns before you think you need them
