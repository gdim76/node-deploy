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

## 4. Локальное развёртывание узлов в docker контейнерах

__Перед началом локального развертывания необходимо выполнить пункты 1 и 2.__  
Для запуска локального развёртывания нужно перейти в директорию `node-deploy` и выполнить следующую команду:

```
ansible-playbook local-deploy.yml
```

По завершении плейбук развернёт сеть из 4 узлов. Апи будет доступно на портах с __8000__ по __8003__
Файл `node.log` для каждого узла хранится в директории `node-deploy/temp` с цифрой соответствующей номеру узла например `node-deploy/temp1`  
По завершении чтения параметров, узел начнёт писать логи в __stdout__ докер контейнера. Просмотреть их можно при помощи команды `docker logs` с названием нужного контейнера или при помощи графаны.
Веб интерфейс для просмотра логов Grafana будет доступен на __3000__ порту. 

Для получения доступа к логам необходимо в меню, находящемся слева, нажать на значок шестерни __Configuration__ затем нажать на кнопку __Add data source__, в появившемся меню пролистать немного вниз и выбрать __Loki__
Теперь перед вами страница настройки источника данных, вам необходимо найти раздел с заголовком __HTTP__ и ввести в поле __url__ следующий адрес `loki_loki_1:3100` остальные поля менять не обязательно. По завершении настройки источника данных необходимо нажать кнопку __Save & Test__ при успешном добавлении источника вы увидите сообщение __Data source connected and labels found__. Теперь можно отправлять запросы через __Explore__, он находится в меню слева и выглядит как компас. Нажмите на компас, на появившейся странице сверху рядом с надписью __Explore__ в выпадающем меню выберите __Loki__ и начните вводить `{` в поле __Log browser__. Вы увидите подсказку и сможете выбрать и посмотреть доступные поля, выберите из списка __job__ появится новая подсказка в ней выберите __node1-node__ Справа сверху нажмите кнопку __Run query__ вам отобразятся логи узла под номером 1.

Команда для остановки и удаления контейнеров:

```
ansible-playbook local-deploy.yml --tags down
```
Существует несколько дополнительных тегов.
- up - создаёт и запускает контейнеры с образов.
- rebuild - пересобирает узел и innochain-cli, запускает контейнеры.
- stop - останавливает все докер контейнеры.
- start - запускает все докер контейнеры, после их остановки.
- clean - удаляет все директории temp
- reload_api - заново копирует апи, пересоздаёт `node.log` файлы и запускает контейнеры с образов

## 5. Глобальное развёртывание узлов в docker контейнерах

По завершении плейбук развернёт сеть из 4 узлов на машинах из группы managed_nodes. Апи будет доступно на __8000__ порте.
Файл `node.log` для каждого узла хранится в директории `~/node/node-ml/node.log` и содержит в себе лог чтения параметров командной строки.
По завершении чтения параметров, узел начнёт писать логи в __stdout__ докер контейнера. Просмотреть их можно при помощи команды `docker logs` с названием нужного контейнера или при помощи графаны.
Веб интерфейс для просмотра логов Grafana будет доступен на __3000__ порту, у каждого узла свой контейнер с графаной. 

Для запуска ансибл плейбука нужно перейти в директорию __node-deploy__ и выполнить следующую команду:

```
ansible-playbook docker-deploy.yml --tags initial_run
```

Для получения доступа к логам необходимо в меню, находящемся слева, нажать на значок шестерни Configuration затем нажать на кнопку __Add data source__, в появившемся меню пролистать немного вниз и выбрать __Loki__
Теперь перед вами страница настройки источника данных, вам необходимо найти раздел с заголовком __HTTP__ и ввести в поле __url__ следующий адрес `node_loki_1:3100` остальные поля менять не обязательно. По завершении настройки источника данных необходимо нажать кнопку __Save & Test__ при успешном добавлении источника вы увидите сообщение __Data source connected and labels found__. Теперь можно отправлять запросы через __Explore__, он находится в меню слева и выглядит как компас. Нажмите на компас, на появившейся странице сверху рядом с надписью __Explore__ в выпадающем меню выберите __Loki__ и начните вводить `{` в поле __Log browser__. Вы увидите подсказку и сможете выбрать и посмотреть доступные поля, выберите из списка __job__ появится новая подсказка в ней выберите __node1-node__ Справа сверху нажмите кнопку __Run query__ вам отобразятся логи узла под номером 1.

Команда для остановки и удаления контейнеров:

```
ansible-playbook docker-deploy.yml --tags down
```
Существует несколько дополнительных тегов.
- up - создаёт и запускает контейнеры с образов.
- rebuild - пересобирает узел и innochain-cli, запускает контейнеры.
- stop - останавливает все докер контейнеры.
- start - запускает все докер контейнеры, после их остановки.
- reload_api - заново копирует апи, пересоздаёт `node.log` файлы и запускает контейнеры с образов.

## 6. Развёртывание одного узла

По завершении плейбук развернёт сеть из одного узла локально. Апи будет доступно на __8000__ порте.
Файл `node.log` для каждого узла хранится в директории `~/node/node.log` и содержит в себе лог чтения параметров командной строки.
По завершении чтения параметров, узел начнёт писать логи в __stdout__ докер контейнера. Просмотреть их можно при помощи команды `docker logs` с названием нужного контейнера или при помощи графаны.
Веб интерфейс для просмотра логов Grafana будет доступен на порте __3000__. 

Для запуска ансибл плейбука нужно перейти в директорию __node-deploy__ и выполнить следующую команду:

```
ansible-playbook deploy-single-node.yml --tags initial_run
``` 

Для получения доступа к логам необходимо в меню, находящемся слева, нажать на значок шестерни __Configuration__ затем нажать на кнопку __Add data source__, в появившемся меню пролистать немного вниз и выбрать __Loki__
Теперь перед вами страница настройки источника данных, вам необходимо найти раздел с заголовком __HTTP__ и ввести в поле __url__ следующий адрес `node_loki_1:3100` остальные поля менять не обязательно. По завершении настройки источника данных необходимо нажать кнопку __Save & Test__ при успешном добавлении источника вы увидите сообщение __Data source connected and labels found__. Теперь можно отправлять запросы через __Explore__, он находится в меню слева и выглядит как компас. Нажмите на компас, на появившейся странице сверху рядом с надписью __Explore__ в выпадающем меню выберите __Loki__ и начните вводить `{` в поле __Log browser__. Вы увидите подсказку и сможете выбрать и посмотреть доступные поля, выберите из списка __job__ появится новая подсказка в ней выберите __node__ Справа сверху нажмите кнопку __Run query__ вам отобразятся логи узла.

Команда для остановки и удаления контейнеров:

```
ansible-playbook deploy-single-node.yml --tags down
```
Существует несколько дополнительных тегов.
- up - создаёт и запускает контейнеры с образов.
- rebuild - пересобирает узел и innochain-cli, запускает контейнеры.
- stop - останавливает все докер контейнеры.
- start - запускает все докер контейнеры, после их остановки.

Для плейбука можно указать названия образов при помощи переменных __node_image__ и __api_image__.
Пример:
```
ansible-playbook deploy-single-node.yml --tags up -e "node_image=node:test api_image=api:test" 
```


## 7. Сборка образов api и node

По завершении будут созданы образы __api:latest__ и __node:latest__ с версией узла и апи, полученными после пункта 2.

__Перед сборкой образов необходимо выполнить пункты 1 и 2__.
Для запуска сборки образов нужно перейти в директорию __node-deploy__ и выполнить следующую команду:

```
ansible-playbook generate-image.yml
``` 

Существует ряд параметров в виде переменных.
- node_name - значение этой переменной задаёт имя образа узла.
- api_name - значение этой переменной задаёт имя образа api.
- tag - значение этой переменной задаёт тег для образов.
- mode - используется в случае необходимости создать только один образ. Может принимать значения api node и all.

Пример команды с параметрами:
```
ansible-playbook generate-image.yml -e "mode=node node_name=some-name tag=some-tag" 
```

После генерации образа можно приступить к развёртыванию контейнеров. 
Для этого необходимо при помощи `docker volumes` передать контейнеру параметры во время запуска.
Примеры дальше будут приведены для __docker-compose__ файла.

Для запуска контейнера узла необходимо передать реквизиты сети и пайпы. Так же можно при необходимости переписать команду запуска узла.
```
volumes:
- '/home/user/node/node-ml/apiin:/root/node/node-ml/apiin'
- '/home/user/node/node-ml/apiout:/root/node/node-ml/apiout'
- '/home/user/node/node-ml/broadcast:/root/node/node-ml/broadcast'
- '/home/user/node/node-ml/testnet:/root/node/node-ml/testnet'
```
Слева от двоеточия путь к файлу на хосте, а справа путь к файлу в контейнере. Файлы __apiin__, __apiout__ и __broadcast__ - это пайпы необходимые для работы с апи, а директория `testnet` содержит сгенерированные реквизиты сети.
При помощи __entrypoint__ можно переписать команду запуска узла указав свои реквизиты сети. Все пути прописаны относительно директории `/root/node/node-ml` в контейнере.
```
entrypoint: ./Main.cake --fgenesis=testnet/genesis --fkeypair=testnet/keypair --fnodesinfo=testnet/nodes --max-block-size=10 --apiin=apiin --apiout=apiout --broadcast=broadcast --init-delay=2.5 --base-exp=1.5 --logger-level=Debug --stdout-log --sync-commits-size=150 --sync-blocks-size=200
```
Для апи необходимо передать файл настроек и пайпы. 
```
volumes:
- '/home/user/node/node-ml/apiin:/root/node/node-ml/apiin'
- '/home/user/node/node-ml/apiout:/root/node/node-ml/apiout'
- '/home/user/node/node-ml/broadcast:/root/node/node-ml/broadcast'
- '/home/user/node/api-settings.py:/root/node/node-ml-api/api/settings.py'
```
Название файла на хосте не обязательно должно совпадать с названием в контейнере. Файл настроек апи в контейнере должен называться __settings.py__
Пример файла настроек для развёртывания одного узла можно просмотреть в контейнере, если запустить его без монтирования собственного файла, он будет находится там, куда монтируется в примере `/root/node/node-ml-api/api/settings.py`
Для того, чтобы иметь доступ к апи с хоста, нужно привязать __8000__ порт контейнера к порту хоста.
```
ports:
  - "8000:8000"
```
Значение слева от : это порт хоста.

Для работы бродкаста нужен третий контейнер __redis:5__ в нем не нужно производить дополнительных настроек, но необходимо указать его адрес или докер домен в настройках апи.

# Взаимодействие с АПИ

Для проверки апи можно воспользоваться транзакцией на добавление аккаунта пользователя.
Сгенерируйте транзакцию при помощи __innochain-cli__ внутри контейнера узла.

```
docker exec -w /root/node/node-ml/innochain-cli node_node_1 ./innochain-cli -u
```
где:
- `/root/node/node-ml/innochain-cli` - путь до __innochain-cli__ внутри контейнера
- node_node_1 - название докер контейнера с кодом узла (может меняться в зависимости от плейбука)
- ./innochain-cli -u - вызов __innochain-cli__ с параметром для генерации транзакции

Для отправки запроса апи нужно скопировать полученный вывод вместо транзакции в примере и выполнить команду: 
```
curl -X POST 'http://127.0.0.1:8000' --data '{"jsonrpc":"2.0","method":"submitRawTransaction","params":{"tx":"0x0f2fcfd2df2ad205ad0dfb74ee9561141f527b8d9b1d28fb00020000000001a501043066301f06082a85030701010101301306072a85030202230106082a85030701010202034300044076ab3562e6ab6dfc668f70802d24d18866cbd7210c5f75126ddda225e60220b327f70d5581c80e0ae4b7d9fc1d15d1722613663adff73d82047b704b751e86f8046fba8dbefb0df898df9026529c9356fad1fde1687e547fedb8c2b388ed8f0eb4a1aabbf6fe03d19c97f5f5f7dd6a6c3fdf53ed5a6c61870f5bb0fb438fa29cc3", "id": 1},"id":1}'
```

# Справочная информация

- Для плейбука docker-deploy.yml
  - api - 8000
  - grafana - 3000
  - logs - ~/node/node-ml/node.log
  - loki - node_loki_1:3100

- Для плейбука local-deploy.yml
  - api - 8000 8001 8002 8003
  - logs - ~/node-ml-deploy/temp1 ~/node-ml-deploy/temp2 ~/node-ml-deploy/temp3 ~/node-ml-deploy/temp4
  - grafana - 3000
  - loki - loki_loki_1:3100

- Для плейбука deploy-single-node.yml
  - api - 8000
  - logs - ~/node/node.log
  - grafana - 3000
  - loki - node_loki_1:3100