# SMB
________________________
**Samba** — пакет программ, которые позволяют обращаться к сетевым дискам и принтерам на различных операционных системах по протоколу SMB/CIFS. Имеет клиентскую и серверную части. Является свободным программным обеспечением, выпущена под лицензией GPL.

Начиная с четвёртой версии, разработка которой велась почти 10 лет, Samba может выступать в роли контроллера домена и сервиса Active Directory, совместимого с реализацией Windows 2000, и способна обслуживать все поддерживаемые Microsoft версии Windows-клиентов, в том числе Windows 10.

Samba работает на большинстве Unix-подобных систем, таких как Linux, POSIX-совместимых Solaris и Mac OS X Server, на различных вариантах BSD; в OS/2 портирован Samba-клиент, являющийся плагином к виртуальной файловой системе NetDrive. Samba включена практически во все дистрибутивы Linux.

Настройка даты и времени
При первоначальной настройке файлового хранилища очень важно уделить внимание настройке даты и времени, чтобы избежать проблем в будущем при поиске необходимых файлов и директорий.

Для настройки даты и времени используется утилита chrony. Ее демона можно добавить в автозагрузку сервера и время всегда будет актуальным. Отправим в терминал команду установки пакета:
```
sudo apt install chrony -y
```

Перейдем непосредственно к установке Samba на Ubuntu. Отправим в терминал команду:
```    
sudo apt install samba -y
```

Добавим сервис smbd в автозапуск:

```
    
sudo systemctl enable smbd
```

Запустим:

```
    
sudo systemctl start smbd
```

Проверим текущий статус:

```
    
sudo systemctl status smbd
```

Далее перечислим оставшиеся команды управления сервисом smbd. Они могут потребоваться в дальнейшем в процессе эксплуатации Samba.

Остановить сервис:

```
    
sudo systemctl stop smbd
```

Перезапуск демона:

```
    
sudo systemctl restart smbd
```

Убрать из автозапуска:

```
    
sudo systemctl disable smbd
```

Перечитать конфигурацию:

```
    
sudo systemctl reload smbd
```

Настройка Samba
Конфигурационный файл Samba расположен по следующему пути:

```
/etc/samba/smb.conf
```
Лучше скопировать файл конфигурации по умолчанию, чтобы всегда оставлять себе возможность откатиться до дефолтных настроек Samba на Ubuntu. 

Копируем командой:

```
    
sudo cp /etc/samba/smb.conf /etc/samba/res_smb.conf
```

Далее оставим в файле конфигурации только те строки, которые используются в работе и не являются комментарием.

```
grep -v '^ *#\|^ *$' /etc/samba/smb.conf | sudo tee /etc/samba/smb.conf
```

На выходе получаем файл конфигурации без комментариев и неиспользуемых директив. 

________________________

**3. Настройте сервер домена на базе HQ-SRV через web интерфейс, выбор технологий обоснуйте**  
**a. Введите машины BR-SRV и CLI в данный домен**  
**b. Организуйте отслеживание подключения к домену**  

## **ISP**
Произведём настройку маршрута для CLI
```
ip route add default via 192.168.0.2
```
**Внимание!** рекомендую сделать snapshot перед началом действий, в случае если возникнут трудности, воизбежении потери времени!
**Обоснование**: Выбор Samba в качестве контроллера домена обеспечивает интеграцию Linux-систем в сети Windows, обеспечивает безопасное управление ресурсами в сети и снижает затраты на создание и поддержку контроллера домена. Также на базе samba в дальнейшем планируется развёртка сетевых ресурсов организации, что позволит в рамках одного программного решения реализовать несколько практических задач.

## **HQ-SRV**
Произведём временное отключение интерфейсов. Обязательно перед началом настройки samba! 
```
nmtui
```
![image](https://github.com/NyashMan/DEMO2024/assets/1348639/bd3570e1-b9be-4a5f-bf5a-887cf27100cd)  
![image](https://github.com/NyashMan/DEMO2024/assets/1348639/e807a623-e6be-4573-92b8-ca5a8227a0c2)  
```
control bind-chroot disabled
grep -q 'bind-dns' /etc/bind/named.conf || echo 'include "/var/lib/samba/bind-dns/named.conf";' >> /etc/bind/named.conf
nano /etc/bind/options.conf
```
```
tkey-gssapi-keytab "/var/lib/samba/bind-dns/dns.keytab";
minimal-responses yes;

category lame-servers {null;};
```
![image](https://github.com/NyashMan/DEMO2024/assets/1348639/52f85f0f-c239-430c-89ae-db65267e00b5)  
![image](https://github.com/NyashMan/DEMO2024/assets/1348639/fe226eca-2c18-4af4-8547-a4ea54a27b5c)  
```
systemctl stop bind
nano /etc/sysconfig/network
```
![image](https://github.com/NyashMan/DEMO2024/assets/1348639/d18b6f9c-ab19-4f81-b4ea-91b161f99be8)  
```
hostnamectl set-hostname hq-srv.demo.first;exec bash
domainname demo.first
rm -f /etc/samba/smb.conf
rm -rf /var/lib/samba
rm -rf /var/cache/samba
mkdir -p /var/lib/samba/sysvol
samba-tool domain provision
```
![image](https://github.com/NyashMan/DEMO2024/assets/1348639/6a877b28-1400-48f4-9f80-9af0be13ee88)  

```
systemctl enable --now samba
systemctl enable --now bind
cp /var/lib/samba/private/krb5.conf /etc/krb5.conf
samba-tool domain info 127.0.0.1
```
![image](https://github.com/NyashMan/DEMO2024/assets/1348639/3d30b5b2-144b-4347-828a-86ffdcb56759)  
```
host -t SRV _kerberos._udp.demo.first.
host -t SRV _ldap._tcp.demo.first.
host -t A hq-srv.demo.first
```
![image](https://github.com/NyashMan/DEMO2024/assets/1348639/a5ac8e81-d2c5-4bc8-aa99-ee6613652bf5)  
```
kinit administrator@DEMO.FIRST
```
![image](https://github.com/NyashMan/DEMO2024/assets/1348639/ab190c07-ce7f-4ed8-80c7-7fe7cc69955c)  

Вводим машины в домен:  
## **CLI**
Указываем DNS сервер домена  
![image](https://github.com/NyashMan/DEMO2024/assets/1348639/5a770be8-4b58-4a5e-9a0b-f0c2c5065ed2)  
![image](https://github.com/NyashMan/DEMO2024/assets/1348639/cf042173-7b61-45f4-805a-41748b9bbc6e)  
Отключаем и включаем сетевое соединение  
![image](https://github.com/NyashMan/DEMO2024/assets/1348639/221af92f-31cf-4443-90dd-31e48613a900)  
![image](https://github.com/NyashMan/DEMO2024/assets/1348639/a1ddcd99-449f-4ee3-9fb1-a14dba15cd0a)  
```
acc
```
![image](https://github.com/NyashMan/DEMO2024/assets/1348639/defc901c-27ac-418e-9198-6af22620939e)  
![image](https://github.com/NyashMan/DEMO2024/assets/1348639/5d753e57-d2ab-433b-8f22-a48df6d62449)  
![image](https://github.com/NyashMan/DEMO2024/assets/1348639/759f6dee-63c7-4a0e-819b-f297e51349f4)  
```
reboot
```
Авторизируемся под учётной записью administrator  
![image](https://github.com/NyashMan/DEMO2024/assets/1348639/8dd05567-f23c-479f-83b3-2a763d4b3e0c)  
Пароль: P@ssw0rd   
![image](https://github.com/NyashMan/DEMO2024/assets/1348639/0da696e1-bfd2-4014-966e-9b7df6eaa484)  

## **BR-SRV**
Указываем DNS сервер домена  
![image](https://github.com/NyashMan/DEMO2024/assets/1348639/5a770be8-4b58-4a5e-9a0b-f0c2c5065ed2)  
![image](https://github.com/NyashMan/DEMO2024/assets/1348639/d46862d7-3ce5-4ee2-9753-7221822ae075)  
Отключаем и включаем сетевое соединение  
![image](https://github.com/NyashMan/DEMO2024/assets/1348639/221af92f-31cf-4443-90dd-31e48613a900)  
![image](https://github.com/NyashMan/DEMO2024/assets/1348639/a1ddcd99-449f-4ee3-9fb1-a14dba15cd0a)  
```
acc
```
![image](https://github.com/NyashMan/DEMO2024/assets/1348639/defc901c-27ac-418e-9198-6af22620939e)  
![image](https://github.com/NyashMan/DEMO2024/assets/1348639/f1a96ed4-f52e-46e7-b3ff-6952c6093d62)  
![image](https://github.com/NyashMan/DEMO2024/assets/1348639/7f034e5c-9c52-4db9-ae51-dabd30f9a366)  
![image](https://github.com/NyashMan/DEMO2024/assets/1348639/1d146328-0b59-49f3-a952-c5d88ccaea95)  

```
reboot
```
Авторизируемся под учётной записью administrator  
![image](https://github.com/NyashMan/DEMO2024/assets/1348639/a486a543-ae64-4c3d-86b7-af40d0e0ec38)  
Пароль: P@ssw0rd  
![image](https://github.com/NyashMan/DEMO2024/assets/1348639/44323daf-1640-4067-9b9c-8226aa18f666)  


## **CLI**
Необходимо зайти обратно под пользователем user
```
kinit administrator
```
![image](https://github.com/NyashMan/DEMO2024/assets/1348639/bfffad87-29d0-4a4c-b8ae-2c938764e237)  
```
admc
```
Необходимо создать доменных пользователей из таблицы (они не конфликтуют с теми, что были созданы ранее), они необходимы для настройки доступа к сетевым хранилищам.  

| Логин | Пароль | 
| :---         |     ---:      | 
| Admin   | P@ssw0rd     | 
| Branch admin     | P@ssw0rd       | 
| Network admin     | P@ssw0rd       | 

![image](https://github.com/NyashMan/DEMO2024/assets/1348639/fad947e9-f101-454b-91f4-c9411b220172)  
Создаём группы для только что созданных пользователей:  
![image](https://github.com/NyashMan/DEMO2024/assets/1348639/49402392-a1ff-4428-a264-497e0e8cdc2f)  

**4. Реализуйте файловый SMB или NFS (выбор обоснуйте) сервер на базе сервера HQ-SRV.**  

**a. Должны быть опубликованы общие папки по названиям:**  
**i. Branch_Files - только для пользователя Branch admin;**  
**ii. Network - только для пользователя Network admin;**  
**iii. Admin_Files - только для пользователя Admin;**  
**b. Каждая папка должна монтироваться на всех серверах в папку /mnt/ (например, /mnt/All_files) автоматически при входе доменного пользователя в систему и отключаться при его выходе из сессии. Монтироваться должны только доступные пользователю каталоги**  
**Обоснование**: Выбор Samba в качестве файлового протокола обеспечивает интеграцию Linux-систем в сети Windows, обеспечивает безопасный обмен файлами в сети и снижает затраты на создание и поддержку сетевого файлового сервера. Для удобного администрирования, т.к. ранее в качестве контроллера домена была выбрана технология samba, было принято решение использовать в качестве протокола для хранения данных - SMB, также реализован алгоритм контроля доступа к сетевым ресурсам с использованием учётных доменных записей пользователей.

## **HQ-SRV**
```
mkdir /opt/{branch,network,admin}
chmod 777 /opt/{branch,network,admin}
nano /etc/samba/smb.conf
```
![image](https://github.com/NyashMan/DEMO2024/assets/1348639/1e08ba6e-6325-4646-b942-a651c2b025a4)  
```
[Branch_Files]
path=/opt/branch
writable = yes
read only = no
valid users = @"DEMO\Branch admins"

[Network]
path=/opt/network
writable = yes
read only = no
valid users = @"DEMO\Network admins"

[Admin_Files]
path=/opt/admin
writable = yes
read only = no
valid users = @"DEMO\Admins"

```


```
systemctl restart samba
```
## **BR-SRV**
```
nano /etc/pam.d/system-auth
```
![image](https://github.com/NyashMan/DEMO2024/assets/1348639/f163a907-7f63-4bd5-937c-b34c860ad594)  

```
include
[success=1 default=ignore]    pam_succeed_if.so service = systemd-user quite
optional pam_mount.so disable_interactive
```

```
nano /etc/security/pam_mount.conf.xml
```
![image](https://github.com/NyashMan/DEMO2024/assets/1348639/eebb7cd4-c869-4d23-acd0-528654b8ac7b)
**P.S. напишите один раздел, затем скопируйте его 2 раза, поменяв uid и path**  

```
<volume uid="Admin"
fstype="cifs"
server="HQ-SRV.demo.first"
path="Admin_Files"
mountpoint="/mnt/All_files"
options="sec=krb5i,cruid=%(USERUID),nounix,uid=%(USERUID),gid=%(USERGID),file_mode=0664,dir_mode=0775"/>

<volume uid="Network admin"
fstype="cifs"
server="HQ-SRV.demo.first"
path="Network"
mountpoint="/mnt/All_files"
options="sec=krb5i,cruid=%(USERUID),nounix,uid=%(USERUID),gid=%(USERGID),file_mode=0664,dir_mode=0775"/>

<volume uid="Branch admin"
fstype="cifs"
server="HQ-SRV.demo.first"
path="Branch_Files"
mountpoint="/mnt/All_files"
options="sec=krb5i,cruid=%(USERUID),nounix,uid=%(USERUID),gid=%(USERGID),file_mode=0664,dir_mode=0775"/>
```

**Проверяем**
Заходим из под пользователя Admin:  
![image](https://github.com/NyashMan/DEMO2024/assets/1348639/0f0573e2-0031-48f4-8768-d69c1a158962)  
Заходим из под пользователя Branch admin:  
![image](https://github.com/NyashMan/DEMO2024/assets/1348639/61e2576e-31f1-4484-8d30-f5a7e7a9740c)  
Заходим из под пользователя Network admin:  
![image](https://github.com/NyashMan/DEMO2024/assets/1348639/28b3c8c2-c35f-43a5-9ad6-203d77ba6c2a)  

**5. Сконфигурируйте веб-сервер LMS Apache на сервере BR-SRV:**  

**a. На главной странице должен отражаться номер места**  
**b. Используйте базу данных mySQL**  
**c. Создайте пользователей в соответствии с таблицей**  

| Пользователь | Группа | 
| :---         |     ---:      | 
| Admin   | Admin     | 
| Manager1     | Manager       | 
| Manager2     | Manager       | 
| Manager3     | Manager       |
| User1     | WS       |
| User2     | WS       |
| User3     | WS       |
| User4     | WS       |
| User5     | TEAM       |
| User6     | TEAM       |
| User7     | TEAM       |  

## **BR-SRV**


```
mariadb
CREATE DATABASE moodle;
CREATE USER 'moodle'@’localhost’ IDENTIFIED BY 'moodle';
GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,CREATE TEMPORARY TABLES,DROP,INDEX,ALTER ON moodle.* TO 'moodle'@’localhost’;
nano /var/www/html/moodle/config.php
```
![image](https://github.com/NyashMan/DEMO2024/assets/1348639/7a56b8d6-6d4c-48b7-95b3-46305949f272)  
```
systemctl enable --now httpd2.service
```
Проверяем доступность развёрнутого портала по адресу http://moodle.localhost/install.php
![image](https://github.com/NyashMan/DEMO2024/assets/1348639/98c858ee-21ba-44de-a1b5-773dd51c381f)  

Настроим доступ по имени на клиенте:
## **CLI**
```
echo 10.0.2.2 moodle.demo.first moodle >> /etc/hosts
```
Проверяем:  
![image](https://github.com/NyashMan/DEMO2024/assets/1348639/3c386270-ec05-4569-a9cb-75420348db7d)  

Производим установку: 
**Выбирайте англ. язык!** 
![image](https://github.com/NyashMan/DEMO2024/assets/1348639/10c8e350-c9d1-44ce-a067-417a83da062d)  
![image](https://github.com/NyashMan/DEMO2024/assets/1348639/d7a7dd69-934c-4c61-90b6-6c9433b97063)  
![image](https://github.com/NyashMan/DEMO2024/assets/1348639/ed9fa557-ca17-4d5e-b7a1-ba6abe03d3f6)  
![image](https://github.com/NyashMan/DEMO2024/assets/1348639/a9059078-920d-4d5d-a864-191e71407b6a)  
![image](https://github.com/NyashMan/DEMO2024/assets/1348639/435487cc-f7bc-4e75-af86-363bce94c161)  
![image](https://github.com/NyashMan/DEMO2024/assets/1348639/a5c0ad7c-7087-4f26-8076-626266f4f341)  
![image](https://github.com/NyashMan/DEMO2024/assets/1348639/a4291eac-c41a-4086-bbb0-ba2ee3e2e64d)  
![image](https://github.com/NyashMan/DEMO2024/assets/1348639/3517aa0a-b74b-4ae4-8497-e4059bd207da)  
![image](https://github.com/NyashMan/DEMO2024/assets/1348639/6cf17575-c680-48e0-9efa-9c63712e504e)  
Создаём группы для пользователей заданных в таблице:  
![image](https://github.com/NyashMan/DEMO2024/assets/1348639/9cb5e81a-05e1-4df6-9872-64411c725070)  
![image](https://github.com/NyashMan/DEMO2024/assets/1348639/be52d171-3065-4542-8995-c67299184cc1)  
![image](https://github.com/NyashMan/DEMO2024/assets/1348639/e1149176-aa8d-4bdb-bc02-1ff0f4014722)  
![image](https://github.com/NyashMan/DEMO2024/assets/1348639/92c2fc9e-ceb3-4b9e-ad5f-b703e29089b0)  
По аналогии создаём оставшиеся группы  
Проверяем:  
![image](https://github.com/NyashMan/DEMO2024/assets/1348639/7180999f-9d31-4987-9a45-fed431832c57)  
Создаём пользователей:  
![image](https://github.com/NyashMan/DEMO2024/assets/1348639/66529690-2e64-4a03-ae3a-2c69f4d1eff4)  
![image](https://github.com/NyashMan/DEMO2024/assets/1348639/006d7ef5-6d7b-4f11-823e-c243d5531ce3)  
![image](https://github.com/NyashMan/DEMO2024/assets/1348639/237d3f1b-7e20-4c45-98a4-d6ad84ab5dbd)  
![image](https://github.com/NyashMan/DEMO2024/assets/1348639/1194c06c-d138-4496-8ab1-7c98b9dc230d)  
По аналогии создаём оставшихся пользователей  
Проверяем:  
![image](https://github.com/NyashMan/DEMO2024/assets/1348639/abc9045b-d7af-4648-8bf0-36330bdda176)  
Добавляем пользователей в соответствующие группы:  
![image](https://github.com/NyashMan/DEMO2024/assets/1348639/2061b9cd-8403-4334-b7f1-42eaa98f4861)  
![image](https://github.com/NyashMan/DEMO2024/assets/1348639/a24eadca-7270-44cd-87f9-5a49d7ad72e8)  
По аналогии добавляем оставшихся пользователей в группы  
Проверяем:  
![image](https://github.com/NyashMan/DEMO2024/assets/1348639/00b648c5-7d9f-46eb-ba90-8d31299f13fc)  

**6. Запустите сервис MediaWiki используя docker на сервере HQ-SRV.**  

**a. Установите Docker и Docker Compose.**  
**b. Создайте в домашней директории пользователя файл wiki.yml для приложения MediaWiki:**  
**i. Средствами docker compose должен создаваться стек контейнеров с приложением MediaWiki и базой данных**  
**ii. Используйте два сервиса;**  
**iii. Основной контейнер MediaWiki должен называться wiki и использовать образ mediawiki;**  
**iv. Файл LocalSettings.php с корректными настройками должен находиться в домашней папке пользователя и автоматически монтироваться в образ;**  
**v. Контейнер с базой данных должен называться db и использовать образ mysql;**  
**vi. Он должен создавать базу с названием mediawiki, доступную по стандартному порту, для пользователя wiki с паролем DEP@ssw0rd;**  
**vii. База должна храниться в отдельном volume с названием dbvolume.**  
**MediaWiki должна быть доступна извне через порт 8080.**  

## **HQ-SRV**

Необходимо выключить сервис alterator
```
systemctl disable --now ahttpd
systemctl disable --now alteratord
```
Проверяем содержимое файла:
```
nano ~/wiki.yml
```
![image](https://github.com/NyashMan/DEMO2024/assets/1348639/b01ce929-61b7-476e-bb53-1986727648dc)  

Перезагружаем сервисы docker:  
```
systemctl restart docker
docker start db
docker start wiki
docker ps
```
![image](https://github.com/NyashMan/DEMO2024/assets/1348639/1fe949ae-8b76-4255-a73a-74e369d56152)  
Проверяем доступность ресурса через браузер:  
![image](https://github.com/NyashMan/DEMO2024/assets/1348639/38eea153-89bb-4690-a4e1-1cce7257cb39)  

Настроим доступность ресурса извне:  
## **CLI**

```
echo 10.0.0.2 mediawiki.demo.first mediawiki >> /etc/hosts
```
Проверяем доступность:  
![image](https://github.com/NyashMan/DEMO2024/assets/1348639/d821eac7-f394-4702-816d-2917aacefa61)  
Производим настройку: 
## **HQ-SRV**
![image](https://github.com/NyashMan/DEMO2024/assets/1348639/21e1e0e5-4c3f-4976-a586-47f4d7c9fe08)  
![image](https://github.com/NyashMan/DEMO2024/assets/1348639/5bc4f9bc-1bf0-4b87-98f5-0144c5d8993d)   
![image](https://github.com/NyashMan/DEMO2024/assets/1348639/7a56c424-d208-4b8e-a08f-9eed876f0b5f)  
Пароль от БД: DEP@ssw0rd  
![image](https://github.com/NyashMan/DEMO2024/assets/1348639/a2d00579-a8f3-4bdf-aa26-7a32fe21900c)  
![image](https://github.com/NyashMan/DEMO2024/assets/1348639/7cd2e4c7-e435-4c51-abe8-97519f8f9bd9)  
![image](https://github.com/NyashMan/DEMO2024/assets/1348639/d8b291f4-d493-485c-bfc7-a836250ae2d8)   
![image](https://github.com/NyashMan/DEMO2024/assets/1348639/34986562-91b2-4b6f-8baa-2ae82f69981f)  
![image](https://github.com/NyashMan/DEMO2024/assets/1348639/fb8bf074-00f5-4cf0-817c-ca4a2c7fa344)    
После настройки, необходимо скачать файл LocalLocalSettings.php  
![image](https://github.com/NyashMan/DEMO2024/assets/1348639/6c6fb61f-8684-4279-bc39-768e0e06676d)  
Переместите скачанный файл в директорию, в которой находится файл wiki.yml  
```
su -
toor
cp /home/user/Загрузки/LocalSettings.php ./
nano ~/wiki.yml
```
Проверяем, что данная стока не закоментирована (Отсутствует #)
![image](https://github.com/NyashMan/DEMO2024/assets/1348639/384057a1-0fcf-407c-a12b-a031d6b2150d)  
```
ctrl-x
y
docker-compose -f wiki.yml stop
docker-compose -f wiki.yml up -d
```
Проверяем доступ к http://mediawiki.demo.first:8080:
## **CLI**
![image](https://github.com/NyashMan/DEMO2024/assets/1348639/1ee2b717-a8db-480a-bbd5-19861bc35efc)  
вход осуществляем из под пользователя **wiki** с паролем **DEP@ssw0rd** :
![image](https://github.com/NyashMan/DEMO2024/assets/1348639/b1a99892-5e4d-4ab0-95b2-cf80523b5850)  
![image](https://github.com/NyashMan/DEMO2024/assets/1348639/5db17453-6a94-4244-8a4d-b29e3dcfe2f7)  

# **Under Construction**
