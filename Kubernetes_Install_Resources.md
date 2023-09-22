# Kubernetes Installation Resources:

Author: **Daan Selen**.<br>
For the left-out sources, refer to the **end of the page**.

This page is made to place notable sources for learning Kubernetes.

## **Resources with description:**

[How to install and initialise a Kubernetes cluster on Debian 11/12](https://www.linuxtechi.com/install-kubernetes-cluster-on-debian/)<br>
This is handy because this tutorial gives a step-by-step explanation on how to deploy a Kubernetes cluster with a working end result.

[How to install Metrics Server onto a running Kubernetes cluster](https://www.linuxtechi.com/how-to-install-kubernetes-metrics-server/)<br>
Metrics Server can be used to monitor a kubernetes cluser its nodes and pods.

[Calico Documentation](https://docs.tigera.io/calico/latest/about/)<br>
What is Calico and how does it work.

Handy commands for loading docker images into containerd runtime:
```
sudo ctr --namespace=k8s.io images import /path/on/remote/node/image.tar
```
And to see all images on the host:
```
sudo ctr --namespace=k8s.io images list
```