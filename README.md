# calico-vpp-configs

## Provision the cluster
### 1. Create an Amzaon EKS cluster without any nodes.
```shell
eksctl create cluster --name my-calico-cluster --without-nodegroup
```

### 2. Delete the aws-node DaemonSet to disable the default AWS VPC networking for the pods.
```shell
kubectl delete daemonset -n kube-system aws-node
```

## Install and configure Calico with the VPP dataplane
### 1. Install the Tigera operator
```shell
kubectl apply -f https://projectcalico.docs.tigera.io/manifests/tigera-operator.yaml
```

### 2. Configure the Calico installation for the VPP dataplane.
```shell
kubectl apply -f https://raw.githubusercontent.com/projectcalico/vpp-dataplane/v3.23.0/yaml/calico/installation-eks.yaml
```

### 3. Install the VPP dataplane components
```shell
kubectl apply -f https://github.com/beumjincho/calico-vpp-configs/blob/main/calico/calico-vpp-eks-dpdk.yaml
```

### 4. Add nodes to the cluster
Don't forget to edit the cluster name, region and other fields as appropriate for your cluster.
```shell
cat <<EOF | eksctl create nodegroup -f -
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig
metadata:
  name: my-calico-cluster
  region: us-east-2
managedNodeGroups:
- name: my-calico-cluster-ng
  desiredCapacity: 2
  instanceType: t3.large
  labels: {role: worker}
  preBootstrapCommands:
    - sudo curl -o /tmp/init_eks.sh "https://raw.githubusercontent.com/projectcalico/vpp-dataplane/master/scripts/init_eks.sh"
    - sudo chmod +x /tmp/init_eks.sh
    - sudo /tmp/init_eks.sh
EOF
```
