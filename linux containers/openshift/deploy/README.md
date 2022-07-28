# Openshift Install
VERSION:v3.11
require:docker-ce

##  Step 0: prepare images required
see imagelist.txt

## Step 1: Install openshift
```bash
    ./install.bash
```

## install imagebuilder
 go get -u github.com/openshift/imagebuilder/cmd/imagebuilder
