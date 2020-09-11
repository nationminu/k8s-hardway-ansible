# Install Tools

## Install cfssl 

```
- hosts: kube-master[0]
  any_errors_fatal: "{{ any_errors_fatal | default(true) }}"
  gather_facts: False
  roles:
    - { role: install-tools/cfssl } 

```
### ansible tasks
```
- name: Download and install cfssl and cfssljson
  get_url:
    url: "{{ item }}"
    dest: "{{ KUBERNETES_BINARY_PATH }}"
    mode: '0700'
  with_items:
    - "https://storage.googleapis.com/kubernetes-the-hard-way/cfssl/1.4.1/linux/cfssl"
    - "https://storage.googleapis.com/kubernetes-the-hard-way/cfssl/1.4.1/linux/cfssljson"

- name: Verify cfssl version 1.4.1 or higher is installed
  shell: "{{ KUBERNETES_BINARY_PATH }}/cfssl version"
  register: cfssl_version

- name: Verify cfssljson version 1.4.1 or higher is installed
  shell: "{{ KUBERNETES_BINARY_PATH }}/cfssljson --version"      
  register: cfssljson_version

- debug:
    msg: 
      - "cfssl version : {{ cfssl_version.stdout_lines[0] }}"
      - "cfssljson version : {{ cfssljson_version.stdout_lines[0] }}"
    verbosity: 2
```

## Install kubectl 

```
- hosts: kube-master[0]
  any_errors_fatal: "{{ any_errors_fatal | default(true) }}"
  gather_facts: False
  roles: 
    - { role: install-tools/kubernetes }

```
### ansible tasks
```
---
# tasks file for install-tools/kubernetes

# The kubectl command line utility is used to interact with the Kubernetes API Server. Download and install kubectl from the official release binaries:
- name: Download and install kubectl
  get_url:
    url: "{{ item }}"
    dest: "{{ KUBERNETES_BINARY_PATH }}"
    mode: '0700'
  with_items:
    - "https://storage.googleapis.com/kubernetes-release/release/{{ KUBERNETES_VERSION }}/bin/linux/amd64/kubectl" 

- name: Verify kubectl version 1.18.6 or higher is installed
  shell: "{{ KUBERNETES_BINARY_PATH }}/kubectl version --client"      
  register: kubectl_version        

- debug:
    msg: 
      - "kubectl version : {{ kubectl_version.stdout_lines[0] }}" 
    verbosity: 2      

- name: Enable kubectl autocompletion (1/2)
  yum: 
    name: bash-completion
    state: installed

- name: Enable kubectl autocompletion (2/2)
  shell: | 
    {{ KUBERNETES_BINARY_PATH }}/kubectl completion bash > /etc/bash_completion.d/kubectl  
```