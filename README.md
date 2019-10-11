# k8s-cluster-node-deploy
A couple of playbooks created for learning purposes and deploy a functional kubernete cluster with an additional node.

Main Playbooks:

ansible-playbook -i <inventory_file> playbooks/k8s/<playbook_name>
ansible-playbook -i cicd_hosts.yaml playbooks/k8s/deploy_all.yaml

- deploy_all.yaml: Deploys a Cluster + Node + Calico Network + Kubernetes Dashboard

- deploy_cluster.yaml: Deploys a cluster only.

- deploy_network.yaml: Deploys the network plugin. It can be configured to flannel instead of calico.

- deploy_node.yaml: Deploys a node.


Other files:

- base.yaml: Base packages, docker, kube* packages and OS configuration required by cluster and nodes.

- install_k8_cluster.yaml: Contains the commands to deploy the cluster

- install_k8_node.yaml: Contains the commands to join the node in the cluster

- install_network_plugin.yaml: Single task related to plugin installation



Notice:
- With "base.yaml", there first task there is 2 variables ued in file 'install_k8_cluster.yaml'.

-- set_fact: apiserver_advertise_address="10.0.0.98"

-- set_fact: pod_network_cidr="192.168.0.0/16"

- With "deploy_all", there is a shell script once cluster and network plugin are deploys, this shell script wait until all PODs are with 'Running' status before moving forward and join the node. 


# Hosts inventory

```
[cluster]
kclustert.home.net

[nodes]
knode2.home.net
```

# Install Cluster+Node

# Install Cluster

# Install Network

# Install Node
