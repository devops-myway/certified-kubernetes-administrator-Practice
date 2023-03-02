##### Install Containerd
containerd is an industry-standard container runtime that manages the complete container lifecycle of its host system. It handles image transfer and storage, container execution and supervision, low-level storage and network attachments, etc


install containerd using the apt command.
``````sh

sudo apt update
sudo apt install -y containerd
sudo systemctl status containerd

``````

##### Containerd configuration for Kubernetes
containerd uses a configuration file config.toml for managing demons.
For Kubernetes, you need to configure the Systemd group driver for runC.

``````sh

cat <<EOF | sudo tee -a /etc/containerd/config.toml

[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
SystemdCgroup = true
EOF

sudo systemctl restart containerd

``````