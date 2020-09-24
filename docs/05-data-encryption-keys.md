# Generating the Data Encryption Config and Key

Kubernetes 에서는 클러스터 데이터를 암호화를 제공하고 있습니다. 클러스터 데이터를 암호화를 위한 키를 생성합니다.

```
---
# tasks file for generating-secrets

- name: Generate an encryption key
  shell: "head -c 32 /dev/urandom | base64"
  register: ENCRYPTION_KEY
  when: "'{{ groups['kube-master'][0] }}' == '{{ inventory_hostname }}'"

- debug:
    msg: 
      - "DEBUG RESULT : {{ ENCRYPTION_KEY.stdout }}" 
    verbosity: 0

- name: Create the encryption-config.yaml encryption config file
  template:
    src: "encryption-config.yaml.j2"    
    dest: "{{ KUBERNETES_DIRECTORY }}/secret/encryption-config.yaml"    
```
* SHELL Commands 
```
ENCRYPTION_KEY=$(head -c 32 /dev/urandom | base64)

cat > encryption-config.yaml <<EOF
kind: EncryptionConfig
apiVersion: v1
resources:
  - resources:
      - secrets
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: ${ENCRYPTION_KEY}
      - identity: {}
EOF
```