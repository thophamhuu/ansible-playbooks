# Ansible Playbooks
Collection used by AWX (Kafka-driven) to configure K8s nodes.

## Playbooks
- **k8s-node.yml** – install kubelet, kubeadm, kubectl on Ubuntu 22.04
- Requirements: Ubuntu cloud-init, public key của AWX trong `authorized_keys`