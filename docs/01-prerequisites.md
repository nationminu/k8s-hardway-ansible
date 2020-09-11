# Prerequisites

## Hardway Tutorial Topology

![Image of Hardway](https://github.com/nationminu/k8s-hardway-ansible/blob/master/docs/images/hardway.png)

```
- hosts: all 
  any_errors_fatal: "{{ any_errors_fatal | default(true) }}"
  gather_facts: False  
  roles:
    - { role: prerequisites/ssh-config }
```

```
- hosts: jumpbox
  any_errors_fatal: "{{ any_errors_fatal | default(true) }}"
  gather_facts: False  
  roles:
    - { role: prerequisites/pre-config }
    - { role: install-tools/haproxy } 
```

```
- hosts: k8s-cluster
  any_errors_fatal: "{{ any_errors_fatal | default(true) }}"
  gather_facts: False  
  roles: 
    - { role: prerequisites/pre-config }
     
```