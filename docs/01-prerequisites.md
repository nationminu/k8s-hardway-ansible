# Prerequisites

## Hardway System

본 튜토리얼은 master 3 대, worker 3 대로 구성합니다.

| Hostname      | CPU           | Memory  |
| ------------- |:-------------:| -------:|
| master1       | 2 core        |    4G   |
| master2       | 2 core        |    4G   |
| master3       | 2 core        |    4G   |
| worker1       | 2 core        |    4G   |
| worker2       | 2 core        |    4G   |
| worker3       | 2 core        |    4G   |

### Hardway Topology
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