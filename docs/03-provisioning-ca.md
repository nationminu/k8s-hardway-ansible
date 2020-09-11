# Provisioning CA

## set environment variables for shell command
```
export TLS_CN="Kubernetes"
export TLS_C="KR"
export TLS_L="SEOUL"
export TLS_OU="rockPLACE PST"
export TLS_O="rockPLACE"
export TLS_ST="SEOUL"
```

## Provisioning CA
```
- hosts: kube-master[0]
  any_errors_fatal: "{{ any_errors_fatal | default(true) }}"
  gather_facts: False
  roles: 
    - { role: provisioning-ca , tags: ["ca", "admin", "client", "controller", "proxy", "scheduler", "api" , "service-account"]}
```    

### ansible tasks
``` 
- name: Create a directory if it does not exist for pki
  file:
    path: "{{ KUBERNETES_DIRECTORY }}/{{ item }}"
    state: directory
    mode: '0755'
  with_items:
    - "pki/config"
    - "pki/admin"
    - "pki/api"
    - "pki/ca"
    - "pki/clients"
    - "pki/controller"
    - "pki/proxy"
    - "pki/scheduler"
    - "pki/service-account"
    - "config"
    - "kubeconfig"
    - "secret"
```

### 1. Generate the CA configuration file, certificate, and private key
```
# Generate the CA configuration file:
- name: Generate the CA configuration file
  copy:
    src: "ca-config.json"
    dest: "{{ KUBERNETES_DIRECTORY }}/pki/config/ca-config.json"   
  tags:
    - always
```

* ca-config.json
```
{
    "signing": {
        "default": {
        "expiry": "8760h"
        },
        "profiles": {
        "kubernetes": {
            "usages": ["signing", "key encipherment", "server auth", "client auth"],
            "expiry": "8760h"
        }
        }
    }
}

# SHELL Command
cat > ca-config.json <<EOF
{
  "signing": {
    "default": {
      "expiry": "8760h"
    },
    "profiles": {
      "kubernetes": {
        "usages": ["signing", "key encipherment", "server auth", "client auth"],
        "expiry": "8760h"
      }
    }
  }
}
EOF
```

### 2. Generate the CA certificate and private key
```
# Generate the CA configuration file, certificate, and private key: 
- name: Generate the CA certificate and private key (1/2)
  template:
    src: "ca-csr.json.j2"
    dest: "{{ KUBERNETES_DIRECTORY }}/pki/config/ca-csr.json"    

## json pretty format : cat ~/pki/xx.json | python -m json.tool
- name: Generate the CA certificate and private key (2/2) 
  copy:
    dest: "{{ KUBERNETES_DIRECTORY }}/gencert-ca.sh"
    content: |
      #/usr/bin/env bash
      {{ KUBERNETES_BINARY_PATH }}/cfssl gencert -initca {{ KUBERNETES_DIRECTORY }}/pki/config/ca-csr.json \
      | {{ KUBERNETES_BINARY_PATH }}/cfssljson -bare {{ KUBERNETES_DIRECTORY }}/pki/ca/ca
    mode: 0700

- name: Certificate Authority
  shell: "{{ KUBERNETES_DIRECTORY }}/gencert-ca.sh"
  register: ca_result 

- debug:
    msg: 
      - "ca result : {{ ca_result }}" 
    verbosity: 2 
```    

* ca-csr.json
```
{
    "CN": "{{ TLS_CN }}",
    "key": 
    {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "{{ TLS_C }}",
            "L": "{{ TLS_L }}",
            "O": "{{ TLS_O }}",
            "OU": "{{ TLS_OU }}",
            "ST": "{{ TLS_ST }}"
        }
    ]
}

# SHELL Commands
cat > ca-csr.json <<EOF
{
  "CN": "${TLS_CN}",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "${TLS_C}",
      "L": "${TLS_L}",
      "O": "${TLS_O}",
      "OU": "${TLS_OU}",
      "ST": "${TLS_ST}"
    }
  ]
}
EOF

cfssl gencert -initca ca-csr.json | cfssljson -bare ca
```

### 3. Generate the admin certificate and private key
```
---
# The Admin Certificate  
- name: Generate the admin certificate and private key (1/2)
  template:
    src: "admin-csr.json.j2"
    dest: "{{ KUBERNETES_DIRECTORY }}/pki/admin/admin-csr.json"    

## json pretty format : cat ~/pki/xx.json | python -m json.tool
- name: Generate the admin certificate and private key (2/2)
  copy:
    dest: "{{ KUBERNETES_DIRECTORY }}/gencert-admin.sh"
    content: |
      #/usr/bin/env bash
      {{ KUBERNETES_BINARY_PATH }}/cfssl gencert \
      -ca={{ KUBERNETES_DIRECTORY }}/pki/ca/ca.pem \
      -ca-key={{ KUBERNETES_DIRECTORY }}/pki/ca/ca-key.pem \
      -config={{ KUBERNETES_DIRECTORY }}/pki/config/ca-config.json \
      -profile=kubernetes \
      {{ KUBERNETES_DIRECTORY }}/pki/admin/admin-csr.json \
      | {{ KUBERNETES_BINARY_PATH }}/cfssljson -bare {{ KUBERNETES_DIRECTORY }}/pki/admin/admin
    mode: 0700

- name: Certificate Authority
  shell: "{{ KUBERNETES_DIRECTORY }}/gencert-admin.sh"
  register: admin_result 

- debug:
    msg: 
      - "admin result : {{ admin_result }}" 
    verbosity: 2 
```

* admin-csr.json
```
{
    "CN": "admin",
    "key": 
    {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "{{ TLS_C }}",
            "L": "{{ TLS_L }}",
            "O": "system:masters",
            "OU": "{{ TLS_OU }}",
            "ST": "{{ TLS_ST }}"
        }
    ]
}

# SHELL Commands
cat > admin-csr.json <<EOF
{
  "CN": "admin",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "${TLS_C}",
      "L": "${TLS_L}",
      "O": "system:masters",
      "OU": "${TLS_OU}",
      "ST": "${TLS_ST}"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  admin-csr.json | cfssljson -bare admin
```

### 4. Generate a certificate and private key for each Kubernetes worker node
```
---
# The Admin Certificate  
- name: Generate a certificate and private key for each Kubernetes worker node (1/2)
  template:
    src: "node-csr.json.j2"
    dest: "{{ KUBERNETES_DIRECTORY }}/pki/clients/{{ item }}-csr.json"  
  with_items:
    - "{{ groups['kube-node'] }}"  

## json pretty format : cat ~/pki/xx.json | python -m json.tool
- name: Generate a certificate and private key for each Kubernetes worker node (2/2)
  copy:
    dest: "{{ KUBERNETES_DIRECTORY }}/gencert-{{ item }}.sh"
    content: |
      #/usr/bin/env bash

      {{ KUBERNETES_BINARY_PATH }}/cfssl gencert \
      -ca={{ KUBERNETES_DIRECTORY }}/pki/ca/ca.pem \
      -ca-key={{ KUBERNETES_DIRECTORY }}/pki/ca/ca-key.pem \
      -config={{ KUBERNETES_DIRECTORY }}/pki/config/ca-config.json \
      -hostname={{ item }},{{ item }}.{{ KUBERNETES_FQDN }},{{ hostvars[item].ansible_host }} \
      -profile=kubernetes \
      {{ KUBERNETES_DIRECTORY }}/pki/clients/{{ item }}-csr.json \
      | /usr/local/bin/cfssljson -bare {{ KUBERNETES_DIRECTORY }}/pki/clients/{{ item }}
    mode: 0700
  with_items:
    - "{{ groups['kube-node'] }}"

- name: Certificate Authority
  shell: "{{ KUBERNETES_DIRECTORY }}/gencert-{{ item }}.sh"
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

* node-csr.json.j2
```
{
    "CN": "system:node:{{ item }}",
    "key": 
    {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "{{ TLS_C }}",
            "L": "{{ TLS_L }}",
            "O": "system:nodes",
            "OU": "{{ TLS_OU }}",
            "ST": "{{ TLS_ST }}"
        }
    ]
}

for instance in $(head -n 6 inventory.ini | awk '{gsub("ansible_host=","",$2); print $1 "," $2}'); do

IFS=',' read -ra array <<< "$instance"
NODE_NAME="${array[0]}"
INTERNAL_IP="${array[1]}"

cat > ${NODE_NAME}-csr.json <<EOF
{
  "CN": "system:node:${NODE_NAME}",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    { 
      "C": "${TLS_C}",
      "L": "${TLS_L}",
      "O": "system:nodes",
      "OU": "${TLS_OU}",
      "ST": "${TLS_ST}"      
    }
  ]
}
EOF 

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -hostname=${NODE_NAME},${INTERNAL_IP} \
  -profile=kubernetes \
  ${NODE_NAME}-csr.json | cfssljson -bare ${NODE_NAME}
done

```

### 5. Generate the kube-controller-manager client certificate and private key
```
---
# The Admin Certificate  
- name: Generate the kube-controller-manager client certificate and private key (1/2)
  template:
    src: "controller-csr.json.j2"
    dest: "{{ KUBERNETES_DIRECTORY }}/pki/controller/kube-controller-manager-csr.json"    

## json pretty format : cat ~/pki/xx.json | python -m json.tool
- name: Generate the kube-controller-manager client certificate and private key (2/2)
  copy:
    dest: "{{ KUBERNETES_DIRECTORY }}/gencert-controller.sh"
    content: |
      #/usr/bin/env bash
      {{ KUBERNETES_BINARY_PATH }}/cfssl gencert \
      -ca={{ KUBERNETES_DIRECTORY }}/pki/ca/ca.pem \
      -ca-key={{ KUBERNETES_DIRECTORY }}/pki/ca/ca-key.pem \
      -config={{ KUBERNETES_DIRECTORY }}/pki/config/ca-config.json \
      -profile=kubernetes \
      {{ KUBERNETES_DIRECTORY }}/pki/controller/kube-controller-manager-csr.json  \
      | {{ KUBERNETES_BINARY_PATH }}/cfssljson -bare {{ KUBERNETES_DIRECTORY }}/pki/controller/kube-controller-manager
    mode: 0700

- name: Certificate Authority
  shell: "{{ KUBERNETES_DIRECTORY }}/gencert-controller.sh"
  register: controller_result 

- debug:
    msg: 
      - "controller result : {{ controller_result }}" 
    verbosity: 2 
```

* controller-csr.json.j2
```
{
    "CN": "system:kube-controller-manager",
    "key": 
    {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "{{ TLS_C }}",
            "L": "{{ TLS_L }}",
            "O": "system:kube-controller-manager",
            "OU": "{{ TLS_OU }}",
            "ST": "{{ TLS_ST }}"
        }
    ]
}

# SHELL Commands
cat > kube-controller-manager-csr.json <<EOF
{
  "CN": "system:kube-controller-manager",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    { 
      "C": "${TLS_C}",
      "L": "${TLS_L}",
      "O": "system:kube-controller-manager",
      "OU": "${TLS_OU}",
      "ST": "${TLS_ST}"         
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  kube-controller-manager-csr.json | cfssljson -bare kube-controller-manager
```

### 6. Generate the kube-proxy client certificate and private key
```
---
# The Admin Certificate  
- name: Generate the kube-proxy client certificate and private key (1/2)
  template:
    src: "proxy-csr.json.j2"
    dest: "{{ KUBERNETES_DIRECTORY }}/pki/proxy/kube-proxy-csr.json"    

## json pretty format : cat ~/pki/xx.json | python -m json.tool
- name: Generate the kube-proxy client certificate and private key (2/2) 
  copy:
    dest: "{{ KUBERNETES_DIRECTORY }}/gencert-proxy.sh"
    content: |
      #/usr/bin/env bash
      {{ KUBERNETES_BINARY_PATH }}/cfssl gencert \
      -ca={{ KUBERNETES_DIRECTORY }}/pki/ca/ca.pem \
      -ca-key={{ KUBERNETES_DIRECTORY }}/pki/ca/ca-key.pem \
      -config={{ KUBERNETES_DIRECTORY }}/pki/config/ca-config.json \
      -profile=kubernetes \
      {{ KUBERNETES_DIRECTORY }}/pki/proxy/kube-proxy-csr.json  \
      | {{ KUBERNETES_BINARY_PATH }}/cfssljson -bare {{ KUBERNETES_DIRECTORY }}/pki/proxy/kube-proxy
    mode: 0700

- name: Certificate Authority
  shell: "{{ KUBERNETES_DIRECTORY }}/gencert-proxy.sh"
  register: proxy_result 

- debug:
    msg: 
      - "proxy result : {{ proxy_result }}" 
    verbosity: 2 
```

* proxy-csr.json.j2
```
{
    "CN": "system:kube-proxy",
    "key": 
    {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "{{ TLS_C }}",
            "L": "{{ TLS_L }}",
            "O": "system:node-proxier", 
            "OU": "{{ TLS_OU }}",
            "ST": "{{ TLS_ST }}"
        }
    ]
}

# SHELL Commands
cat > kube-proxy-csr.json <<EOF
{
  "CN": "system:kube-proxy",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "${TLS_C}",
      "L": "${TLS_L}",
      "O": "system:node-proxier",
      "OU": "${TLS_OU}",
      "ST": "${TLS_ST}"            
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  kube-proxy-csr.json | cfssljson -bare kube-proxy
```

### 7. Generate the kube-scheduler client certificate and private key
```
---
# The Admin Certificate  
- name: Generate the kube-scheduler client certificate and private key (1/2)
  template:
    src: "scheduler-csr.json.j2"
    dest: "{{ KUBERNETES_DIRECTORY }}/pki/scheduler/kube-scheduler-csr.json"    

## json pretty format : cat ~/pki/xx.json | python -m json.tool
- name: Generate the kube-scheduler client certificate and private key (2/2) 
  copy:
    dest: "{{ KUBERNETES_DIRECTORY }}/gencert-scheduler.sh"
    content: |
      #/usr/bin/env bash
      {{ KUBERNETES_BINARY_PATH }}/cfssl gencert \
      -ca={{ KUBERNETES_DIRECTORY }}/pki/ca/ca.pem \
      -ca-key={{ KUBERNETES_DIRECTORY }}/pki/ca/ca-key.pem \
      -config={{ KUBERNETES_DIRECTORY }}/pki/config/ca-config.json \
      -profile=kubernetes \
      {{ KUBERNETES_DIRECTORY }}/pki/scheduler/kube-scheduler-csr.json  \
      | {{ KUBERNETES_BINARY_PATH }}/cfssljson -bare {{ KUBERNETES_DIRECTORY }}/pki/scheduler/kube-scheduler
    mode: 0700

- name: Certificate Authority
  shell: "{{ KUBERNETES_DIRECTORY }}/gencert-scheduler.sh"
  register: scheduler_result 

- debug:
    msg: 
      - "scheduler result : {{ scheduler_result }}" 
    verbosity: 2 
```

* scheduler.json.j2
```
{
    "CN": "system:kube-scheduler",
    "key": 
    {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "{{ TLS_C }}",
            "L": "{{ TLS_L }}",
            "O": "system:kube-scheduler",
            "OU": "{{ TLS_OU }}",
            "ST": "{{ TLS_ST }}"
        }
    ]
}

# SHELL Commands
cat > kube-scheduler-csr.json <<EOF
{
  "CN": "system:kube-scheduler",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {D
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  kube-scheduler-csr.json | cfssljson -bare kube-scheduler
```

### 8. Generate the Kubernetes API Server certificate and private key
```
```

* kubernetes-csr.json.j2
```
{
    "CN": "{{ TLS_CN }}",
    "key": 
    {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "{{ TLS_C }}",
            "L": "{{ TLS_L }}",
            "O": "{{ TLS_O }}",
            "OU": "{{ TLS_OU }}",
            "ST": "{{ TLS_ST }}"
        }
    ]
}

# SHELL Commands

export KUBERNETES_PUBLIC_ADDRESS=10.65.40.10
export MASTERS_ADDRESS=10.65.40.11,10.65.40.12,10.65.40.13
export KUBERNETES_HOSTNAMES=kubernetes,kubernetes.default,kubernetes.default.svc,kubernetes.default.svc.cluster,kubernetes.svc.cluster.local

cat > kubernetes-csr.json <<EOF
{
  "CN": "${TLS_CN}",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "${TLS_C}",
      "L": "${TLS_L}",
      "O": "${TLS_O}",
      "OU": "${TLS_OU}",
      "ST": "${TLS_ST}"      
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -hostname=10.32.0.1,${MASTERS_ADDRESS},${KUBERNETES_PUBLIC_ADDRESS},127.0.0.1,${KUBERNETES_HOSTNAMES} \
  -profile=kubernetes \
  kubernetes-csr.json | cfssljson -bare kubernetes
```

### 9. Generate the service-account certificate and private key
```
--- 
- name: Generate the service-account certificate and private key (1/2)
  template:
    src: "service-account-csr.json.j2"
    dest: "{{ KUBERNETES_DIRECTORY }}/pki/service-account/service-account-csr.json"    

## json pretty format : cat ~/pki/xx.json | python -m json.tool
- name: Generate the service-account certificate and private key (2/2)
  copy:
    dest: "{{ KUBERNETES_DIRECTORY }}/gencert-service-account.sh"
    content: |
      #/usr/bin/env bash
      {{ KUBERNETES_BINARY_PATH }}/cfssl gencert \
      -ca={{ KUBERNETES_DIRECTORY }}/pki/ca/ca.pem \
      -ca-key={{ KUBERNETES_DIRECTORY }}/pki/ca/ca-key.pem \
      -config={{ KUBERNETES_DIRECTORY }}/pki/config/ca-config.json \
      -profile=kubernetes \
      {{ KUBERNETES_DIRECTORY }}/pki/service-account/service-account-csr.json \
      | {{ KUBERNETES_BINARY_PATH }}/cfssljson -bare {{ KUBERNETES_DIRECTORY }}/pki/service-account/service-account
    mode: 0700

- name: Certificate Authority
  shell: "{{ KUBERNETES_DIRECTORY }}/gencert-service-account.sh"
  register: account_result 

- debug:
    msg: 
      - "service-account result : {{ account_result }}" 
    verbosity: 2 
```

* service-account-csr.json.j2
```
{
    "CN": "service-accounts",
    "key": 
    {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "{{ TLS_C }}",
            "L": "{{ TLS_L }}",
            "O": "{{ TLS_O }}",
            "OU": "{{ TLS_OU }}",
            "ST": "{{ TLS_ST }}"
        }
    ]
}

# SHELL Commands
cat > service-account-csr.json <<EOF
{
  "CN": "service-accounts",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "${TLS_C}",
      "L": "${TLS_L}",
      "O": "${TLS_O}",
      "OU": "${TLS_OU}",
      "ST": "${TLS_ST}"  
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  service-account-csr.json | cfssljson -bare service-account
```