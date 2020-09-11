# Kubernetes Hardway with Ansible

본 튜토리얼은 Kubernetes hardway 과정을 ansible-playbook 으로 변경해서 실행하는 과정을 담고있습니다.

#Cluster Details
 
* [kubernetes](https://github.com/kubernetes/kubernetes) v1.18.8
* [containerd](https://github.com/containerd/containerd) v1.4.0
* [coredns](https://github.com/coredns/coredns) v1.7.0
* [cni](https://github.com/containernetworking/cni) v0.8.7
* [etcd](https://github.com/coreos/etcd) v3.4.13

--- 

* [cri-o] crio v1.18
* [runc] runc v1.0.0-rc92

## Labs
본 튜토리얼은 RHEV 환경에서 진행하며 VM 생성 과정은 포함하지 않습니다.  

* [Prerequisites](docs/01-prerequisites.md)
* [Installing the Client Tools](docs/02-client-tools.md) 
* [Provisioning the CA and Generating TLS Certificates](docs/04-certificate-authority.md)
* [Generating Kubernetes Configuration Files for Authentication](docs/05-kubernetes-configuration-files.md)
* [Generating the Data Encryption Config and Key](docs/06-data-encryption-keys.md)
* [Bootstrapping the etcd Cluster](docs/07-bootstrapping-etcd.md)
* [Bootstrapping the Kubernetes Control Plane](docs/08-bootstrapping-kubernetes-controllers.md)
* [Bootstrapping the Kubernetes Worker Nodes](docs/09-bootstrapping-kubernetes-workers.md)
* [Configuring kubectl for Remote Access](docs/10-configuring-kubectl.md)
* [Provisioning Pod Network Routes](docs/11-pod-network-routes.md)
* [Deploying the DNS Cluster Add-on](docs/12-dns-addon.md)
* [Smoke Test](docs/13-smoke-test.md)
* [Cleaning Up](docs/14-cleanup.md)