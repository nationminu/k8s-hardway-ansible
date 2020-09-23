# Kubernetes Hardway with Ansible

본 튜토리얼은 Kubernetes hardway 과정을 ansible-playbook 으로 변경해서 실행하는 과정을 담고있습니다.

#Cluster Details
 
* [kubernetes](https://github.com/kubernetes/kubernetes) v1.18.8
* [containerd](https://github.com/containerd/containerd) v1.4.0
* [coredns](https://github.com/coredns/coredns) v1.7.0
* [cni](https://github.com/containernetworking/cni) v0.8.7
* [etcd](https://github.com/coreos/etcd) v3.4.13

--- 

* [cri-o](https://cri-o.io/) v1.18
* [runc](https://github.com/opencontainers/runc) v1.0.0-rc92

## Labs
본 튜토리얼은 RHEV 환경에서 진행하며 VM 생성 과정은 포함하지 않습니다.  

* [Prerequisites](docs/01-prerequisites.md)
* [Installing the Client Tools](docs/02-install-tools.md) 
* [Provisioning the CA and Generating TLS Certificates](docs/03-provisioning-ca.md)
* [Generating Kubernetes Configuration Files for Authentication](docs/04-kubernetes-configuration-files.md)
* [Generating the Data Encryption Config and Key](docs/05-data-encryption-keys.md)
* [Bootstrapping the etcd Cluster](docs/06-bootstrapping-etcd.md)
* [Bootstrapping the Kubernetes Control Plane](docs/07-bootstrapping-kubernetes-controllers.md)
* [Bootstrapping the Kubernetes Worker Nodes](docs/08-bootstrapping-kubernetes-workers.md) 
* [Provisioning Pod Network Routes](docs/09-pod-network-routes.md)
* [Deploying the DNS Cluster Add-on](docs/10-dns-addon.md)
* [Smoke Test](docs/11-smoke-test.md) 