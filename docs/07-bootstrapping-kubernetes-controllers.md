# Bootstrapping the Kubernetes Control Plane

## ansible tasks
```
--- 
# tasks file for boostrapping/control-plane

- name: Download and Install the Kubernetes Controller Binaries
  get_url:
    url: "{{ item }}"
    dest: "{{ KUBERNETES_BINARY_PATH }}"
    mode: '0700'
  with_items: 
    - "https://storage.googleapis.com/kubernetes-release/release/{{ KUBERNETES_VERSION }}/bin/linux/amd64/kube-apiserver"
    - "https://storage.googleapis.com/kubernetes-release/release/{{ KUBERNETES_VERSION }}/bin/linux/amd64/kube-controller-manager"
    - "https://storage.googleapis.com/kubernetes-release/release/{{ KUBERNETES_VERSION }}/bin/linux/amd64/kube-scheduler"

- name: Configure the Kubernetes API Server (1/2)
  file:
    path: "{{ item }}"
    state: directory
    mode: '0700'
  with_items: 
    - "/var/lib/kubernetes" 

- name: Configure the Kubernetes API Server (2/2)
  copy: 
    src: "{{ KUBERNETES_DIRECTORY }}/{{ item }}"
    dest: "/var/lib/kubernetes"
    mode: 0600
    remote_src: yes
  with_items:
    - pki/ca/ca.pem 
    - pki/ca/ca-key.pem
    - pki/api/kubernetes-key.pem
    - pki/api/kubernetes.pem  
    - pki/service-account/service-account-key.pem 
    - pki/service-account/service-account.pem
    - secret/encryption-config.yaml  
    - kubeconfig/kube-controller-manager.kubeconfig
    - kubeconfig/kube-scheduler.kubeconfig
 
- name: Create the kube-apiserver.service systemd unit file
  template: 
    src: "kube-apiserver.service.j2"
    dest: "/etc/systemd/system/kube-apiserver.service"
    mode: 0700

- name: Create the kube-controller-manager.service systemd unit file
  template: 
    src: "kube-controller-manager.service.j2"
    dest: "/etc/systemd/system/kube-controller-manager.service"
    mode: 0700

- name: Create the kube-scheduler.yaml configuration file
  copy: 
    src: "kube-scheduler.yaml"
    dest: "/etc/kubernetes/config/kube-scheduler.yaml"
    mode: 0600  

- name: Create the kube-scheduler.service systemd unit file
  copy: 
    src: "kube-scheduler.service"
    dest: "/etc/systemd/system/kube-scheduler.service"
    mode: 0600  

- name: Start the etcd Server
  systemd:
    name: "{{ item }}" 
    daemon_reload: yes
    state: started
    enabled: yes
  with_items:
    - kube-apiserver 
    - kube-controller-manager 
    - kube-scheduler    

```

* kube-apiserver.service.j2
```
[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-apiserver \
  --advertise-address={{ hostvars[inventory_hostname].ansible_host }} \
  --allow-privileged=true \
  --apiserver-count=3 \
  --audit-log-maxage=30 \
  --audit-log-maxbackup=3 \
  --audit-log-maxsize=100 \
  --audit-log-path=/var/log/audit.log \
  --authorization-mode=Node,RBAC \
  --bind-address=0.0.0.0 \
  --client-ca-file=/var/lib/kubernetes/ca.pem \
  --enable-admission-plugins=NamespaceLifecycle,NodeRestriction,LimitRanger,ServiceAccount,DefaultStorageClass,ResourceQuota \
  --etcd-cafile=/var/lib/kubernetes/ca.pem \
  --etcd-certfile=/var/lib/kubernetes/kubernetes.pem \
  --etcd-keyfile=/var/lib/kubernetes/kubernetes-key.pem \
{% if groups['kube-master'] | length > 1 %}
  --etcd-servers=https://{{ hostvars[groups['kube-master'][0]].ansible_host }}:2379,https://{{ hostvars[groups['kube-master'][1]].ansible_host }}:2379,https://{{ hostvars[groups['kube-master'][2]].ansible_host }}:2379 \
{% else %}
  --etcd-servers=https://{{ hostvars[groups['kube-master'][0]].ansible_host }}:2379 \
{% endif %}
  --event-ttl=1h \
  --encryption-provider-config=/var/lib/kubernetes/encryption-config.yaml \
  --kubelet-certificate-authority=/var/lib/kubernetes/ca.pem \
  --kubelet-client-certificate=/var/lib/kubernetes/kubernetes.pem \
  --kubelet-client-key=/var/lib/kubernetes/kubernetes-key.pem \
  --kubelet-https=true \
  --service-account-key-file=/var/lib/kubernetes/service-account.pem \
  --service-cluster-ip-range=10.32.0.0/24 \
  --service-node-port-range=30000-32767 \
  --tls-cert-file=/var/lib/kubernetes/kubernetes.pem \
  --tls-private-key-file=/var/lib/kubernetes/kubernetes-key.pem \
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

* kube-controller-manager.service.j2
```
[Unit]
Description=Kubernetes Controller Manager
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-controller-manager \
  --bind-address=0.0.0.0 \
  --cluster-cidr={{ POD_CIDR }} \
  --cluster-name=kubernetes \
  --cluster-signing-cert-file=/var/lib/kubernetes/ca.pem \
  --cluster-signing-key-file=/var/lib/kubernetes/ca-key.pem \
  --kubeconfig=/var/lib/kubernetes/kube-controller-manager.kubeconfig \
  --leader-elect=true \
  --root-ca-file=/var/lib/kubernetes/ca.pem \
  --service-account-private-key-file=/var/lib/kubernetes/service-account-key.pem \
  --service-cluster-ip-range=10.32.0.0/24 \
  --use-service-account-credentials=true \
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```
* kube-scheduler.yaml
```
apiVersion: kubescheduler.config.k8s.io/v1alpha1
kind: KubeSchedulerConfiguration
clientConnection:
  kubeconfig: "/var/lib/kubernetes/kube-scheduler.kubeconfig"
leaderElection:
  leaderElect: true
```

* kube-scheduler.service
```
[Unit]
Description=Kubernetes Scheduler
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-scheduler \
  --config=/etc/kubernetes/config/kube-scheduler.yaml \
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target 
```
