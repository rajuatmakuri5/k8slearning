# Kubernetes Cheat Sheet & Exercises

This repository contains a collection of **Kubernetes commands** and exercises to help you get hands-on experience with Kubernetes. The exercises cover basic operations, from creating a pod to deploying applications and services in Kubernetes.

## Common Commands

Here are some of the most commonly used `kubectl` commands for managing Kubernetes resources:

```bash
alias k=kubectl

# View the status of pods, nodes, and deployments
k get pods
k get nodes
k get pods -o wide
k get nodes -o wide
k get pods --show-labels
k get all
k get pods -n kube-system
k get deployments
k get svc
k get pvc
k get rs
k get secrets
k get endpoints

# Describe resources for detailed information
k describe pod <podname>
k explain pod.spec.name   # or `k explain pod`
k explain service
k explain deployment

```
