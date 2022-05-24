# Openshift Install
VERSION:v3.11

##  Step 0: images required
```bash
[root@centos8 ~]# docker images
REPOSITORY                       TAG       IMAGE ID       CREATED        SIZE
openshift/origin-control-plane   v3.11     7a2f8e6e6a15   2 weeks ago    839MB
openshift/origin-cli             v3.11     4a9bc42a342b   2 weeks ago    390MB
openshift/origin-node            v3.11     3f7a62b09e77   6 months ago   1.2GB
openshift/origin-hyperkube       v3.11     0cfb433fadc5   6 months ago   515MB
openshift/origin-hypershift      v3.11     9e866d795822   6 months ago   556MB
```
## Step 1: Install openshift
```bash
    ./install.bash
```