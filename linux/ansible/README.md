# Первые шаги с Ansible

## Цель домашнего задания

Написать первые шаги с Ansible.

## Описание домашнего задания

Подготовить стенд на Vagrant как минимум с одним сервером. На этом сервере, используя Ansible необходимо развернуть nginx со следующими условиями:

- необходимо использовать модуль yum/apt
- конфигурационный файлы должны быть взяты из шаблона jinja2 с переменными
- после установки nginx должен быть в режиме enabled в systemd
- должен быть использован notify для старта nginx после установки
- сайт должен слушать на нестандартном порту - 8080, для этого использовать переменные в Ansible

* Сделать все это с использованием Ansible роли

### install ansible on hypervisor  
  [rustem@centos-fileserver ansible]$ ansible --version
  ansible [core 2.14.14]  
  python version = 3.9.18 (main, Jan 24 2024, 00:00:00) [GCC 11.4.1 20231218 (Red Hat 11.4.1-3)] (/usr/bin/python3)
  jinja version = 3.1.2
  libyaml = True  

### create virtual machine nginx with Vagrantfile  
  [rustem@centos-fileserver ubuntu2204]$ vagrant status
  Current machine states:
  nginx                     running (virtualbox)

### create inventory file hosts  
ping virtual machine nginx  
  [rustem@centos-fileserver ansible]$ ansible nginx -i staging/hosts -m ping
  nginx | SUCCESS => {
      "ansible_facts": {
          "discovered_interpreter_python": "/usr/bin/python3"
      },
      "changed": false,
      "ping": "pong"
  }

### create ansible.cfg file where specified inventory file and ssh user  
### ping nginx host  
  [rustem@centos-fileserver ansible]$ ansible nginx -m ping
  nginx | SUCCESS => {
      "ansible_facts": {
          "discovered_interpreter_python": "/usr/bin/python3"
      },
      "changed": false,
      "ping": "pong"
  }

### execute ad-hoc command  
  [rustem@centos-fileserver ansible]$ ansible nginx -m command -a "uname -r"
  nginx | CHANGED | rc=0 >>
  5.15.0-91-generic  
  [rustem@centos-fileserver ansible]$ ansible nginx -m systemd -a name=firewalld
  nginx | SUCCESS => {
      "ansible_facts": {
          "discovered_interpreter_python": "/usr/bin/python3"
      },
      "changed": false,
      "name": "firewalld",
      "status": {

### create file nginx.yaml and check it on nginx host  
  [rustem@centos-fileserver ansible]$ ansible-playbook nginx.yaml --syntax-check

  playbook: nginx.yaml
  [rustem@centos-fileserver ansible]$ ansible-playbook nginx.yaml --check

  PLAY [NGINX | Install and config NGINX] ***

  TASK [Gathering Facts] ***
  ok: [nginx]

  TASK [update] ***
  changed: [nginx]

  TASK [NGINX | Install NGINX] ***
  changed: [nginx]

  PLAY RECAP ***
  nginx                      : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

### create template file nginx.conf.j2, add module for copy template to nginx.yaml; add var nginx_listen_port and tags to config

### add handler to nginx.yaml; add notify to install nginx and copy template module

### run playbook  
  [rustem@centos-fileserver ansible]$ ansible-playbook nginx.yaml

  PLAY [NGINX | Install and config NGINX] ***

  TASK [Gathering Facts] ***
  ok: [nginx]

  TASK [update] ***
  changed: [nginx]

  TASK [NGINX | Install NGINX] ***
  changed: [nginx]

  TASK [NGINX | Create NGINX config file from template] ***
  changed: [nginx]

  RUNNING HANDLER [restart nginx] ***
  changed: [nginx]

  RUNNING HANDLER [reload nginx] ***
  changed: [nginx]

  PLAY RECAP ***
  nginx                      : ok=6    changed=5    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

### check site availability  
nginx.png
