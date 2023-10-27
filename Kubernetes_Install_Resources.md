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

[Youtube how to deploy NFS-Storage guide](https://www.youtube.com/watch?v=efa8gwmbPms)<br>
[Written version of that Youtube video](https://askcloudarchitech.com/posts/tutorials/add-nfs-shared-storage-microk8s-cluster/)<br>
[Kubernetes NFS-Container Storage Interface (CSI) GitHub](https://github.com/kubernetes-csi/csi-driver-nfs)<br>
[MicroK8S Install guide](https://microk8s.io/docs/nfs)<br>
How to setup NFS-Storage for Kubernetes, with needed GitHub.

[How to change the default port range in nodePort, using Calico](https://www.thinkcode.se/blog/2019/02/20/kubernetes-service-node-port-range)
Handy for custom port ranges if needed and only known ports are used.

[How the Kubernetes Scheduler works](https://www.youtube.com/watch?v=rDCWxkvPlAw)<br>
[Kubernetes Descheduler](https://github.com/kubernetes-sigs/descheduler)<br>
[Small Kubernetes Descheduler Explanation](https://www.sobyte.net/post/2023-04/k8s-descheduler/)<br>

Handy commands for loading docker images into containerd runtime:
```
sudo ctr --namespace=k8s.io images import /path/on/remote/node/image.tar
```
And to see all images on the host:
```
sudo ctr --namespace=k8s.io images list
```