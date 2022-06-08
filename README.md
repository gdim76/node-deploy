# node-deploy

Ansible deploy script for Innochain node based on CakeML

## 1. Getting started

Перед началом работы необходимо установить:
[ansible 2.9.24](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#installing-ansible-on-specific-operating-systems)

```
$ sudo apt update
$ sudo apt install software-properties-common
$ sudo add-apt-repository --yes --update ppa:ansible/ansible
$ sudo apt install ansible
```

Затем нужно клонировать репозиторий node-deploy

```
git clone https://github.com/gdim76/node-deploy.git
```

Далее нужно внести изменения в файл hosts.
Группа `[managed_nodes]` состоит из описания машин на которых будут развёрнуты узлы. 
В файл необходим добавить IP адреса nodes.  
Для тестирования необходимо внести в файл connect_to_host.yml
список IP адресов nodes для распространния ssh ключей для подключения.

## 2. Подготовка Клонирование репозиториев node-ml и node-ml-api


## 3. Развёртывание узлов

Плейбук __deployment.yml__  подготовливает Nodes для развертывания.
hosts: all 
Role  Predeploy - создает папку в home директории и скачивает из репозиториев node-ml и node-ml-api 

git clone https://gitlab.com/InnoChain/node-ml.git
git clone https://gitlab.com/RIRussel/node-ml-api.git

Role Common и Gosts устанавливает необ

разворачивает узлы непосредственно на машинах.
По завершению развёртывания, узлы будут работать в виде сервисов с названием __ml-innochain.service__

Для запуска ансибл плейбука нужно перейти в директорию `node-deploy` 

```
ansible-playbook deployment.yml
```
Пример запуска плейбука с предоставлением файла хостов через флаг:

```
ansible-playbook deployment.yml -i hosts
```
Для остановки узлов на всех машинах можно воспользоваться плейбуком __ml-innochain-stop.yml__

```
ansible-playbook ml-innochain-stop.yml
```
Для последующих запусков можно использовать теги:  
__rebuild__ - полезен при изменении кода узла, с его помощью можно пересобрать и запустить узел.

```
ansible-playbook deployment.yml --tags rebuild
```
__restart__ - просто перезапускает узел.
```
ansible-playbook deployment.yml --tags restart
```
При необходимости отчистки базы данных можно воспользоваться флагом __clean_db__

```
ansible-playbook deployment.yml --tags restart -e clean_db=yes
```



