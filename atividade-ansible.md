# Atividades

Instalar as coisas
```
apk add ansible nano
```

Executar em cada cliente
```
ssh-copy-id 192.168.0.13
```

## Atividade 1

Criar inventory
```
nano inventory
[maquinas]
192.168.0.12
192.168.0.11
192.168.0.10
192.168.0.9
```

nano atividade1.yml
```yml
---
- name: Atualizar cache
  hosts: maquinas
 
  tasks:
    - apk: update_cache=yes
      when: ansible_os_family == "Alpine"
   
    - apt: update_cache=yes
      when: ansible_os_family == "Debian"
```

Executar: `ansible-playbook -i inventory atividade1.yml`

## Atividade 2

Add no inventory
```
nano inventory
[ubuntu]
192.168.0.12
192.168.0.11
192.168.0.10

[webserver]
192.168.0.9
```

nano atividade2.yml
```yml
---
- hosts: ubuntu
  tasks:
    - name: cont
      community.general.docker_container:
        name: ubuntu-container
        image: ubuntu:20.04
        command: sleep infinity

- hosts: webserver
  tasks:
    - name: nginx
      community.general.docker_container:
        name: nginx-server
        image: nginx:latest
        state: started
        ports:
          - 8080:80
```

Executar
```sh
ansible-galaxy collection install community.general
ansible-playbook -i inventory atividade2.yml
```

### Atividade 3

nano atividade3.yml
```yml
---
- name: Copiar inventory (copiar para as máquinas ubuntu)
  hosts: ubuntu

  tasks:
    - name: Copiar arquivo do controlador para as máquinas ubuntu
      copy:
        src: /root/inventory
        dest: /root/inventory
        remote_src: no

    - name: Copiar arquivo das máquinas ubuntu para outra pasta nas mesmas máquinas
      copy:
        src: /root/inventory
        dest: /tmp/inventory
        remote_src: yes
```

Execute `ansible-playbook -i inventory atividade3.yml`

### Atividade 4


nano atividade4.yml
```yml
---
- name: Create a file and directory
  hosts: maquinas

  tasks:
    - name: Criar arquivo
      file:
        path: /root/arquivo-atividade-4.txt
        state: touch
        mode: '0644'

    - name: Criar pasta
      file:
        path: /root/atividade-4
        state: directory
        mode: '0755'
```

Execute `ansible-playbook -i inventory atividade4.yml`

### Atividade 5

```
nano inventory5
[nginx]
192.168.0.12
192.168.0.11
192.168.0.10

[banco]
192.168.0.9
```

nano atividade5.yml
```yml
---
- name: docker
  hosts: banco

  tasks:
    - name: postgres databse
      community.general.docker_container:
        name: postgres-container
        image: "postgres:latest"
        state: started


- name: docker nginx
  hosts: nginx

  tasks:
    - name: nginx app
      community.general.docker_container:
        name: nginx-container
        image: nginx:latest
        state: started
        ports:
          - 8081:80


- name: Listar containers
  hosts:
    - banco
    - nginx

  tasks:
    - name: list
      shell: docker ps
      register: out

    - debug: var=out.stdout_lines
```

Execute `ansible-playbook -i inventory5 atividade5.yml`

### Atividade 6

nano atividade6.yml
```yml
---
- name: Ips com command
  hosts: maquinas

  tasks:
    - command: ifconfig
      register: out

    - debug:
        msg: "{{ out.stdout_lines[1] }}"

- name: Ips com facts
  hosts: maquinas

  tasks:
    - debug:
        msg: "{{ ansible_all_ipv4_addresses }}"
```

Execute `ansible-playbook -i inventory atividade6.yml`
