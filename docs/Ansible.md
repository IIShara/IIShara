# Ansible

## Установка
Выполним как саму установку ansible, так и дополнительные коллекции.

### Установка ansible на Linux
Установка для различных дистрибутивов Linux выполняется, немного, по-разному. Рассмотрим варианты.

**Ubuntu / Debian / Astra**

Выполняем команды:
```shell
apt update
```
```shell
apt install ansible
```
**CentOS / Rocky**

По умолчанию Ansible нет в репозитории CentOS — устанавливаем EPEL:
```shell
yum install epel-release
```
После устанавливаем сам сервер управления:
```shell
yum install ansible
```
\* *система автоматически обновит список пакетов с учетом нового репозитория и начнет установку Ansible. Если появится запрос на подтверждение, отвечаем Y.*

**РЕД ОС**
Устанавливаем ансибл командой:
```shell
dnf install ansible
```
### Установка и обновление коллекций
Рекомендуется сразу установить или обновить стандартные коллекции для задач Ansible:
```shell
ansible-galaxy collection install community.general
```
## Начальная настройка и тестовый запуск Ansible
Разобьем наши действия на небольшие группы.

### Настройка инвентарного файла
В данном файле хранится информация о хостах и/или группах хостов. Также в нем может храниться переменные, определенные для конкретной группы хостов или конкретного компьютера.

Есть два варианта написания инвентарного файла — в формате yml или ini. Рассмотрим оба.

**1. Файл формата ini.**

Даннай формат используется по умолчанию. Откроем на редактирование файл с серверами, которыми хотим управлять:
```shell
vi /etc/ansible/hosts
```
и приведем его к следующему виду:
```vim
[test_servers]
192.168.1.100
192.168.1.101
```
\* *в данном примере создана группа серверов test_servers, в которую добавлены два сервера с IP-адресами 192.168.1.100 и 192.168.1.101.*

**2. Файл формата yml.**

Создадим отдельный каталог:
```shell
mkdir /etc/ansible/inventory
```
И создадим файл:
```shell
vi /etc/ansible/inventory/test_servers.yml
```
```vim
---

test_servers:
  vars:
    ansible_python_interpreter: /usr/bin/python3
  hosts:
    server01:
      ansible_ssh_host: 192.168.1.100
      ansible_ssh_port: 22
    server02:
      ansible_ssh_host: 192.168.1.101
      ansible_ssh_port: 22
    server03:
      ansible_connection: local
```
\* *в данном примере также создана группа серверов test_servers, в которую добавлены три сервера server01 и server02 с IP-адресами 192.168.1.100 и 192.168.1.101, а также server03, который является локальным сервером с типом подключения local. Адреса не обязательно писать, если имя машины разрешается с помощью DNS. Также не обязательно указывать порты, если они стандартные (22).*

** *обратите внимание, что мы также добавили переменную ansible_python_interpreter с указанием пути для запуска python.*

### Настройка ansible
Открываем конфигурационный файл ansible:
```shell
vi /etc/ansible/ansible.cfg
```
Для более новых версий ansible конфигурационный файл не создается. Создаем для него каталог и сам файл:
```shell
mkdir /etc/ansible
```
```shell
vi /etc/ansible/ansible.cfg
```
Снимаем комментарий с опции host_key_checking, приведя ее к виду:
```vim
host_key_checking = False
```
\* *данная настройка позволит нашему серверу управления автоматически принимать ssh fingerprint, избавляя нас от необходимости постоянно вводить yes, когда мы впервые конфигурируем новый сервер.*

Также в секцию `defaults` добавим:
```vim
[defaults]
...
interpreter_python = auto_silent
```
\* *данная опция указывает, чтобы ansible автоматически искал python на целевом хосте без показа предупреждений.*

### Тестовый запуск
Теперь выполним проверку доступности добавленных серверов:
```shell
ansible -m ping test_servers -u root -kK
```
\* *данная команда проверит доступность по сети двух серверов из группы test_servers от учетной записи root.*

> 1. Если мы создали и хотим использовать инвентарный файл yml, то вводим:
>```shell
>ansible -i /etc/ansible/inventory/test_servers.yml -m ping test_servers -u root -kK
>```
>\* *мы должны указать путь до файла inventory с помощью опции -i.*
>
>2. Если мы хотим выполнить сценарий, подключившись к локальному компьютеру, то выполняем:
>```shell
>ansible -m ping localhost --connection=local
>```
Если подключение не local, будет запрошен пароль от учетной записи (в нашем случае, root). После будет запрошен пароль суперпользователя на серверах.

На экране должно появиться, примерно, следующее:
```shell
192.168.1.100 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
192.168.1.101 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
```
Наш сервер управления готов к работе.

## Подключение без пароля
В нашем примере аутентификация на узлах выполняется с помощью пароля. Это не очень удобно и безопасно. Рассмотрим вариант использования ssh ключа.

На компьютере с `ansible` сгенерируем пару ключей следующей командой:
```shell
ssh-keygen -t ed25519
```
После нажатия *Enter* система попросит ввести параметры размещения ключа и пароль. Ничего не меняем, нажимая ввод и соглашаясь со значениями по умолчанию.

Мы увидим что-то на подобие:
```shell
Generating public/private ed25519 key pair.
Enter file in which to save the key (/root/.ssh/id_ed25519): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_ed25519
Your public key has been saved in /root/.ssh/id_ed25519.pub
The key fingerprint is:
SHA256:FTDGR6Gz92WI33/ywMSImcqfx2IRQjrD20tE/qzqZ5Y root@ansible.dmosk.ru
The key's randomart image is:
+--[ED25519 256]--+
|       .+o+.     |
|       .+o..     |
|     . =o..      |
|      = ++.= +   |
|       *S+*.o =  |
|      ..oo+o *   |
|       .o+ oo +  |
|        E.o.o .o.|
|     .o= .oo   o+|
+----[SHA256]-----+
```
\* *обратите внимание, что мы сгенерировали ключи под пользователем root и из местоположение в каталоге */root/.ssh.**

Теперь скопируем публичный ключ на все серверы, к которым будем подключаться:
```shell
ssh-copy-id -i /root/.ssh/id_ed25519.pub root@192.168.1.100
```
```shell
ssh-copy-id -i /root/.ssh/id_ed25519.pub root@192.168.1.101
```
\* *в нашем примере мы копируем наш ключ для пользователя root на удаленной системе.*

>Мы также можем скопировать ключ вручную. Для этого содержимое публичного файла (в нашем случае, >`id_ed25519.pub`) добавить в файл `authorized_keys` для нужного пользователя. Например:
>```shell
>mkdir /root/.ssh
>```
>```shell
>vi /root/.ssh/authorized_keys
>```
>\* *в данном примере мы сможем подключиться к компьютеру с нашим сертификатом под пользователм root.*

Готово. Попробуем подключиться по SSH без пароля к любому из серверов:
```shell
ssh root@192.168.1.100
```
Мы должны подключиться к серверу без запроса пароля.

Теперь можно попробовать запустить наш ансибл без ввода паролей:
```shell
ansible -m ping test_servers -u root
```
>Или с использованием инвентарного файла yml:
>```shell
>ansible -i /etc/ansible/inventory/test_servers.yml -m ping test_servers -u root
>```
Команда должна выполниться без запросов пароля.

## Использование плейбуков
Плейбук представляет из себя файл, в котором мы перечисляем список задач и/или ролей для выполнения ансиблом. Более того, в нем можно задать переменные и поведение работы ansible. Рассмотрим пример:
```shell
vi my_play.yml
```
```vim
- hosts: all
  gather_facts: yes
  user: root
  vars:
    app_ver: 14
  roles:
    - role: prepare
    - role: install
    - role: settings

- hosts: server1
  gather_facts: no
  user: root
  tasks:
    - name: install curl
      package:
        name: curl
        state: present
```
В данном примере мы выполняем 2 блока задач:

1. На всех хостах (hosts: all) нашего инвентарного файла запускаются 3 роли (prepare, install, settings). Ansible выполняет сбор фактов (gather_facts: yes), подключение к хостам будет выполняться от пользователя root, а также создается переменная app_ver: 14.
2. На узле server1 запускается одна задача по установке пакета curl.

Теперь запустим плейбук командой:
```shell
ansible-playbook -i /etc/ansible/inventory/test_servers.yml ./my_play.yml
```
\* *мы используем созданный ранее инвентарный файл и запускаем плейбук my_play.yml.*