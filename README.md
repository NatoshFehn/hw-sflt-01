# Домашнее задание к занятию «Disaster recovery и Keepalived» - Наталья Мартынова (Пономарева)

### Задание 1

1). Конфигурируем маршрутизатор: Router1

```
  enable
  configure terminal
```
  
2). Проверяем текущую конфигурацию
 
```
  do show running-config
  do show standby brief
```
3). Настраиваем интерфейс

```
  interface GigabitEthernet0/1
```
4). Выставляем приоритет
 
```
  standby 1 priority 105  
  standby 1 preempt
  standby 1 track GigabitEthernet 0/0
```
5). Конфигурируем маршрутизатор: Router1

6). Проверяем текущую конфигурацию
 
```
  do show running-config
  do show standby brief
```
7). Настраиваем интерфейс

```
  interface GigabitEthernet0/1
```
8). Выставляем приоритет
 
```
  standby 1 preempt
  standby 1 track GigabitEthernet 0/0
```
[Схема pkt](https://github.com/NatoshFehn/hw-sflt-01/blob/main/hw-sflt-01-1.pkt)

 ---

### Задание 2

1). Создаю две виртуальные машины Linux

2). Конфигурационный файл keepalived.conf

```
global_defs {
    script_user root
    enable_script_security
}

vrrp_script check_srv {
    script "/etc/keepalived/scripts/check_testsrv.sh"
    interval 3
}

vrrp_instance VI_1 {
        state MASTER
        interface enp0s3
        virtual_router_id 15
        priority 100
        advert_int 1

        virtual_ipaddress {
                192.168.56.15/24
        }

        track_script {
           check_srv
        }
}

```

3). Для выполения условия проверки наличия файла index.html и доступности сервера на 80 порту применяем скрипт:

```
#!/bin/bash

FILE="/var/www/html/index.html"
PORT="80"

# netstat
if ! [ -x "$(command -v netstat)" ]; then
  echo 'netstat is not installed.' >&2
  exit 1
fi

# check port
if netstat -tuln | grep ":$PORT " > /dev/null; then
  if [ -e "$FILE" ]; then exit 0  # "0"
  else exit 1 # "1" "Port 80 is open, File \"index.html\" does not exist"
  fi
else
  if [ -e "$FILE" ]; then exit 1  # "1" "Port 80 is closed, File \"index.html\" exists"
  else exit 1 # "1" Port 80 is closed, File \"index.html\" does not exist"
  fi
fi

```

4). Веб-сервер Nginx, доступен на виртуальном ip-адресе 192.168.56.15

5). Состояние сервиса keepalived на сервере (Master)

 ![Снимок1](https://github.com/NatoshFehn/hw-sflt-01/blob/main/Снимок1.jpg)  
 
6). При остановке сервера или при отсутсвии файла index.html 

 ![Снимок2](https://github.com/NatoshFehn/hw-sflt-01/blob/main/Снимок2.jpg)
 
7). Web страница
 ![Снимок3](https://github.com/NatoshFehn/hw-sflt-01/blob/main/Снимок3.jpg)
 
8). Состояние сервиса keepalived на втором серевере (Backup)
 
 ![Снимок4](https://github.com/NatoshFehn/hw-sflt-01/blob/main/Снимок4.jpg)
 
9). При возобновлении работы сервера и при наличии файла index.html 
 
 ![Снимок5](https://github.com/NatoshFehn/hw-sflt-01/blob/main/Снимок5.jpg)

10). Web страница
 
 ![Снимок6](https://github.com/NatoshFehn/hw-sflt-01/blob/main/Снимок6.jpg)
