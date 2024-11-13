# Excerises: Verify the behavior of ephemeral volumes (emptyDir) on a single node cluster

## Main Points

1. The actual location of an Ephemeral volume (emptyDir) is a folder of the node where the current pod is running.
2. If the pod is deleted, the corresponding folder (actual storage location) will be deleted.

```text
# single node cluster created through minikube

NAME       STATUS   ROLES           AGE   VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION                       CONTAINER-RUNTIME
minikube   Ready    control-plane   20d   v1.31.0   192.168.49.2   <none>        Ubuntu 22.04.4 LTS   5.15.153.1-microsoft-standard-WSL2   docker://27.2.0
```

### deploy pod with ephemeral volume

```bash

# deploy pod with ephemeral volume
k apply -f pod-ephemeral-volume.yaml

```

### check the actual location of the emptyDir (Ephemeral Volume)

```bash

# find the uid of the pod - examine-ephemeral-volume-pod

k get pod examine-ephemeral-volume-pod -o=jsonpath='{.metadata.uid}' # for my environment, the uid is f029a915-33a2-454a-b697-f611248ab24f, it may be different for your case.

# login the single node. Since it's a signle node cluster, the actual storage location must be located in this node.
minikube ssh


# navigate to the following directory
sudo -i  # switch to root user, otherwise there is no permission to view the following folder.
cd /var/lib/kubelet/pods/f029a915-33a2-454a-b697-f611248ab24f/volumes/kubernetes.io~empty-dir/tmp-volume

# create a txt file within this folder
echo "hello emptyDir">hello.txt

# open a new terminal and run the following command
k exec -it examine-ephemeral-volume-pod -- bash
cd /tmp
ls # you will see hello.txt is there
cat hello.txt

# create a new file within this container
touch world.txt

# go back to the pervious terminal
ls /var/lib/kubelet/pods/f029a915-33a2-454a-b697-f611248ab24f/volumes/kubernetes.io~empty-dir/tmp-volume

# world.txt and hello.txt should be there.


# quit this terminal and delete the pod

k delete -f pod-ephemeral-volume.yml

# verify the folder - /var/lib/kubelet/pods/f029a915-33a2-454a-b697-f611248ab24f/volumes/kubernetes.io~empty-dir/tmp-volume is deleted 
minikube ssh
sudo -i
ls /var/lib/kubelet/pods/f029a915-33a2-454a-b697-f611248ab24f/volumes/kubernetes.io~empty-dir/tmp-volume

# we will get the message: 
# ls: cannot access '/var/lib/kubelet/pods/f029a915-33a2-454a-b697-f611248ab24f/volumes/kubernetes.io~empty-dir/tmp-volume': No such file or directory

```
