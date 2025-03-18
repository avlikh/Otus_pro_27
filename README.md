# OTUS PRO Homework 27 backup

## Домашняя работа 27: Резервное копирование

### Домашнее задание:
Описание домашнего задания:

**1. Подготовка рабочего места**

**2. В Office1 в тестовой подсети появляется сервера с доп интерфейсами и адресами в internal сети testLAN:**    
* testClient1 - 10.10.10.254
* testClient2 - 10.10.10.254
* testServer1- 10.10.10.1
* testServer2- 10.10.10.1    
    
**Развести вланами:**
* testClient1 <-> testServer1
* testClient2 <-> testServer2

Между centralRouter и inetRouter "пробросить" 2 линка (общая inernal сеть) и объединить их в бонд, проверить работу c отключением интерфейсов.

---
### 1. Подготовка рабочего места:
Выполнение домашнего задания предполагает, что на компьютере установлен Vagrant+VirtualBox   
**[Как установить Vagrant на Debian 12](https://github.com/avlikh/Install_Vagrant_Debian12/blob/main/README.md)**   

Развернем Vagrant-стенд:
  - Создайте папку с проектом и зайдите в нее (например: /opt/otus/backup):
```
mkdir -p /opt/otus/backup ; cd /opt/otus/backup
```
  - Клонируете проект с Github, набрав команду:
```
apt update -y && apt install git -y ; git clone https://github.com/avlikh/Otus_pro_27.git .
```
  - Запустите проект из папки, в которую склонировали проект (в нашем примере /opt/otus/backup):
```
vagrant up
```
<details>
<summary> Результатом выполнения команды vagrant up станут созданные и настроенные 7 виртуальных машин: </summary>

```
123
```
</details>
    
    
* **inetRouter** 
* **centralRouter** 
* **office1Router**
* **testClient1**
* **testServer1**
* **testClient2**
* **testServer2**
---

## Выполнение задания:
### 2. Развернуть 7 виртуальных машин:    
    
В результате подготовки рабочего места (пункт 1) были созданы 7 виртуальные машины, к которым были применены необходимые настройки.    
Был выполнен playbook Ansible https://github.com/avlikh/Otus_pro_37/blob/main/provision.yaml.    
Созданные виртуальные машины соединены между сосбой, согласно схеме сети (см. ниже)

---
**Схема сети:**
     
![image](https://github.com/user-attachments/assets/84655336-28d8-411c-b7f5-fa5780355417)

**2.1 Проверим работу VLAN 1.** Для этого: 
* зайдем на **testClient1**: `vagrant ssh testClient1`
* посмотрим настройки сетевых интерфейсов: `ip -4 a`
* выполним пинг 10.10.10.254 и 10.10.10.1: `ping -c2 10.10.10.254 && ping -c2 10.10.10.1`
<details>
<summary> результат выполнения команды </summary>

```
123
```
</details>
    
Итак, видим что VLAN 1 настроен и работает. 

---

**2.2 Проверим работу VLAN 2.** Для этого: 
* зайдем на **testClient2**: `vagrant ssh testClient2`
* посмотрим настройки сетевых интерфейсов: `ip -4 a`
* выполним пинг 10.10.10.254 и 10.10.10.1: `ping -c2 10.10.10.254 && ping -c2 10.10.10.1`
<details>
<summary> результат выполнения команды </summary>

```
123
```
</details>
    
Итак, видим что VLAN 2 настроен и работает.

---

**2.3 Проверим работу LACP (Link Aggregation Control Protocol).** Для этого:

* зайдем на **inetRouter**: `vagrant ssh inetRouter`
* посмотрим настройки сетевых интерфейсов: `ip -4 a && cat /proc/net/bonding/bond0`
* выполним бесконечный ping до centralRouter (192.168.255.2): `ping 192.168.255.2`
<details>
<summary> результат выполнения команды </summary>

```
123
```
</details>

**Далее, в другом окне терминала зайдем на хост inetRouter и проверим работу LACP.** Для Этого: 
* Посмотрим настройки сетевых интерфейсов: `ip -4 a && cat /proc/net/bonding/bond0`
* Отключим интерфейс eth1 на 10 секунд `ip link set down eth1`
* Посмотрим настройки сетевых интерфейсов: `ip -4 a && cat /proc/net/bonding/bond0`
* Включим его обратно `ip link set up eth1`
* Посмотрим настройки сетевых интерфейсов: `ip -4 a && cat /proc/net/bonding/bond0`
* Отключим интерфейс eth2 на 10 секунд: `ip link set down eth2`
* Посмотрим настройки сетевых интерфейсов: `ip -4 a && cat /proc/net/bonding/bond0`
* Включим его обратно: `ip link set up eth2`
* Посмотрим настройки сетевых интерфейсов: `ip -4 a && cat /proc/net/bonding/bond0`
* Примечание: Интерфейсы eth1 и eth2 объединены в bond что должно гарантировать, что при отключении одного из интерфейсов, сетевая связанность не нарушится.
<details>
<summary> результат выполнения команд </summary>

```
123
```
</details>

Вернемся на хост **inetRouter** и посмотрим были ли потери пакетов до centralRouter (192.168.255.2):

<details>
<summary> Видим, что потери сетевой связанности не было </summary>

```
123
```
</details>
    
**Итак, видим что LACP работает и пакеты не теряются при отключении одного из интерфейсов сети, объединенной в BOND.**

