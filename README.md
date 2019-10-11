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
- With "base.yaml", in the first task, there are 2 variables used in file 'install_k8_cluster.yaml'. They will define API Server IP and POD Network CIDR, required by network plugin during kubeadm init.

-- set_fact: apiserver_advertise_address="10.0.0.98"

-- set_fact: pod_network_cidr="192.168.0.0/16"

- With "deploy_all", there is a shell script once cluster and network plugin are deploys, this shell script wait until all PODs are with 'Running' status before moving forward and join the node. 

```
    - name: Awaiting all PODs to start
      #script: sh check_awaiting_solution.sh
      shell:
       cmd: |
        echo "Get total PODs and wait until all of them are in Running state to move forward"
        TOT=`kubectl get pods --all-namespaces|awk '{print $4}'|wc -l`
        COUNT_RUNNING=`kubectl get pods --all-namespaces|awk '{print $4}'|grep Running|wc -l`
        echo "Total PODs: $TOT"
        echo "Total running pods: $COUNT_RUNNING"
        let DIFF="$(($TOT)) - $(($COUNT_RUNNING))"
        sleep 40
        while [  $(($DIFF)) -gt 1 ]; do
          sleep 20
          TOT=`kubectl get pods --all-namespaces|awk '{print $4}'|wc -l`
          COUNT_RUNNING=`kubectl get pods --all-namespaces|awk '{print $4}'|grep Running|wc -l`
          let DIFF="$(($TOT)) - $(($COUNT_RUNNING))"
          echo "Total PODs : $COUNT_RUNNING"
          echo "Pending to initiate: $DIFF"
        done
        echo "All PODs are running"
```


# Hosts inventory

```
[cluster]
kclustert.home.net

[nodes]
knode2.home.net
```

# Install Cluster+Node
```
Daniels-MacBook-Pro:kube daniel$ ansible-playbook -i cicd_hosts.yaml playbooks/k8s/deploy_all.yaml 

PLAY [cluster] *********************************************************

TASK [Gathering Facts] *********************************************************
ok: [kclustert.home.net]

TASK [Define GLOBAL Variables] *********************************************************
changed: [kclustert.home.net]

TASK [set_fact] *********************************************************
ok: [kclustert.home.net]

TASK [set_fact] *********************************************************
ok: [kclustert.home.net]

TASK [Define utf-8 codepage]*********************************************************
changed: [kclustert.home.net]
...
...
TASK [debug] *********************************************************
ok: [kclustert.home.net] => {
    "msg": "[init] Using Kubernetes version: v1.16.1\n[preflight] Running pre-flight checks\n[preflight] Pulling images required for setting up a Kubernetes cluster\n[preflight] This might take a minute or two, depending on the speed of your internet connection\n[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'\n[kubelet-start] Writing kubelet environment file with flags to file \"/var/lib/kubelet/kubeadm-flags.env\"\n[kubelet-start] Writing kubelet configuration to file \"/var/lib/kubelet/config.yaml\"\n[kubelet-start] Activating the kubelet service\n[certs] Using certificateDir folder \"/etc/kubernetes/pki\"\n[certs] Generating \"ca\" certificate and key\n[certs] Generating \"apiserver\" certificate and key\n[certs] apiserver serving cert is signed for DNS names [kclustert.home.net kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 10.0.0.98]\n[certs] Generating \"apiserver-kubelet-client\" certificate and key\n[certs] Generating \"front-proxy-ca\" certificate and key\n[certs] Generating \"front-proxy-client\" certificate and key\n[certs] Generating \"etcd/ca\" certificate and key\n[certs] Generating \"etcd/server\" certificate and key\n[certs] etcd/server serving cert is signed for DNS names [kclustert.home.net localhost] and IPs [10.0.0.98 127.0.0.1 ::1]\n[certs] Generating \"etcd/peer\" certificate and key\n[certs] etcd/peer serving cert is signed for DNS names [kclustert.home.net localhost] and IPs [10.0.0.98 127.0.0.1 ::1]\n[certs] Generating \"etcd/healthcheck-client\" certificate and key\n[certs] Generating \"apiserver-etcd-client\" certificate and key\n[certs] Generating \"sa\" key and public key\n[kubeconfig] Using kubeconfig folder \"/etc/kubernetes\"\n[kubeconfig] Writing \"admin.conf\" kubeconfig file\n[kubeconfig] Writing \"kubelet.conf\" kubeconfig file\n[kubeconfig] Writing \"controller-manager.conf\" kubeconfig file\n[kubeconfig] Writing \"scheduler.conf\" kubeconfig file\n[control-plane] Using manifest folder \"/etc/kubernetes/manifests\"\n[control-plane] Creating static Pod manifest for \"kube-apiserver\"\n[control-plane] Creating static Pod manifest for \"kube-controller-manager\"\n[control-plane] Creating static Pod manifest for \"kube-scheduler\"\n[etcd] Creating static Pod manifest for local etcd in \"/etc/kubernetes/manifests\"\n[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory \"/etc/kubernetes/manifests\". This can take up to 4m0s\n[apiclient] All control plane components are healthy after 34.004607 seconds\n[upload-config] Storing the configuration used in ConfigMap \"kubeadm-config\" in the \"kube-system\" Namespace\n[kubelet] Creating a ConfigMap \"kubelet-config-1.16\" in namespace kube-system with the configuration for the kubelets in the cluster\n[upload-certs] Skipping phase. Please see --upload-certs\n[mark-control-plane] Marking the node kclustert.home.net as control-plane by adding the label \"node-role.kubernetes.io/master=''\"\n[mark-control-plane] Marking the node kclustert.home.net as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule]\n[bootstrap-token] Using token: yer1c6.3g0jps020b58t39q\n[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles\n[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials\n[bootstrap-token] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token\n[bootstrap-token] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster\n[bootstrap-token] Creating the \"cluster-info\" ConfigMap in the \"kube-public\" namespace\n[addons] Applied essential addon: CoreDNS\n[addons] Applied essential addon: kube-proxy\n\nYour Kubernetes control-plane has initialized successfully!\n\nTo start using your cluster, you need to run the following as a regular user:\n\n  mkdir -p $HOME/.kube\n  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config\n  sudo chown $(id -u):$(id -g) $HOME/.kube/config\n\nYou should now deploy a pod network to the cluster.\nRun \"kubectl apply -f [podnetwork].yaml\" with one of the options listed at:\n  https://kubernetes.io/docs/concepts/cluster-administration/addons/\n\nThen you can join any number of worker nodes by running the following on each as root:\n\nkubeadm join 10.0.0.98:6443 --token yer1c6.3g0jps020b58t39q \\\n    --discovery-token-ca-cert-hash sha256:442900d0706efc3c62a5d5d57a58e270ebe24d75cbb8774f537654ca120e33eb "
}
...
...
TASK [debug] *********************************************************
ok: [knode2.home.net] => {
    "msg": {
        "changed": true, 
        "cmd": "kubeadm join 10.0.0.98:6443 --token 9th6ul.s0jet2tqmme84j93     --discovery-token-ca-cert-hash sha256:442900d0706efc3c62a5d5d57a58e270ebe24d75cbb8774f537654ca120e33eb ", 
        "delta": "0:00:22.303194", 
        "end": "2019-10-11 17:43:09.595894", 
        "failed": false, 
        "rc": 0, 
        "start": "2019-10-11 17:42:47.292700", 
        "stderr": "\t[WARNING SystemVerification]: this Docker version is not on the list of validated versions: 19.03.3. Latest validated version: 18.09", 
        "stderr_lines": [
            "\t[WARNING SystemVerification]: this Docker version is not on the list of validated versions: 19.03.3. Latest validated version: 18.09"
        ], 
        "stdout": "[preflight] Running pre-flight checks\n[preflight] Reading configuration from the cluster...\n[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'\n[kubelet-start] Downloading configuration for the kubelet from the \"kubelet-config-1.16\" ConfigMap in the kube-system namespace\n[kubelet-start] Writing kubelet configuration to file \"/var/lib/kubelet/config.yaml\"\n[kubelet-start] Writing kubelet environment file with flags to file \"/var/lib/kubelet/kubeadm-flags.env\"\n[kubelet-start] Activating the kubelet service\n[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...\n\nThis node has joined the cluster:\n* Certificate signing request was sent to apiserver and a response was received.\n* The Kubelet was informed of the new secure connection details.\n\nRun 'kubectl get nodes' on the control-plane to see this node join the cluster.", 
        "stdout_lines": [
            "[preflight] Running pre-flight checks", 
            "[preflight] Reading configuration from the cluster...", 
            "[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'", 
            "[kubelet-start] Downloading configuration for the kubelet from the \"kubelet-config-1.16\" ConfigMap in the kube-system namespace", 
            "[kubelet-start] Writing kubelet configuration to file \"/var/lib/kubelet/config.yaml\"", 
            "[kubelet-start] Writing kubelet environment file with flags to file \"/var/lib/kubelet/kubeadm-flags.env\"", 
            "[kubelet-start] Activating the kubelet service", 
            "[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...", 
            "", 
            "This node has joined the cluster:", 
            "* Certificate signing request was sent to apiserver and a response was received.", 
            "* The Kubelet was informed of the new secure connection details.", 
            "", 
            "Run 'kubectl get nodes' on the control-plane to see this node join the cluster."
        ]
    }
}

PLAY RECAP *********************************************************
kclustert.home.net         : ok=40   changed=32   unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
knode2.home.net            : ok=33   changed=27   unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

# Install Cluster

```
Daniels-MacBook-Pro:kube daniel$ ansible-playbook -i cicd_hosts.yaml playbooks/k8s/deploy_cluster.yaml 

PLAY [cluster] *********************************************************

TASK [Gathering Facts] *********************************************************
ok: [kclustert.home.net]

TASK [Define GLOBAL Variables] *********************************************************
changed: [kclustert.home.net]
...
...
TASK [debug] *****************************************************************************************************************************************************************************************************************************************************************
ok: [kclustert.home.net] => {
    "msg": "[init] Using Kubernetes version: v1.16.1\n[preflight] Running pre-flight checks\n[preflight] Pulling images required for setting up a Kubernetes cluster\n[preflight] This might take a minute or two, depending on the speed of your internet connection\n[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'\n[kubelet-start] Writing kubelet environment file with flags to file \"/var/lib/kubelet/kubeadm-flags.env\"\n[kubelet-start] Writing kubelet configuration to file \"/var/lib/kubelet/config.yaml\"\n[kubelet-start] Activating the kubelet service\n[certs] Using certificateDir folder \"/etc/kubernetes/pki\"\n[certs] Generating \"ca\" certificate and key\n[certs] Generating \"apiserver\" certificate and key\n[certs] apiserver serving cert is signed for DNS names [kclustert.home.net kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 10.0.0.98]\n[certs] Generating \"apiserver-kubelet-client\" certificate and key\n[certs] Generating \"front-proxy-ca\" certificate and key\n[certs] Generating \"front-proxy-client\" certificate and key\n[certs] Generating \"etcd/ca\" certificate and key\n[certs] Generating \"etcd/server\" certificate and key\n[certs] etcd/server serving cert is signed for DNS names [kclustert.home.net localhost] and IPs [10.0.0.98 127.0.0.1 ::1]\n[certs] Generating \"etcd/peer\" certificate and key\n[certs] etcd/peer serving cert is signed for DNS names [kclustert.home.net localhost] and IPs [10.0.0.98 127.0.0.1 ::1]\n[certs] Generating \"etcd/healthcheck-client\" certificate and key\n[certs] Generating \"apiserver-etcd-client\" certificate and key\n[certs] Generating \"sa\" key and public key\n[kubeconfig] Using kubeconfig folder \"/etc/kubernetes\"\n[kubeconfig] Writing \"admin.conf\" kubeconfig file\n[kubeconfig] Writing \"kubelet.conf\" kubeconfig file\n[kubeconfig] Writing \"controller-manager.conf\" kubeconfig file\n[kubeconfig] Writing \"scheduler.conf\" kubeconfig file\n[control-plane] Using manifest folder \"/etc/kubernetes/manifests\"\n[control-plane] Creating static Pod manifest for \"kube-apiserver\"\n[control-plane] Creating static Pod manifest for \"kube-controller-manager\"\n[control-plane] Creating static Pod manifest for \"kube-scheduler\"\n[etcd] Creating static Pod manifest for local etcd in \"/etc/kubernetes/manifests\"\n[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory \"/etc/kubernetes/manifests\". This can take up to 4m0s\n[apiclient] All control plane components are healthy after 35.503706 seconds\n[upload-config] Storing the configuration used in ConfigMap \"kubeadm-config\" in the \"kube-system\" Namespace\n[kubelet] Creating a ConfigMap \"kubelet-config-1.16\" in namespace kube-system with the configuration for the kubelets in the cluster\n[upload-certs] Skipping phase. Please see --upload-certs\n[mark-control-plane] Marking the node kclustert.home.net as control-plane by adding the label \"node-role.kubernetes.io/master=''\"\n[mark-control-plane] Marking the node kclustert.home.net as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule]\n[bootstrap-token] Using token: myicac.8oepry1i37mh5m55\n[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles\n[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials\n[bootstrap-token] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token\n[bootstrap-token] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster\n[bootstrap-token] Creating the \"cluster-info\" ConfigMap in the \"kube-public\" namespace\n[addons] Applied essential addon: CoreDNS\n[addons] Applied essential addon: kube-proxy\n\nYour Kubernetes control-plane has initialized successfully!\n\nTo start using your cluster, you need to run the following as a regular user:\n\n  mkdir -p $HOME/.kube\n  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config\n  sudo chown $(id -u):$(id -g) $HOME/.kube/config\n\nYou should now deploy a pod network to the cluster.\nRun \"kubectl apply -f [podnetwork].yaml\" with one of the options listed at:\n  https://kubernetes.io/docs/concepts/cluster-administration/addons/\n\nThen you can join any number of worker nodes by running the following on each as root:\n\nkubeadm join 10.0.0.98:6443 --token myicac.8oepry1i37mh5m55 \\\n    --discovery-token-ca-cert-hash sha256:f5855ee6907178e423e8dcd3e47b4ac5d78868ee3271f6151c33a713119902ed "
}

TASK [Mkdir $HOME/.kube] *********************************************************
 [WARNING]: Consider using the file module with state=absent rather than running 'rm'.  If you need to use command because file is insufficient you can add 'warn: false' to this command task or set 'command_warnings=False' in ansible.cfg to get rid of this message.

changed: [kclustert.home.net]

TASK [Copy kube config to users home] *********************************************************
changed: [kclustert.home.net]

TASK [Apply correct permissions] *********************************************************
changed: [kclustert.home.net]

PLAY RECAP *********************************************************
kclustert.home.net         : ok=34   changed=20   unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

```

# Install Network

```
Daniels-MacBook-Pro:kube daniel$ ansible-playbook -i cicd_hosts.yaml playbooks/k8s/deploy_network.yaml 

PLAY [cluster] *********************************************************

TASK [Gathering Facts] *********************************************************
ok: [kclustert.home.net]

TASK [Install Calico Network] *********************************************************
changed: [kclustert.home.net]

PLAY RECAP *********************************************************
kclustert.home.net         : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

```

# Install Node

```
Daniels-MacBook-Pro:kube daniel$ ansible-playbook -i cicd_hosts.yaml playbooks/k8s/deploy_node.yaml 

PLAY [cluster] *********************************************************

TASK [Gathering Facts] *********************************************************
ok: [kclustert.home.net]

TASK [Get the command information to later join the node] *********************************************************
changed: [kclustert.home.net]

TASK [set_fact] *********************************************************
ok: [kclustert.home.net]

TASK [debug] *********************************************************
ok: [kclustert.home.net] => {
    "msg": "kubeadm join 10.0.0.98:6443 --token 3oc898.6kmp2i7jq82wo6z2     --discovery-token-ca-cert-hash sha256:f5855ee6907178e423e8dcd3e47b4ac5d78868ee3271f6151c33a713119902ed "
}
...
...
TASK [Join the Node server into the K8 cluster] *********************************************************
changed: [knode2.home.net]

TASK [debug] *********************************************************
ok: [knode2.home.net] => {
    "msg": {
        "changed": true, 
        "cmd": "kubeadm join 10.0.0.98:6443 --token 3oc898.6kmp2i7jq82wo6z2     --discovery-token-ca-cert-hash sha256:f5855ee6907178e423e8dcd3e47b4ac5d78868ee3271f6151c33a713119902ed ", 
        "delta": "0:00:22.363906", 
        "end": "2019-10-11 19:35:23.198114", 
        "failed": false, 
        "rc": 0, 
        "start": "2019-10-11 19:35:00.834208", 
        "stderr": "\t[WARNING SystemVerification]: this Docker version is not on the list of validated versions: 19.03.3. Latest validated version: 18.09", 
        "stderr_lines": [
            "\t[WARNING SystemVerification]: this Docker version is not on the list of validated versions: 19.03.3. Latest validated version: 18.09"
        ], 
        "stdout": "[preflight] Running pre-flight checks\n[preflight] Reading configuration from the cluster...\n[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'\n[kubelet-start] Downloading configuration for the kubelet from the \"kubelet-config-1.16\" ConfigMap in the kube-system namespace\n[kubelet-start] Writing kubelet configuration to file \"/var/lib/kubelet/config.yaml\"\n[kubelet-start] Writing kubelet environment file with flags to file \"/var/lib/kubelet/kubeadm-flags.env\"\n[kubelet-start] Activating the kubelet service\n[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...\n\nThis node has joined the cluster:\n* Certificate signing request was sent to apiserver and a response was received.\n* The Kubelet was informed of the new secure connection details.\n\nRun 'kubectl get nodes' on the control-plane to see this node join the cluster.", 
        "stdout_lines": [
            "[preflight] Running pre-flight checks", 
            "[preflight] Reading configuration from the cluster...", 
            "[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'", 
            "[kubelet-start] Downloading configuration for the kubelet from the \"kubelet-config-1.16\" ConfigMap in the kube-system namespace", 
            "[kubelet-start] Writing kubelet configuration to file \"/var/lib/kubelet/config.yaml\"", 
            "[kubelet-start] Writing kubelet environment file with flags to file \"/var/lib/kubelet/kubeadm-flags.env\"", 
            "[kubelet-start] Activating the kubelet service", 
            "[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...", 
            "", 
            "This node has joined the cluster:", 
            "* Certificate signing request was sent to apiserver and a response was received.", 
            "* The Kubelet was informed of the new secure connection details.", 
            "", 
            "Run 'kubectl get nodes' on the control-plane to see this node join the cluster."
        ]
    }
}

PLAY RECAP *********************************************************
kclustert.home.net         : ok=4    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
knode2.home.net            : ok=31   changed=26   unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   


```

# Credits
A lot of this work has been inspired by https://github.com/geerlingguy/ansible-role-kubernetes (from Jeff Geerling  https://github.com/geerlingguy), those are way more elegant and sophysticated scripts that delivers a running cluster.  
