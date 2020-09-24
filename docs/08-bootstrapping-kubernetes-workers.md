# Bootstrapping the Kubernetes Worker Nodes

## ansible tasks
```
---
# tasks file for boostrapping/worker-node

- name: Configure the Worker Server 
  file:
    path: "{{ item }}"
    state: directory
    mode: '0700'
  with_items: 
    - /etc/cni/net.d
    - /opt/cni/bin
    - /var/lib/kubelet
    - /var/lib/kube-proxy
    - /var/lib/kubernetes
    - /var/run/kubernetes   
    - /etc/containerd 
    - /tmp/containerd
  tags:
    - always
    
- import_tasks: install.yaml  
  tags:
    - install

- import_tasks: docker.yaml
  tags:
    - runtime
  when: "KUBERNETES_RUNTIME == 'docker'"

- import_tasks: containerd.yaml
  tags:
    - runtime
  when: "KUBERNETES_RUNTIME == 'containerd'"

- import_tasks: crio.yaml
  tags:
    - runtime
  when: "KUBERNETES_RUNTIME == 'crio'" 

- import_tasks: service.yaml  
  tags:
    - service 
```

### install.yaml
```
---
- name: Download and Install Binaries for control plane
  get_url:
    url: "{{ item }}"
    dest: "{{ KUBERNETES_BINARY_PATH }}"
    mode: '0700'
  with_items:   
    - https://storage.googleapis.com/kubernetes-release/release/{{ KUBERNETES_VERSION }}/bin/linux/amd64/kubectl 
    - https://storage.googleapis.com/kubernetes-release/release/{{ KUBERNETES_VERSION }}/bin/linux/amd64/kube-proxy 
    - https://storage.googleapis.com/kubernetes-release/release/{{ KUBERNETES_VERSION }}/bin/linux/amd64/kubelet    

- name: Download and Install Binaries for control plane - cni
  unarchive:
    src: "{{ item }}"
    dest: "/opt/cni/bin/" 
    remote_src: yes    
    mode: '0700'
  with_items:   
    - https://github.com/containernetworking/plugins/releases/download/{{ CNI_VER }}/cni-plugins-linux-amd64-{{ CNI_VER }}.tgz 

```

### runtime : docker, containerd, crio
```

```

### service.yaml
```

```