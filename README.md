# Kubernetes setup on Amazon AWS using Kops and Ansible

This repository contains tooling for deploying Kubernetes cluster in Amazon AWS using the [Kops](https://github.com/kubernetes/kops) tool. Kops is a great tool if you want to setup HA cluster and don't require too much flexibility. If you prefer flexibility instead of HA setup you should have a look at [another repsoitory](https://github.com/scholzj/aws-kubernetes) where I have Kubernetes setup implemented using Terraform and Kubeadm tool. I have also a [special *minikube* single node installation](https://github.com/scholzj/aws-minikube).

<!-- TOC depthFrom:2 -->

- [Updates](#updates)
- [Installing the cluster](#installing-the-cluster)
    - [Install Ansible](#install-ansible)
    - [Kubectl installation](#kubectl-installation)
    - [Kops installation](#kops-installation)
    - [AWS Credentials](#aws-credentials)
    - [S3 bucket for state store](#s3-bucket-for-state-store)
    - [Install Kubernetes cluster](#install-kubernetes-cluster)
    - [Install add-ons (optional)](#install-add-ons-optional)
    - [Install ingress (optional)](#install-ingress-optional)
    - [Install the tagging lambda function (optional)](#install-the-tagging-lambda-function-optional)
- [Updating the cluster](#updating-the-cluster)
- [Deleting the cluster](#deleting-the-cluster)
    - [Deleting the tagging lambda function](#deleting-the-tagging-lambda-function)
- [Frequently Asked Questions](#frequently-asked-questions)
    - [How to access Kuberntes Dashboard](#how-to-access-kuberntes-dashboard)

<!-- /TOC -->

## Installing the cluster

The cluster can be deployed from your local host by following the steps described below. If you cannot install Ansible, kubectl or kops on your local PC or in case your local PC is running Windows, you can create a EC2 host in Aamzon AWS and run the installation from this host.

### Install Ansible

Download and install Ansible - you can follow the guide from [Ansible website](http://docs.ansible.com/ansible/latest/intro_installation.html).

### Kubectl installation

Install the latest version of `kubectl` on Linux or MacOS:
```
ansible-playbook install-kubectl.yaml
```
You may need either `--ask-sudo-pass` or `ansible_become_pass`


### Kops installation

Install the latest version of `Kops` utility on Linux or MacOS:
```
ansible-playbook install-kops.yaml
```
You may need either `--ask-sudo-pass` or `ansible_become_pass`

### AWS Credentials

Export the AWS credentials whih will be used to authenticate with Amazon AWS:
```
export AWS_ACCESS_KEY_ID="XXX"
export AWS_SECRET_ACCESS_KEY="XXX"
```

### S3 bucket for state store

Create S3 bucket to store where `Kops` will store its information:
```
ansible-playbook create-state-store.yaml
```

**The bucket will contain also the access details for the clusters configured with Kops. It should be secured accordingly.**

### Install Kubernetes cluster

Create the Kubernetes cluster using `Kops`:
```
ansible-playbook create.yaml
```

The main configuration of the cluster is in the variables in `group_vars/all/vars.yaml`. Following table shows the different options.

| Option | Explanation | Example |
|--------|-------------|---------|
| `cluster_name` | Name of the cluster which should be created. The name has to end with the domain name of the DNS zone hosted in Route 53. | `kubernetes.my-cluster.com` |
| `state_store` | Name of the Amazon S3 bucket which should be used as a Kops state store. It should start with `s3://`. | `s3://kops-state-store` |
| `ssh_public_key` | Path to the public part of the SSH key, which should be used for the SSH access to the Kubernetes hosts | `~/.ssh/id_rsa.pub` |
| `aws_region` | AWS region where the cluster should be installed. | `eu-west-1` |
| `aws_zones` | List of availability zones in which the cluster should be installed. Must be an odd number (1, 3 or 5) of zones (at least 3 zones are needed for AWS setup accross availability zones). | `eu-west-1a,eu-west-1b,eu-west-1c` |
| `master_zones` | List of availability zones in which the master nodes should be installed. Must be an odd number (1, 3 or 5) of zones (at least 3 zones are needed for AWS setup accross availability zones). If not specified, `aws_zones` will be used instead | `eu-west-1a,eu-west-1b,eu-west-1c` |
| `dns_zone` | Name of the Rote 53 hosted zone. Must be reflected in the cluster name. | `my-cluster.com` |
| `network_cidr` | A new VPC will be created. It will use the CIDR specified in this option. | `172.16.0.0/16` |
| `topology` | Defines whether the cluster should be deployed into private subnet (`private` - more secure) with bastion host or into public subnet (`public` - less secure). | `private` |
| `kubernetes_networking` | Defines which networking plugin should be used in Kubernetes. Tested with Calico only. | `calico` |
| `master_size` | EC2 size of the nodes used for the Kubernetes masters (and Etcd hosts) | `m4.large` |
| `master_count` | Number of EC2 master hosts. | `3` |
| `master_volume_size` | Size of the master disk volume in GB. | `50` |
| `node_size` | EC2 size of the nodes used as workers. | `m4.large` |
| `node_count` | Number of EC2 worker hosts (initial count). | `6` |
| `node_volume_size` | Size of the node disk volume in GB. | `50` |
| `node_autoscaler_min` | Minimum number of nodes (for the autoscaler). | `3` |
| `node_autoscaler_max` | Maximum number of nodes (for the autoscaler). | `6` |
`

### Install ingress

Ingress can be used route inbound traffic from the outside of the Kubernetes cluster. It can be used for SSL termination, virtual hosts, load balancing etc. For more details about ingress, go to [Kubernetes website](https://kubernetes.io/docs/concepts/services-networking/ingress/).

To install ingress controller based on Nginx, run
```
ansible-playbook ingress.yaml
```

## Updating the cluster

All updates to the running Kubernetes cluster can be done directly using `Kops`. The Ansible playbooks from this project only simplify the initial setup.

## Deleting the cluster

To delete the cluster export the AWS credentials:
```
export AWS_ACCESS_KEY_ID="XXX"
export AWS_SECRET_ACCESS_KEY="XXX"
```

And run:
```
ansible-playbook delete.yaml
```
