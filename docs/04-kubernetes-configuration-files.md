# Generating Kubernetes Configuration Files for Authentication

# The kubelet Kubernetes Configuration File

```
- hosts: kube-master[0]
  any_errors_fatal: "{{ any_errors_fatal | default(true) }}"
  gather_facts: False
  roles:
    ...
    - { role: generating-kubeconfig, tags: ["node", "admin", "controller", "proxy", "scheduler"]}
```

* kubelet 을 위해 사용하는 kubeconfig 파일을 생성합니다.
```
---
# 1. The kubelet Kubernetes Configuration File
- name: Generate a kubeconfig file for each worker node (1/2)
#  shell: >
  copy:
    dest: "{{ KUBERNETES_DIRECTORY }}/genconfig-{{ item }}.sh"
    content: |
      #/usr/bin/env bash

      {{ KUBERNETES_BINARY_PATH }}/kubectl config set-cluster {{ KUBERNETS_CLUSTER_NAME }} \
      --certificate-authority={{ KUBERNETES_DIRECTORY }}/pki/ca/ca.pem \
      --embed-certs=true \
      --server=https://{{ KUBERNETES_PUBLIC_ADDRESS }}:6443 \
      --kubeconfig={{ KUBERNETES_DIRECTORY }}/kubeconfig/{{ item }}.kubeconfig \
        
      {{ KUBERNETES_BINARY_PATH }}/kubectl config set-credentials system:node:{{ item }} \
      --client-certificate={{ KUBERNETES_DIRECTORY }}/pki/clients/{{ item }}.pem \
      --client-key={{ KUBERNETES_DIRECTORY }}/pki/clients/{{ item }}-key.pem \
      --embed-certs=true \
      --kubeconfig={{ KUBERNETES_DIRECTORY }}/kubeconfig/{{ item }}.kubeconfig
        
      {{ KUBERNETES_BINARY_PATH }}/kubectl config set-context default \
      --cluster={{ KUBERNETS_CLUSTER_NAME }} \
      --user=system:node:{{ item }} \
      --kubeconfig={{ KUBERNETES_DIRECTORY }}/kubeconfig/{{ item }}.kubeconfig \
        
      {{ KUBERNETES_BINARY_PATH }}/kubectl config use-context default --kubeconfig={{ KUBERNETES_DIRECTORY }}/kubeconfig/{{ item }}.kubeconfig 
    mode: 0700
  with_items:
    - "{{ groups['kube-node'] }}"

- name: Generate a kubeconfig file for each worker node  (2/2)
  shell: "{{ KUBERNETES_DIRECTORY }}/genconfig-{{ item }}.sh"
  register: "node_result"
  with_items:
    - "{{ groups['kube-node'] }}"

- debug:
    msg: 
      - "node result :  {{ item.results }} " 
    verbosity: 2
  with_items:
    - "{{ node_result }}"
```
* kube-proxy 를 위해 사용하는 kubeconfig 파일을 생성합니다.
```
---
# 1. The kubelet Kubernetes Configuration File
- name: Generate a kubeconfig file for the kube-proxy service (1/2)
#  shell: >
  copy:
    dest: "{{ KUBERNETES_DIRECTORY }}/genconfig-proxy.sh"
    content: |
      #/usr/bin/env bash

      {{ KUBERNETES_BINARY_PATH }}/kubectl config set-cluster {{ KUBERNETS_CLUSTER_NAME }} \
      --certificate-authority={{ KUBERNETES_DIRECTORY }}/pki/ca/ca.pem \
      --embed-certs=true \
      --server=https://{{ KUBERNETES_PUBLIC_ADDRESS }}:6443 \
      --kubeconfig={{ KUBERNETES_DIRECTORY }}/kubeconfig/kube-proxy.kubeconfig \
        
      {{ KUBERNETES_BINARY_PATH }}/kubectl config set-credentials system:kube-proxy \
      --client-certificate={{ KUBERNETES_DIRECTORY }}/pki/proxy/kube-proxy.pem \
      --client-key={{ KUBERNETES_DIRECTORY }}/pki/proxy/kube-proxy-key.pem \
      --embed-certs=true \
      --kubeconfig={{ KUBERNETES_DIRECTORY }}/kubeconfig/kube-proxy.kubeconfig
        
      {{ KUBERNETES_BINARY_PATH }}/kubectl config set-context default \
      --cluster={{ KUBERNETS_CLUSTER_NAME }} \
      --user=system:kube-proxy \
      --kubeconfig={{ KUBERNETES_DIRECTORY }}/kubeconfig/kube-proxy.kubeconfig \
        
      {{ KUBERNETES_BINARY_PATH }}/kubectl config use-context default --kubeconfig={{ KUBERNETES_DIRECTORY }}/kubeconfig/kube-proxy.kubeconfig 
    mode: 0700 

- name: Generate a kubeconfig file for the kube-proxy service (2/2)
  shell: "{{ KUBERNETES_DIRECTORY }}/genconfig-proxy.sh"
  register: "proxy_result" 

- debug:
    msg: 
      - "proxy result :  {{ proxy_result }} " 
    verbosity: 2 
```

* kube-controller-manager 를 위해 사용하는 kubeconfig 파일을 생성합니다.
```
---
# 1. The kubelet Kubernetes Configuration File
- name: Generate a kubeconfig file for the kube-controller-manager service (1/2)
#  shell: >
  copy:
    dest: "{{ KUBERNETES_DIRECTORY }}/genconfig-controller.sh"
    content: |
      #/usr/bin/env bash

      {{ KUBERNETES_BINARY_PATH }}/kubectl config set-cluster {{ KUBERNETS_CLUSTER_NAME }} \
      --certificate-authority={{ KUBERNETES_DIRECTORY }}/pki/ca/ca.pem \
      --embed-certs=true \
      --server=https://{{ KUBERNETES_PUBLIC_ADDRESS }}:6443 \
      --kubeconfig={{ KUBERNETES_DIRECTORY }}/kubeconfig/kube-controller-manager.kubeconfig \
        
      {{ KUBERNETES_BINARY_PATH }}/kubectl config set-credentials system:kube-controller-manager \
      --client-certificate={{ KUBERNETES_DIRECTORY }}/pki/controller/kube-controller-manager.pem \
      --client-key={{ KUBERNETES_DIRECTORY }}/pki/controller/kube-controller-manager-key.pem \
      --embed-certs=true \
      --kubeconfig={{ KUBERNETES_DIRECTORY }}/kubeconfig/kube-controller-manager.kubeconfig
        
      {{ KUBERNETES_BINARY_PATH }}/kubectl config set-context default \
      --cluster={{ KUBERNETS_CLUSTER_NAME }} \
      --user=system:kube-controller-manager \
      --kubeconfig={{ KUBERNETES_DIRECTORY }}/kubeconfig/kube-controller-manager.kubeconfig \
        
      {{ KUBERNETES_BINARY_PATH }}/kubectl config use-context default --kubeconfig={{ KUBERNETES_DIRECTORY }}/kubeconfig/kube-controller-manager.kubeconfig 
    mode: 0700 

- name: Generate a kubeconfig file for the kube-controller-manager service (2/2)
  shell: "{{ KUBERNETES_DIRECTORY }}/genconfig-controller.sh"
  register: "controller_result" 

- debug:
    msg: 
      - "controller result :  {{ controller_result }} " 
    verbosity: 2 
```

* kube-scheduler 를 위해 사용하는 kubeconfig 파일을 생성합니다.
```
---
# 1. The kubelet Kubernetes Configuration File
- name: Generate a kubeconfig file for the kube-scheduler service (1/2)
#  shell: >
  copy:
    dest: "{{ KUBERNETES_DIRECTORY }}/genconfig-scheduler.sh"
    content: |
      #/usr/bin/env bash

      {{ KUBERNETES_BINARY_PATH }}/kubectl config set-cluster {{ KUBERNETS_CLUSTER_NAME }} \
      --certificate-authority={{ KUBERNETES_DIRECTORY }}/pki/ca/ca.pem \
      --embed-certs=true \
      --server=https://{{ KUBERNETES_PUBLIC_ADDRESS }}:6443 \
      --kubeconfig={{ KUBERNETES_DIRECTORY }}/kubeconfig/kube-scheduler.kubeconfig \
        
      {{ KUBERNETES_BINARY_PATH }}/kubectl config set-credentials system:kube-scheduler \
      --client-certificate={{ KUBERNETES_DIRECTORY }}/pki/scheduler/kube-scheduler.pem \
      --client-key={{ KUBERNETES_DIRECTORY }}/pki/scheduler/kube-scheduler-key.pem \
      --embed-certs=true \
      --kubeconfig={{ KUBERNETES_DIRECTORY }}/kubeconfig/kube-scheduler.kubeconfig
        
      {{ KUBERNETES_BINARY_PATH }}/kubectl config set-context default \
      --cluster={{ KUBERNETS_CLUSTER_NAME }} \
      --user=system:kube-scheduler \
      --kubeconfig={{ KUBERNETES_DIRECTORY }}/kubeconfig/kube-scheduler.kubeconfig \
        
      {{ KUBERNETES_BINARY_PATH }}/kubectl config use-context default --kubeconfig={{ KUBERNETES_DIRECTORY }}/kubeconfig/kube-scheduler.kubeconfig 
    mode: 0700 

- name: Generate a kubeconfig file for the kube-scheduler service (2/2)
  shell: "{{ KUBERNETES_DIRECTORY }}/genconfig-scheduler.sh"
  register: "scheduler_result" 

- debug:
    msg: 
      - "scheduler result :  {{ scheduler_result }} " 
    verbosity: 2 
```

* admin 를 위해 사용하는 kubeconfig 파일을 생성합니다.
```
---
# 1. The kubelet Kubernetes Configuration File
- name: Generate a kubeconfig file for the admin service (1/2)
#  shell: >
  copy:
    dest: "{{ KUBERNETES_DIRECTORY }}/genconfig-admin.sh"
    content: |
      #/usr/bin/env bash

      {{ KUBERNETES_BINARY_PATH }}/kubectl config set-cluster {{ KUBERNETS_CLUSTER_NAME }} \
      --certificate-authority={{ KUBERNETES_DIRECTORY }}/pki/ca/ca.pem \
      --embed-certs=true \
      --server=https://{{ KUBERNETES_PUBLIC_ADDRESS }}:6443 \
      --kubeconfig={{ KUBERNETES_DIRECTORY }}/kubeconfig/admin.kubeconfig \
        
      {{ KUBERNETES_BINARY_PATH }}/kubectl config set-credentials admin \
      --client-certificate={{ KUBERNETES_DIRECTORY }}/pki/admin/admin.pem \
      --client-key={{ KUBERNETES_DIRECTORY }}/pki/admin/admin-key.pem \
      --embed-certs=true \
      --kubeconfig={{ KUBERNETES_DIRECTORY }}/kubeconfig/admin.kubeconfig
        
      {{ KUBERNETES_BINARY_PATH }}/kubectl config set-context default \
      --cluster={{ KUBERNETS_CLUSTER_NAME }} \
      --user=admin \
      --kubeconfig={{ KUBERNETES_DIRECTORY }}/kubeconfig/admin.kubeconfig \
        
      {{ KUBERNETES_BINARY_PATH }}/kubectl config use-context default --kubeconfig={{ KUBERNETES_DIRECTORY }}/kubeconfig/admin.kubeconfig 
    mode: 0700 

- name: Generate a kubeconfig file for the admin service (2/2)
  shell: "{{ KUBERNETES_DIRECTORY }}/genconfig-admin.sh"
  register: "admin_result" 

- debug:
    msg: 
      - "admin result :  {{ admin_result }} " 
    verbosity: 2 
```