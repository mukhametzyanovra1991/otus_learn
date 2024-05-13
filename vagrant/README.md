# configure workstation centos 9
# install VirtualBox
# create git repository mukhametzyanovra1991/otus_learn
# create Vagrantfile
# create VM with Vagrantfile
# commit and push files to repository


Занятие 1. Vagrant-стенд для обновления ядра и создания образа системы
Цель домашнего задания
Научиться обновлять ядро в ОС Linux. Получение навыков работы с Vagrant. 
Описание домашнего задания
1) Запустить ВМ с помощью Vagrant.
2) Обновить ядро ОС из репозитория ELRepo.
3) Оформить отчет в README-файле в GitHub-репозитории

connect to vagrant centos7 VM
	vagrant ssh
check kernel version
	[vagrant@centos7 ~]$ uname -r
	3.10.0-1160.105.1.el7.x86_64
connect repo 
	sudo yum install https://www.elrepo.org/elrepo-release-7.el7.elrepo.noarch.rpm
install latest kernel
	sudo yum --enablerepo elrepo-kernel install kernel-ml -y
update boot config
	sudo grub2-mkconfig -o /boot/grub2/grub.cfg
choose new kernel to boot
	sudo grub2-set-default 0
check kernel version
	[vagrant@centos7 ~]$ uname -r
	6.8.9-1.el7.elrepo.x86_64
