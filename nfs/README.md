# Методическое пособие по выполнению домашнего задания по курсу "Администратор Linux. Professional"   
# Стенд Vagrant с NFS  

## Цель домашнего задания  
### Научиться самостоятельно разворачивать сервис NFS и подключать к нему клиентов.  

## Описание домашнего задания  
### Основная часть:  
- vagrant up должен поднимать 2 настроенных виртуальных машины (сервер NFS и клиента) без дополнительных ручных действий;  
- на сервере NFS должна быть подготовлена и экспортирована директория;   
- в экспортированной директории должна быть поддиректория с именем upload с правами на запись в неё;  
- экспортированная директория должна автоматически монтироваться на клиенте при старте виртуальной машины (systemd, autofs или fstab — любым способом);  
- монтирование и работа NFS на клиенте должна быть организована с использованием NFSv3.  
### Для самостоятельной реализации:   
- настроить аутентификацию через KERBEROS с использованием NFSv4.  

### Vagrantfile  
create Vagrantfile to up 2 virtual machines  
Vagrantfile generates inventory file for ansible in .vagrant/provisioners/ansible/inventory/vagrant_ansible_inventory  

### ansible-playbook  
with ansible playbook file install and configure nfs server and client  
ping-playbook.yaml for test connectivity