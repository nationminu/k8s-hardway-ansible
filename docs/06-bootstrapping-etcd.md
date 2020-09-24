# Bootstrapping the etcd Cluster
Kubernetes 는 클러스터 상태를 etcd 에 저장합니다.
```
---
# tasks file for boostrapping/etcd
- name: Download and Install the etcd Binaries (1/2)
  unarchive:
    src: "{{ item }}"
    dest: "/tmp" 
    remote_src: yes
  with_items:
    - "https://storage.googleapis.com/etcd/{{ ETCD_VER }}/etcd-{{ ETCD_VER }}-linux-amd64.tar.gz"

- name: Download and Install the etcd Binaries (2/2)
  copy: 
    src: "/tmp/etcd-{{ ETCD_VER }}-linux-amd64/{{ item }}"
    dest: "{{ KUBERNETES_BINARY_PATH }}"
    mode: 0700
    remote_src: yes
  with_items:
    - etcd
    - etcdctl

- name: Recursively remove directory
  file:
    path: "/tmp/etcd-{{ ETCD_VER }}-linux-amd64/"
    state: absent

- name: Configure the etcd Server (1/2)
  file:
    path: "{{ item }}"
    state: directory
    mode: '0700'
  with_items:
    - "/etc/etcd"
    - "/var/lib/etcd" 

- name: Configure the etcd Server (2/2)
  copy: 
    src: "{{ KUBERNETES_DIRECTORY }}/{{ item }}"
    dest: "/etc/etcd"
    mode: 0600
    remote_src: yes
  with_items:
    - pki/ca/ca.pem 
    - pki/api/kubernetes-key.pem
    - pki/api/kubernetes.pem

- name: Create the etcd.service systemd unit file
  template: 
    src: "etcd.service.j2"
    dest: "/etc/systemd/system/etcd.service"
    mode: 0700

- name: Start the etcd Server
  systemd:
    name: "{{ item }}" 
    daemon_reload: yes
    state: restarted
    enabled: yes
  with_items:
    - etcd

- name: Verification ETCD
  shell: >
    ETCDCTL_API=3 {{ KUBERNETES_BINARY_PATH }}/etcdctl member list 
    --endpoints=https://127.0.0.1:2379 
    --cacert=/etc/etcd/ca.pem 
    --cert=/etc/etcd/kubernetes.pem 
    --key=/etc/etcd/kubernetes-key.pem
  register: "verification_result" 

- debug:
    msg: 
      - "Verification result :  {{ verification_result.stdout_lines }} " 
    verbosity: 0     
```

* 
```
ETCD_VER="v3.4.13"
wget https://storage.googleapis.com/etcd/${ETCD_VER}/etcd-${ETCD_VER}-linux-amd64.tar.gz

tar -xvf etcd-${ETCD_VER}-linux-amd64.tar.gz
sudo mv etcd-${ETCD_VER}-linux-amd64/etcd* /usr/local/bin/

sudo mkdir -p /etc/etcd /var/lib/etcd
sudo chmod 700 /var/lib/etcd
sudo cp ca.pem kubernetes-key.pem kubernetes.pem /etc/etcd/ 

ETCD_NAME=$(hostname -s)

INTERNAL_IP=$(hostnale -I)

cat <<EOF | sudo tee /etc/systemd/system/etcd.service
[Unit]
Description=etcd
Documentation=https://github.com/coreos

[Service]
Type=notify
ExecStart=/usr/local/bin/etcd \\
  --name ${ETCD_NAME} \\
  --cert-file=/etc/etcd/kubernetes.pem \\
  --key-file=/etc/etcd/kubernetes-key.pem \\
  --peer-cert-file=/etc/etcd/kubernetes.pem \\
  --peer-key-file=/etc/etcd/kubernetes-key.pem \\
  --trusted-ca-file=/etc/etcd/ca.pem \\
  --peer-trusted-ca-file=/etc/etcd/ca.pem \\
  --peer-client-cert-auth \\
  --client-cert-auth \\
  --initial-advertise-peer-urls https://${INTERNAL_IP}:2380 \\
  --listen-peer-urls https://${INTERNAL_IP}:2380 \\
  --listen-client-urls https://${INTERNAL_IP}:2379,https://127.0.0.1:2379 \\
  --advertise-client-urls https://${INTERNAL_IP}:2379 \\
  --initial-cluster-token etcd-cluster-0 \\
  --initial-cluster controller-0=https://10.65.40.11:2380,controller-1=https://10.65.40.12:2380,controller-2=https://10.65.40.13:2380 \\
  --initial-cluster-state new \\
  --data-dir=/var/lib/etcd
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

``` 
sudo systemctl daemon-reload
sudo systemctl enable etcd
sudo systemctl start etcd
```

Verification
``` 
sudo ETCDCTL_API=3 etcdctl member list \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/etcd/ca.pem \
  --cert=/etc/etcd/kubernetes.pem \
  --key=/etc/etcd/kubernetes-key.pem
```
Output
```
3a57933972cb5131, started, controller-2, https://10.65.40.13:2380, https://10.240.0.13:2379, false
f98dc20bce6225a0, started, controller-0, https://10.65.40.11:2380, https://10.65.40.11:2379, false
ffed16798470cab5, started, controller-1, https://10.65.40.12:2380, https://10.65.40.12:2379, false
```
