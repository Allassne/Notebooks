
Déploiement de multi-master k3s


ssh albam@10.1.16.2 -A
option a pour embarqué ma clé

Pour installer les masters
vim inventory/hosts
```
agents
servers

[servers]
albam-M1.dev01.abj.smile.lan
albam-M2.dev01.abj.smile.lan
albam-M3.dev01.abj.smile.lan

[agents]
```
vim playbooks/k3s.yml 
```
---

- hosts: all
  vars:
	k3s_servers: " groups['servers'] "
	k3s_agents: " groups['agents'] "
  roles:
	- role: k3s
```

ansible-playbook playbooks/k3s.yml -u albam -b 


Pour installer les workers
vim inventory/hosts
```
agents
servers

[servers]
albam-M1.dev01.abj.smile.lan
albam-M2.dev01.abj.smile.lan
albam-M3.dev01.abj.smile.lan


[agents]
albam-W1.dev01.abj.smile.lan
albam-W2.dev01.abj.smile.lan
albam-W3.dev01.abj.smile.lan

``` 
ansible-playbook playbooks/k3s.yml -u albam -b 
