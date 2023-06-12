https://github.com/kubernetes/k8s.io/pull/4837

https://github.com/kubernetes/release/issues/2862

https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/

##### How To Handle apt-key and add-apt-repository Deprecation Using gpg to Add External Repositories on Ubuntu 22.04

sudo apt-key list

On Ubuntu, the /usr/share/keyrings directory is the recommended location for your converted GPG files, as it is the default location where Ubuntu stores its keyrings.

##### For example:Temporarily host apt-key.gpg as original url is failing

--dearmor: Pack or unpack an arbitrary input into/from an OpenPGP ASCII armo

NOTE: you can use gpg --dearmor -o apt-key.gpg apt-key.asc to generate the binary version from the ascii version. checked in both for easier inspection

Folks please use: https://dl.k8s.io/apt/doc/apt-key.gpg instead of the google apt-key gpg keys url https://packages.cloud.google.com/apt/doc/apt-key.gpg !

###### Download the Google Cloud public signing key:
sudo curl -fsSLo /etc/apt/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg

##### Kubernetes - https://github.com/kubernetes/release/issues/2862
They have updated their host address, so now we should update it to use the key from https://dl.k8s.io/apt/doc/apt-key.gpg. Then use something like:

sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://dl.k8s.io/apt/doc/apt-key.gpg
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update -y

##### Another option is:
sudo apt-key list
mkdir -p /etc/apt/keyrings
curl -fsSL "https://packages.cloud.google.com/apt/doc/apt-key.gpg" | gpg --dearmor -o /etc/apt/trusted.gpg.d/kubernetes-archive-keyring.gpg

use this below for command to make corrections : sudo gpg --dearmor -o
###### kubernetes apt-key gpg errors.
curl -fsSLo /etc/apt/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg

curl -fsSL https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-archive-keyring.gpg