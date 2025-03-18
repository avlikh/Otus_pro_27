# OTUS PRO Homework 27 backup

## Домашняя работа 27: Резервное копирование

### Домашнее задание:
Описание домашнего задания:

**1. Подготовка рабочего места**

**2. Настроить стенд Vagrant с двумя виртуальными машинами: backup_server и client. (Студент самостоятельно настраивает Vagrant)**    
    
**2.1 Настроить удаленный бэкап каталога /etc c сервера client при помощи borgbackup. Резервные копии должны соответствовать следующим критериям:**
* директория для резервных копий /var/backup. Это должна быть отдельная точка монтирования. В данном случае для демонстрации размер не принципиален, достаточно будет и 2GB; (Студент самостоятельно настраивает)
* репозиторий для резервных копий должен быть зашифрован ключом или паролем - на усмотрение студента;
имя бэкапа должно содержать информацию о времени снятия бекапа;
* глубина бекапа должна быть год, хранить можно по последней копии на конец месяца, кроме последних трех. Последние три месяца должны содержать копии на каждый день. Т.е. должна быть правильно настроена политика удаления старых бэкапов;
* резервная копия снимается каждые 5 минут. Такой частый запуск в целях демонстрации;
* написан скрипт для снятия резервных копий. Скрипт запускается из соответствующей Cron джобы, либо systemd timer-а - на усмотрение студента;
* настроено логирование процесса бекапа. Для упрощения можно весь вывод перенаправлять в logger с соответствующим тегом. Если настроите не в syslog, то обязательна ротация логов.

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
<summary> Результатом выполнения команды vagrant up станут созданные и настроенные 2 виртуальных машин: </summary>

```
root@deb4likh:/opt/otus/backup# vagrant up
Bringing machine 'backup' up with 'virtualbox' provider...
Bringing machine 'client' up with 'virtualbox' provider...
==> backup: Importing base box 'debian/bookworm64'...
==> backup: Matching MAC address for NAT networking...
==> backup: Checking if box 'debian/bookworm64' version '12.20240905.1' is up to date...
==> backup: A newer version of the box 'debian/bookworm64' for provider 'virtualbox' is
==> backup: available! You currently have version '12.20240905.1'. The latest is version
==> backup: '12.20250126.1'. Run `vagrant box update` to update.
==> backup: Setting the name of the VM: backup_backup_1742293516341_82645
==> backup: Clearing any previously set network interfaces...
==> backup: Preparing network interfaces based on configuration...
    backup: Adapter 1: nat
    backup: Adapter 2: hostonly
==> backup: Forwarding ports...
    backup: 22 (guest) => 2222 (host) (adapter 1)
==> backup: Configuring storage mediums...
    backup: Disk 'backup' not found in guest. Creating and attaching disk to guest...
==> backup: Running 'pre-boot' VM customizations...
==> backup: Booting VM...
==> backup: Waiting for machine to boot. This may take a few minutes...
    backup: SSH address: 127.0.0.1:2222
    backup: SSH username: vagrant
    backup: SSH auth method: private key
    backup: Warning: Remote connection disconnect. Retrying...
    backup: Warning: Connection reset. Retrying...
    backup:
    backup: Vagrant insecure key detected. Vagrant will automatically replace
    backup: this with a newly generated keypair for better security.
    backup:
    backup: Inserting generated public key within guest...
    backup: Removing insecure key from the guest if it's present...
    backup: Key inserted! Disconnecting and reconnecting using new SSH key...
==> backup: Machine booted and ready!
==> backup: Checking for guest additions in VM...
    backup: The guest additions on this VM do not match the installed version of
    backup: VirtualBox! In most cases this is fine, but in rare cases it can
    backup: prevent things such as shared folders from working properly. If you see
    backup: shared folder errors, please make sure the guest additions within the
    backup: virtual machine match the version of VirtualBox you have installed on
    backup: your host and reload your VM.
    backup:
    backup: Guest Additions Version: 6.0.0 r127566
    backup: VirtualBox Version: 7.1
==> backup: Setting hostname...
==> backup: Configuring and enabling network interfaces...
==> backup: Mounting shared folders...
    backup: /opt/otus/backup => /vagrant
==> backup: Running provisioner: shell...
    backup: Running: inline script
..........
==> client: Running provisioner: ansible...
    client: Running ansible-playbook...

PLAY [Install Borg] ************************************************************

TASK [Gathering Facts] *********************************************************
ok: [backup]
ok: [client]

TASK [install : install_borg] **************************************************
changed: [backup]
changed: [client]

PLAY [Configure server] ********************************************************

TASK [Gathering Facts] *********************************************************
ok: [backup]

TASK [server : Create user borg] ***********************************************
changed: [backup]

TASK [server : check and setup dir /var/backup] ********************************
changed: [backup]

TASK [server : clean dir /var/backup] ******************************************
changed: [backup]

PLAY [Configure ssh] ***********************************************************

TASK [Gathering Facts] *********************************************************
ok: [backup]

TASK [ssh_key : SSH  Disable confirmation for add ssh-key в known_hosts] *******
changed: [backup -> client(192.168.56.12)]

TASK [ssh_key : Generate SSH key for client (if none)] *************************
changed: [backup -> client(192.168.56.12)]

TASK [ssh_key : Get public key from client] ************************************
ok: [backup -> client(192.168.56.12)]

TASK [ssh_key : Create directory (if none)] ************************************
changed: [backup]

TASK [ssh_key : Add public key client into authorized_keys in backup host] *****
changed: [backup]

RUNNING HANDLER [ssh_key : restart_ssh] ****************************************
changed: [backup]

PLAY [Init Repo] ***************************************************************

TASK [Gathering Facts] *********************************************************
ok: [client]

TASK [init_repo : Repository init] *********************************************
changed: [client]

PLAY [Install service & timer] *************************************************

TASK [Gathering Facts] *********************************************************
ok: [client]

TASK [service : Create service from template] **********************************
changed: [client]

TASK [service : Create timer from template] ************************************
changed: [client]

TASK [service : run and enable timer] ******************************************
changed: [client]

TASK [service : run backup first time so timer will have first record] *********
changed: [client]

PLAY RECAP *********************************************************************
backup                     : ok=13   changed=9    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
client                     : ok=9    changed=6    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0


==> backup: Machine 'backup' has a post `vagrant up` message. This is a message
==> backup: from the creator of the Vagrantfile, and not from Vagrant itself:
==> backup:
==> backup: Vanilla Debian box. See https://app.vagrantup.com/debian for help and bug reports

==> client: Machine 'client' has a post `vagrant up` message. This is a message
==> client: from the creator of the Vagrantfile, and not from Vagrant itself:
==> client:
==> client: Vanilla Debian box. See https://app.vagrantup.com/debian for help and bug reports

```
</details>
    
    
* **backup** 
* **client** 

---

## Выполнение задания:
**2. Настроить стенд Vagrant с двумя виртуальными машинами: backup_server и client. (Студент самостоятельно настраивает Vagrant)**    
    
* Выполнено при помощи Vagrant+Ansible

---

**2.1 Настроить удаленный бэкап каталога /etc c сервера client при помощи borgbackup.**
    
В результате подготовки рабочего места (пункт 1) были созданы 2 виртуальные машины, к которым были применены необходимые настройки.    
Был выполнен playbook Ansible https://github.com/avlikh/Otus_pro_27/blob/main/provision.yaml.    
Созданные виртуальные машины соединены между сосбой, согласно схеме сети (см. ниже)
     
* зайдем на **client** и повысим полномочия до root: `vagrant ssh testClient1` `sudo -i`
<details>
<summary> результат выполнения команд </summary>

```
root@deb4likh:/opt/otus/backup# vagrant ssh client
Linux client 6.1.0-25-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.106-3 (2024-08-26) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Tue Mar 18 10:28:59 2025 from 192.168.56.1
vagrant@client:~$ sudo -i
```

<details>
    
Убедимся что служба таймера работает `systemctl status borg-backup-client.timer` 
<summary> результат выполнения команды  </summary>

```
root@client:~# systemctl status borg-backup-client.timer
● borg-backup-client.timer - Borg Service client Backup
     Loaded: loaded (/etc/systemd/system/borg-backup-client.timer; enabled; preset: enabled)
     Active: active (waiting) since Tue 2025-03-18 10:28:59 UTC; 6h ago
    Trigger: Tue 2025-03-18 10:44:35 UTC; 44s left
   Triggers: ● borg-backup-client.service

Mar 18 10:28:59 client systemd[1]: Started borg-backup-client.timer - Borg Service client Backup.
```
</details>
    
Видим что таймер работает.   
    
* Далее, оставим наш стенд работать на несколько часов.    

Посмотрим на репозитории borg: `borg list borg@192.168.56.11:/var/backup`
* Примечание: для доступа к репозиториям borg потребуется пароль: **Otus1234**

<details>
<summary> результат выполнения команды </summary>

```
root@client:~# borg list borg@192.168.56.11:/var/backup
Remote: Warning: Permanently added '192.168.56.11' (ED25519) to the list of known hosts.
Enter passphrase for key ssh://borg@192.168.56.11/var/backup:
client-2025-03-18_10:29:00           Tue, 2025-03-18 10:29:01 [0a95c2fe0c01a5947c15d31631dd32a5902dc6a70ad6531c95a564740b874e72]
client-2025-03-18_16:50:54           Tue, 2025-03-18 16:50:55 [53f2196e51b7af3ab09516be7b988c658f2217b367a0b0f38ba759c5cf6806e2]
```
</details>

Посмотрим логи borg: `journalctl -t borg`


<details>
<summary> результат выполнения команды </summary>

```
root@client:~# journalctl -t borg
Mar 18 10:29:00 client borg[16107]: Remote: Warning: Permanently added '192.168.56.11' (ED25519) to the list of known hosts.
Mar 18 10:29:01 client borg[16107]: ------------------------------------------------------------------------------
Mar 18 10:29:01 client borg[16107]: Repository: ssh://borg@192.168.56.11/var/backup
Mar 18 10:29:01 client borg[16107]: Archive name: client-2025-03-18_10:29:00
Mar 18 10:29:01 client borg[16107]: Archive fingerprint: 0a95c2fe0c01a5947c15d31631dd32a5902dc6a70ad6531c95a564740b874e72
Mar 18 10:29:01 client borg[16107]: Time (start): Tue, 2025-03-18 10:29:01
Mar 18 10:29:01 client borg[16107]: Time (end):   Tue, 2025-03-18 10:29:01
Mar 18 10:29:01 client borg[16107]: Duration: 0.44 seconds
Mar 18 10:29:01 client borg[16107]: Number of files: 452
Mar 18 10:29:01 client borg[16107]: Utilization of max. archive size: 0%
Mar 18 10:29:01 client borg[16107]: ------------------------------------------------------------------------------
Mar 18 10:29:01 client borg[16107]:                        Original size      Compressed size    Deduplicated size
Mar 18 10:29:01 client borg[16107]: This archive:                1.52 MB            666.97 kB            665.25 kB
Mar 18 10:29:01 client borg[16107]: All archives:                1.52 MB            666.40 kB            715.23 kB
Mar 18 10:29:01 client borg[16107]:                        Unique chunks         Total chunks
Mar 18 10:29:01 client borg[16107]: Chunk index:                     433                  443
Mar 18 10:29:01 client borg[16107]: ------------------------------------------------------------------------------
Mar 18 10:29:02 client borg[16109]: Remote: Warning: Permanently added '192.168.56.11' (ED25519) to the list of known hosts.
Mar 18 10:29:04 client borg[16111]: Remote: Warning: Permanently added '192.168.56.11' (ED25519) to the list of known hosts.
Mar 18 10:34:08 client borg[16120]: Remote: Warning: Permanently added '192.168.56.11' (ED25519) to the list of known hosts.
Mar 18 10:34:09 client borg[16120]: ------------------------------------------------------------------------------
Mar 18 10:34:09 client borg[16120]: Repository: ssh://borg@192.168.56.11/var/backup
Mar 18 10:34:09 client borg[16120]: Archive name: client-2025-03-18_10:34:08
Mar 18 10:34:09 client borg[16120]: Archive fingerprint: 46271845421a8440ca4a2bf11abb93c96f8b5aaf0cb793f093e384d6eaeaf476
Mar 18 10:34:09 client borg[16120]: Time (start): Tue, 2025-03-18 10:34:09
Mar 18 10:34:09 client borg[16120]: Time (end):   Tue, 2025-03-18 10:34:09
Mar 18 10:34:09 client borg[16120]: Duration: 0.13 seconds
Mar 18 10:34:09 client borg[16120]: Number of files: 452
Mar 18 10:34:09 client borg[16120]: Utilization of max. archive size: 0%
Mar 18 10:34:09 client borg[16120]: ------------------------------------------------------------------------------
Mar 18 10:34:09 client borg[16120]:                        Original size      Compressed size    Deduplicated size
Mar 18 10:34:09 client borg[16120]: This archive:                1.52 MB            666.97 kB                568 B
Mar 18 10:34:09 client borg[16120]: All archives:                3.04 MB              1.33 MB            715.80 kB
Mar 18 10:34:09 client borg[16120]:                        Unique chunks         Total chunks
Mar 18 10:34:09 client borg[16120]: Chunk index:                     434                  886
Mar 18 10:34:09 client borg[16120]: ------------------------------------------------------------------------------
Mar 18 10:34:10 client borg[16122]: Remote: Warning: Permanently added '192.168.56.11' (ED25519) to the list of known hosts.
Mar 18 10:34:12 client borg[16124]: Remote: Warning: Permanently added '192.168.56.11' (ED25519) to the list of known hosts.
Mar 18 10:39:28 client borg[16127]: Remote: Warning: Permanently added '192.168.56.11' (ED25519) to the list of known hosts.

------------------------------------------------------------------------------

Mar 18 16:13:41 client borg[16742]: Repository: ssh://borg@192.168.56.11/var/backup
Mar 18 16:13:41 client borg[16742]: Archive name: client-2025-03-18_16:13:40
Mar 18 16:13:41 client borg[16742]: Archive fingerprint: f5d6ade224e6f8a85dc7910281ffeac1e42669b35019963172bc516c933f43e5
Mar 18 16:13:41 client borg[16742]: Time (start): Tue, 2025-03-18 16:13:41
Mar 18 16:13:41 client borg[16742]: Time (end):   Tue, 2025-03-18 16:13:41
Mar 18 16:13:41 client borg[16742]: Duration: 0.14 seconds
Mar 18 16:13:41 client borg[16742]: Number of files: 452
Mar 18 16:13:41 client borg[16742]: Utilization of max. archive size: 0%
Mar 18 16:13:41 client borg[16742]: ------------------------------------------------------------------------------
Mar 18 16:13:41 client borg[16742]:                        Original size      Compressed size    Deduplicated size
Mar 18 16:13:41 client borg[16742]: This archive:                1.52 MB            666.97 kB                568 B
Mar 18 16:13:41 client borg[16742]: All archives:                4.57 MB              2.00 MB            716.37 kB
Mar 18 16:13:41 client borg[16742]:                        Unique chunks         Total chunks
Mar 18 16:13:41 client borg[16742]: Chunk index:                     435                 1329
Mar 18 16:13:41 client borg[16742]: ------------------------------------------------------------------------------
Mar 18 16:13:42 client borg[16744]: Remote: Warning: Permanently added '192.168.56.11' (ED25519) to the list of known hosts.
Mar 18 16:13:44 client borg[16746]: Remote: Warning: Permanently added '192.168.56.11' (ED25519) to the list of known hosts.
Mar 18 16:18:54 client borg[16755]: Remote: Warning: Permanently added '192.168.56.11' (ED25519) to the list of known hosts.
Mar 18 16:18:55 client borg[16755]: ------------------------------------------------------------------------------
Mar 18 16:18:55 client borg[16755]: Repository: ssh://borg@192.168.56.11/var/backup
Mar 18 16:18:55 client borg[16755]: Archive name: client-2025-03-18_16:18:54
Mar 18 16:18:55 client borg[16755]: Archive fingerprint: af256e23a06b67174f57ae10db91e1094d52a1850eafd7107b47b5eea9180c23
Mar 18 16:18:55 client borg[16755]: Time (start): Tue, 2025-03-18 16:18:55
Mar 18 16:18:55 client borg[16755]: Time (end):   Tue, 2025-03-18 16:18:55
Mar 18 16:18:55 client borg[16755]: Duration: 0.14 seconds
Mar 18 16:18:55 client borg[16755]: Number of files: 452
Mar 18 16:18:55 client borg[16755]: Utilization of max. archive size: 0%
Mar 18 16:18:55 client borg[16755]: ------------------------------------------------------------------------------
Mar 18 16:18:55 client borg[16755]:                        Original size      Compressed size    Deduplicated size
Mar 18 16:18:55 client borg[16755]: This archive:                1.52 MB            666.97 kB                568 B
Mar 18 16:18:55 client borg[16755]: All archives:                4.57 MB              2.00 MB            716.37 kB
Mar 18 16:18:55 client borg[16755]:                        Unique chunks         Total chunks
Mar 18 16:18:55 client borg[16755]: Chunk index:                     435                 1329
Mar 18 16:18:55 client borg[16755]: ------------------------------------------------------------------------------
Mar 18 16:18:56 client borg[16757]: Remote: Warning: Permanently added '192.168.56.11' (ED25519) to the list of known hosts.
Mar 18 16:18:59 client borg[16759]: Remote: Warning: Permanently added '192.168.56.11' (ED25519) to the list of known hosts.
Mar 18 16:24:15 client borg[16762]: Remote: Warning: Permanently added '192.168.56.11' (ED25519) to the list of known hosts.
Mar 18 16:24:16 client borg[16762]: ------------------------------------------------------------------------------
Mar 18 16:24:16 client borg[16762]: Repository: ssh://borg@192.168.56.11/var/backup
Mar 18 16:24:16 client borg[16762]: Archive name: client-2025-03-18_16:24:15
Mar 18 16:24:16 client borg[16762]: Archive fingerprint: 821a0c567f66b61eae6262d17c3cd16d1417634af5d5fb08e976083b3fe230af
Mar 18 16:24:16 client borg[16762]: Time (start): Tue, 2025-03-18 16:24:16
Mar 18 16:24:16 client borg[16762]: Time (end):   Tue, 2025-03-18 16:24:16
Mar 18 16:24:16 client borg[16762]: Duration: 0.14 seconds
Mar 18 16:24:16 client borg[16762]: Number of files: 452
Mar 18 16:24:16 client borg[16762]: Utilization of max. archive size: 0%
Mar 18 16:24:16 client borg[16762]: ------------------------------------------------------------------------------
Mar 18 16:24:16 client borg[16762]:                        Original size      Compressed size    Deduplicated size
Mar 18 16:24:16 client borg[16762]: This archive:                1.52 MB            666.97 kB                568 B
Mar 18 16:24:16 client borg[16762]: All archives:                4.57 MB              2.00 MB            716.37 kB
Mar 18 16:24:16 client borg[16762]:                        Unique chunks         Total chunks
Mar 18 16:24:16 client borg[16762]: Chunk index:                     435                 1329
Mar 18 16:24:16 client borg[16762]: ------------------------------------------------------------------------------
Mar 18 16:24:17 client borg[16764]: Remote: Warning: Permanently added '192.168.56.11' (ED25519) to the list of known hosts.
Mar 18 16:24:19 client borg[16767]: Remote: Warning: Permanently added '192.168.56.11' (ED25519) to the list of known hosts.
Mar 18 16:29:35 client borg[16803]: Remote: Warning: Permanently added '192.168.56.11' (ED25519) to the list of known hosts.
Mar 18 16:29:37 client borg[16803]: ------------------------------------------------------------------------------
Mar 18 16:29:37 client borg[16803]: Repository: ssh://borg@192.168.56.11/var/backup
Mar 18 16:29:37 client borg[16803]: Archive name: client-2025-03-18_16:29:35
Mar 18 16:29:37 client borg[16803]: Archive fingerprint: 6bc2586fd4f38167331e92f949b69fdfeecbaae7ae6912b81f43e9f49b8d1cc3
Mar 18 16:29:37 client borg[16803]: Time (start): Tue, 2025-03-18 16:29:37
Mar 18 16:29:37 client borg[16803]: Time (end):   Tue, 2025-03-18 16:29:37
Mar 18 16:29:37 client borg[16803]: Duration: 0.14 seconds
Mar 18 16:29:37 client borg[16803]: Number of files: 452
Mar 18 16:29:37 client borg[16803]: Utilization of max. archive size: 0%
Mar 18 16:29:37 client borg[16803]: ------------------------------------------------------------------------------
Mar 18 16:29:37 client borg[16803]:                        Original size      Compressed size    Deduplicated size
Mar 18 16:29:37 client borg[16803]: This archive:                1.52 MB            666.97 kB                568 B
Mar 18 16:29:37 client borg[16803]: All archives:                4.57 MB              2.00 MB            716.37 kB
Mar 18 16:29:37 client borg[16803]:                        Unique chunks         Total chunks
Mar 18 16:29:37 client borg[16803]: Chunk index:                     435                 1329
Mar 18 16:29:37 client borg[16803]: ------------------------------------------------------------------------------
Mar 18 16:29:38 client borg[16805]: Remote: Warning: Permanently added '192.168.56.11' (ED25519) to the list of known hosts.
Mar 18 16:29:40 client borg[16807]: Remote: Warning: Permanently added '192.168.56.11' (ED25519) to the list of known hosts.
Mar 18 16:34:54 client borg[16843]: Remote: Warning: Permanently added '192.168.56.11' (ED25519) to the list of known hosts.
Mar 18 16:34:55 client borg[16843]: ------------------------------------------------------------------------------
Mar 18 16:34:55 client borg[16843]: Repository: ssh://borg@192.168.56.11/var/backup
Mar 18 16:34:55 client borg[16843]: Archive name: client-2025-03-18_16:34:54
Mar 18 16:34:55 client borg[16843]: Archive fingerprint: 4807f48cf68966024c6c1e24901042be161e4f1761795ce2f1b03b3279c08eb4
Mar 18 16:34:55 client borg[16843]: Time (start): Tue, 2025-03-18 16:34:55
Mar 18 16:34:55 client borg[16843]: Time (end):   Tue, 2025-03-18 16:34:55
Mar 18 16:34:55 client borg[16843]: Duration: 0.15 seconds
Mar 18 16:34:55 client borg[16843]: Number of files: 452
Mar 18 16:34:55 client borg[16843]: Utilization of max. archive size: 0%
Mar 18 16:34:55 client borg[16843]: ------------------------------------------------------------------------------
Mar 18 16:34:55 client borg[16843]:                        Original size      Compressed size    Deduplicated size
Mar 18 16:34:55 client borg[16843]: This archive:                1.52 MB            666.97 kB                568 B
Mar 18 16:34:55 client borg[16843]: All archives:                4.57 MB              2.00 MB            716.37 kB
Mar 18 16:34:55 client borg[16843]:                        Unique chunks         Total chunks
Mar 18 16:34:55 client borg[16843]: Chunk index:                     435                 1329
Mar 18 16:34:55 client borg[16843]: ------------------------------------------------------------------------------
Mar 18 16:34:56 client borg[16845]: Remote: Warning: Permanently added '192.168.56.11' (ED25519) to the list of known hosts.
Mar 18 16:34:58 client borg[16847]: Remote: Warning: Permanently added '192.168.56.11' (ED25519) to the list of known hosts.
Mar 18 16:40:14 client borg[16855]: Remote: Warning: Permanently added '192.168.56.11' (ED25519) to the list of known hosts.
Mar 18 16:40:15 client borg[16855]: ------------------------------------------------------------------------------
Mar 18 16:40:15 client borg[16855]: Repository: ssh://borg@192.168.56.11/var/backup
Mar 18 16:40:15 client borg[16855]: Archive name: client-2025-03-18_16:40:14
Mar 18 16:40:15 client borg[16855]: Archive fingerprint: ca1776ea622523c40977387e669cb53ba570788750090aec558081fc13cd8d6f
Mar 18 16:40:15 client borg[16855]: Time (start): Tue, 2025-03-18 16:40:15
Mar 18 16:40:15 client borg[16855]: Time (end):   Tue, 2025-03-18 16:40:15
Mar 18 16:40:15 client borg[16855]: Duration: 0.14 seconds
Mar 18 16:40:15 client borg[16855]: Number of files: 452
Mar 18 16:40:15 client borg[16855]: Utilization of max. archive size: 0%
Mar 18 16:40:15 client borg[16855]: ------------------------------------------------------------------------------
Mar 18 16:40:15 client borg[16855]:                        Original size      Compressed size    Deduplicated size
Mar 18 16:40:15 client borg[16855]: This archive:                1.52 MB            666.97 kB                568 B
Mar 18 16:40:15 client borg[16855]: All archives:                4.57 MB              2.00 MB            716.37 kB
Mar 18 16:40:15 client borg[16855]:                        Unique chunks         Total chunks
Mar 18 16:40:15 client borg[16855]: Chunk index:                     435                 1329
Mar 18 16:40:15 client borg[16855]: ------------------------------------------------------------------------------
Mar 18 16:40:16 client borg[16857]: Remote: Warning: Permanently added '192.168.56.11' (ED25519) to the list of known hosts.
Mar 18 16:40:18 client borg[16860]: Remote: Warning: Permanently added '192.168.56.11' (ED25519) to the list of known hosts.
Mar 18 16:45:33 client borg[16868]: Remote: Warning: Permanently added '192.168.56.11' (ED25519) to the list of known hosts.
Mar 18 16:45:35 client borg[16868]: ------------------------------------------------------------------------------
Mar 18 16:45:35 client borg[16868]: Repository: ssh://borg@192.168.56.11/var/backup
Mar 18 16:45:35 client borg[16868]: Archive name: client-2025-03-18_16:45:33
Mar 18 16:45:35 client borg[16868]: Archive fingerprint: 36d6150f000e2c5320ffb85d7a95276159b51a4fb0baa3ed94910a79f21618fc
Mar 18 16:45:35 client borg[16868]: Time (start): Tue, 2025-03-18 16:45:35
Mar 18 16:45:35 client borg[16868]: Time (end):   Tue, 2025-03-18 16:45:35
Mar 18 16:45:35 client borg[16868]: Duration: 0.14 seconds
Mar 18 16:45:35 client borg[16868]: Number of files: 452
Mar 18 16:45:35 client borg[16868]: Utilization of max. archive size: 0%
Mar 18 16:45:35 client borg[16868]: ------------------------------------------------------------------------------
Mar 18 16:45:35 client borg[16868]:                        Original size      Compressed size    Deduplicated size
Mar 18 16:45:35 client borg[16868]: This archive:                1.52 MB            666.97 kB                568 B
Mar 18 16:45:35 client borg[16868]: All archives:                4.57 MB              2.00 MB            716.37 kB
Mar 18 16:45:35 client borg[16868]:                        Unique chunks         Total chunks
Mar 18 16:45:35 client borg[16868]: Chunk index:                     435                 1329
Mar 18 16:45:35 client borg[16868]: ------------------------------------------------------------------------------
Mar 18 16:45:36 client borg[16872]: Remote: Warning: Permanently added '192.168.56.11' (ED25519) to the list of known hosts.
Mar 18 16:45:38 client borg[16872]: Failed to create/acquire the lock /var/backup/lock (timeout).
Mar 18 16:50:54 client borg[16889]: Remote: Warning: Permanently added '192.168.56.11' (ED25519) to the list of known hosts.
Mar 18 16:50:55 client borg[16889]: ------------------------------------------------------------------------------
Mar 18 16:50:55 client borg[16889]: Repository: ssh://borg@192.168.56.11/var/backup
Mar 18 16:50:55 client borg[16889]: Archive name: client-2025-03-18_16:50:54
Mar 18 16:50:55 client borg[16889]: Archive fingerprint: 53f2196e51b7af3ab09516be7b988c658f2217b367a0b0f38ba759c5cf6806e2
Mar 18 16:50:55 client borg[16889]: Time (start): Tue, 2025-03-18 16:50:55
Mar 18 16:50:55 client borg[16889]: Time (end):   Tue, 2025-03-18 16:50:55
Mar 18 16:50:55 client borg[16889]: Duration: 0.15 seconds
Mar 18 16:50:55 client borg[16889]: Number of files: 453
Mar 18 16:50:55 client borg[16889]: Utilization of max. archive size: 0%
Mar 18 16:50:55 client borg[16889]: ------------------------------------------------------------------------------
Mar 18 16:50:55 client borg[16889]:                        Original size      Compressed size    Deduplicated size
Mar 18 16:50:55 client borg[16889]: This archive:                1.52 MB            666.97 kB                568 B
Mar 18 16:50:55 client borg[16889]: All archives:                6.09 MB              2.67 MB            766.98 kB
Mar 18 16:50:55 client borg[16889]:                        Unique chunks         Total chunks
Mar 18 16:50:55 client borg[16889]: Chunk index:                     437                 1772
Mar 18 16:50:55 client borg[16889]: ------------------------------------------------------------------------------
Mar 18 16:50:56 client borg[16891]: Remote: Warning: Permanently added '192.168.56.11' (ED25519) to the list of known hosts.
Mar 18 16:50:59 client borg[16893]: Remote: Warning: Permanently added '192.168.56.11' (ED25519) to the list of known hosts.

```
</details>
    
Видим, что каждые 5 минут создавались архивы.

Посмотрим содержимое архива: `borg list borg@192.168.56.11:/var/backup::client-2025-03-18_17:01:34`
* Пароль все тот же ;-) : **Otus1234** 
<details>
<summary> результат выполнения команды </summary>

```
root@client:~# borg list borg@192.168.56.11:/var/backup::client-2025-03-18_17:01:34
Remote: Warning: Permanently added '192.168.56.11' (ED25519) to the list of known hosts.
Enter passphrase for key ssh://borg@192.168.56.11/var/backup:
drwxr-xr-x root   root          0 Tue, 2025-03-18 16:49:20 etc
-rw-r--r-- root   root       3040 Thu, 2023-05-25 15:54:35 etc/adduser.conf
-rw-r--r-- root   root       1706 Thu, 2023-05-25 15:54:35 etc/deluser.conf
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:34:04 etc/apt
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:34:19 etc/apt/apt.conf.d
-rw-r--r-- root   root        399 Thu, 2023-05-25 14:11:37 etc/apt/apt.conf.d/01autoremove
-rw-r--r-- root   root        182 Sun, 2023-01-08 21:50:51 etc/apt/apt.conf.d/70debconf
-rw-r--r-- root   root        307 Sun, 2021-03-28 11:06:44 etc/apt/apt.conf.d/20listchanges
-rw-r--r-- root   root         80 Sat, 2022-12-31 20:59:00 etc/apt/apt.conf.d/20auto-upgrades
-rw-r--r-- root   root       7338 Sat, 2022-12-31 20:59:00 etc/apt/apt.conf.d/50unattended-upgrades
drwxr-xr-x root   root          0 Thu, 2023-05-25 14:11:37 etc/apt/auth.conf.d
drwxr-xr-x root   root          0 Thu, 2023-05-25 14:11:37 etc/apt/keyrings
drwxr-xr-x root   root          0 Thu, 2023-05-25 14:11:37 etc/apt/preferences.d
drwxr-xr-x root   root          0 Thu, 2023-05-25 14:11:37 etc/apt/sources.list.d
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:33:36 etc/apt/trusted.gpg.d
-rw-r--r-- root   root      11861 Sun, 2023-07-30 19:30:54 etc/apt/trusted.gpg.d/debian-archive-bookworm-automatic.asc
-rw-r--r-- root   root      11873 Sun, 2023-07-30 19:30:54 etc/apt/trusted.gpg.d/debian-archive-bookworm-security-automatic.asc
-rw-r--r-- root   root        461 Sun, 2023-07-30 19:30:54 etc/apt/trusted.gpg.d/debian-archive-bookworm-stable.asc
-rw-r--r-- root   root      11861 Sun, 2023-07-30 19:30:54 etc/apt/trusted.gpg.d/debian-archive-bullseye-automatic.asc
-rw-r--r-- root   root      11873 Sun, 2023-07-30 19:30:54 etc/apt/trusted.gpg.d/debian-archive-bullseye-security-automatic.asc
-rw-r--r-- root   root       3403 Sun, 2023-07-30 19:30:54 etc/apt/trusted.gpg.d/debian-archive-bullseye-stable.asc
-rw-r--r-- root   root      11093 Sun, 2023-07-30 19:30:54 etc/apt/trusted.gpg.d/debian-archive-buster-automatic.asc
-rw-r--r-- root   root      11105 Sun, 2023-07-30 19:30:54 etc/apt/trusted.gpg.d/debian-archive-buster-security-automatic.asc
-rw-r--r-- root   root       1704 Sun, 2023-07-30 19:30:54 etc/apt/trusted.gpg.d/debian-archive-buster-stable.asc
-rw-r--r-- root   root        482 Thu, 2024-09-05 04:34:56 etc/apt/sources.list
drwxr-xr-x root   root          0 Sun, 2021-03-28 11:06:44 etc/apt/listchanges.conf.d
-rw-r--r-- root   root        150 Thu, 2024-09-05 04:34:04 etc/apt/listchanges.conf
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:34:00 etc/cron.daily
-rwxr-xr-x root   root       1478 Thu, 2023-05-25 14:11:37 etc/cron.daily/apt-compat
-rwxr-xr-x root   root        123 Mon, 2023-03-27 00:41:09 etc/cron.daily/dpkg
-rw-r--r-- root   root        102 Thu, 2023-03-02 07:33:55 etc/cron.daily/.placeholder
-rwxr-xr-x root   root        377 Wed, 2022-12-14 18:16:50 etc/cron.daily/logrotate
-rwxr-xr-x root   root       1395 Sun, 2023-03-12 22:23:59 etc/cron.daily/man-db
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:34:12 etc/kernel
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:34:37 etc/kernel/postinst.d
-rwxr-xr-x root   root        863 Mon, 2020-08-31 23:59:17 etc/kernel/postinst.d/initramfs-tools
-rwxr-xr-x root   root        646 Mon, 2023-10-02 14:11:34 etc/kernel/postinst.d/zz-update-grub
-rwxr-xr-x root   root        364 Sat, 2022-12-31 20:59:00 etc/kernel/postinst.d/unattended-upgrades
drwxr-xr-x root   root          0 Sun, 2024-08-25 17:35:39 etc/kernel/install.d
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:34:37 etc/kernel/postrm.d
-rwxr-xr-x root   root        816 Mon, 2020-08-31 23:59:17 etc/kernel/postrm.d/initramfs-tools
-rwxr-xr-x root   root        646 Mon, 2023-10-02 14:11:34 etc/kernel/postrm.d/zz-update-grub
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:34:18 etc/logrotate.d
-rw-r--r-- root   root        173 Thu, 2023-05-25 14:11:37 etc/logrotate.d/apt
-rw-r--r-- root   root        120 Tue, 2023-02-14 22:56:51 etc/logrotate.d/alternatives
-rw-r--r-- root   root        112 Tue, 2023-02-14 22:56:51 etc/logrotate.d/dpkg
-rw-r--r-- root   root        130 Mon, 2019-10-14 12:10:31 etc/logrotate.d/btmp
-rw-r--r-- root   root        145 Mon, 2019-10-14 12:10:31 etc/logrotate.d/wtmp
-rw-r--r-- root   root        160 Mon, 2023-05-08 20:05:00 etc/logrotate.d/chrony
-rw-r--r-- root   root        248 Wed, 2023-02-22 19:43:00 etc/logrotate.d/rsyslog
-rw-r--r-- root   root        235 Sat, 2022-12-31 20:59:00 etc/logrotate.d/unattended-upgrades
-rw-r--r-- root   root        681 Tue, 2023-01-17 22:34:38 etc/xattr.conf
-rw-r--r-- root   root        191 Thu, 2023-02-09 09:36:04 etc/libaudit.conf
-rw-r--r-- root   root          5 Wed, 2024-08-14 16:10:00 etc/debian_version
drwxr-xr-x root   root          0 Tue, 2025-03-18 10:27:46 etc/default
-rw-r--r-- root   root       1756 Tue, 2024-07-30 22:16:44 etc/default/nss
-rw-r--r-- root   root       1117 Fri, 2022-11-11 08:28:15 etc/default/useradd
-rw-r--r-- root   root         81 Thu, 2024-03-28 09:52:12 etc/default/hwclock
-rw-r--r-- root   root        955 Sun, 2022-07-17 08:49:40 etc/default/cron
-rw-r--r-- root   root        297 Sat, 2023-09-16 10:03:58 etc/default/dbus
-rw-r--r-- root   root       1032 Tue, 2021-09-14 13:27:00 etc/default/networking
-rw-r--r-- root   root         13 Thu, 2024-09-05 04:34:56 etc/default/locale
-rw-r--r-- root   root       1528 Thu, 2024-09-05 04:34:57 etc/default/grub
-rw-r--r-- root   root        255 Mon, 2023-05-08 20:05:00 etc/default/chrony
-rw-r--r-- root   root        133 Tue, 2023-12-19 12:55:09 etc/default/ssh
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:34:56 etc/default/grub.d
-rw-r--r-- root   root        274 Mon, 2023-10-02 14:11:34 etc/default/grub.d/init-select.cfg
-rw-r--r-- root   root        142 Thu, 2024-09-05 04:34:56 etc/default/grub.d/10_cloud_disk_scheduler.cfg
-rw-r--r-- root   root         15 Thu, 2024-09-05 04:34:56 etc/default/grub.d/15_timeout.cfg
drwxr-xr-x root   root          0 Tue, 2025-03-18 10:28:06 etc/dpkg
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:33:36 etc/dpkg/origins
-rw-r--r-- root   root         83 Sat, 2021-04-10 20:15:00 etc/dpkg/origins/debian
lrwxrwxrwx root   root          6 Thu, 2024-09-05 04:33:28 etc/dpkg/origins/default -> debian
-rw-r--r-- root   root        446 Tue, 2023-02-14 22:56:51 etc/dpkg/dpkg.cfg
drwxr-xr-x root   root          0 Thu, 2023-05-11 02:04:01 etc/dpkg/dpkg.cfg.d
-rw-r--r-- root   root        260 Tue, 2023-02-14 22:56:51 etc/dpkg/shlibs.default
-rw-r--r-- root   root        253 Tue, 2023-02-14 22:56:51 etc/dpkg/shlibs.override
-rw-r--r-- root   root          9 Mon, 2006-08-07 17:14:09 etc/host.conf
-rw-r--r-- root   root         27 Wed, 2024-08-14 16:10:00 etc/issue
-rw-r--r-- root   root         20 Wed, 2024-08-14 16:10:00 etc/issue.net
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:33:58 etc/profile.d
-rw-r--r-- root   root        726 Sun, 2022-04-03 12:14:34 etc/profile.d/bash_completion.sh
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:33:36 etc/skel
-rw-r--r-- root   root        220 Fri, 2024-03-29 19:40:10 etc/skel/.bash_logout
-rw-r--r-- root   root       3526 Fri, 2024-03-29 19:40:10 etc/skel/.bashrc
-rw-r--r-- root   root        807 Fri, 2024-03-29 19:40:10 etc/skel/.profile
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:34:18 etc/update-motd.d
-rwxr-xr-x root   root         23 Tue, 2017-04-04 12:00:00 etc/update-motd.d/10-uname
-rwxr-xr-x root   root        165 Sat, 2022-12-31 20:59:00 etc/update-motd.d/92-unattended-upgrades
-rw-r--r-- root   root       1994 Fri, 2024-03-29 19:40:10 etc/bash.bashrc
-rw-r--r-- root   root       2969 Sun, 2023-01-08 21:50:51 etc/debconf.conf
drwxr-xr-x root   root          0 Tue, 2025-03-18 10:28:06 etc/alternatives
-rw-r--r-- root   root        100 Thu, 2023-05-11 02:04:01 etc/alternatives/README
lrwxrwxrwx root   root         38 Fri, 2024-03-29 19:40:10 etc/alternatives/builtins.7.gz -> /usr/share/man/man7/bash-builtins.7.gz
lrwxrwxrwx root   root         13 Fri, 2022-06-17 15:35:26 etc/alternatives/awk -> /usr/bin/mawk
lrwxrwxrwx root   root         29 Fri, 2022-06-17 15:35:26 etc/alternatives/awk.1.gz -> /usr/share/man/man1/mawk.1.gz
lrwxrwxrwx root   root         13 Fri, 2022-06-17 15:35:26 etc/alternatives/nawk -> /usr/bin/mawk
lrwxrwxrwx root   root         29 Fri, 2022-06-17 15:35:26 etc/alternatives/nawk.1.gz -> /usr/share/man/man1/mawk.1.gz
lrwxrwxrwx root   root         26 Fri, 2023-07-28 23:46:35 etc/alternatives/which -> /usr/bin/which.debianutils
lrwxrwxrwx root   root         17 Sat, 2024-01-20 09:27:07 etc/alternatives/rmt -> /usr/sbin/rmt-tar
lrwxrwxrwx root   root         32 Sat, 2024-01-20 09:27:07 etc/alternatives/rmt.8.gz -> /usr/share/man/man8/rmt-tar.8.gz
lrwxrwxrwx root   root         42 Fri, 2023-07-28 23:46:35 etc/alternatives/which.1.gz -> /usr/share/man/man1/which.debianutils.1.gz
lrwxrwxrwx root   root         45 Fri, 2023-07-28 23:46:35 etc/alternatives/which.de1.gz -> /usr/share/man/de/man1/which.debianutils.1.gz
lrwxrwxrwx root   root         45 Fri, 2023-07-28 23:46:35 etc/alternatives/which.es1.gz -> /usr/share/man/es/man1/which.debianutils.1.gz
lrwxrwxrwx root   root         45 Fri, 2023-07-28 23:46:35 etc/alternatives/which.fr1.gz -> /usr/share/man/fr/man1/which.debianutils.1.gz
lrwxrwxrwx root   root         45 Fri, 2023-07-28 23:46:35 etc/alternatives/which.it1.gz -> /usr/share/man/it/man1/which.debianutils.1.gz
lrwxrwxrwx root   root         45 Fri, 2023-07-28 23:46:35 etc/alternatives/which.ja1.gz -> /usr/share/man/ja/man1/which.debianutils.1.gz
lrwxrwxrwx root   root         45 Fri, 2023-07-28 23:46:35 etc/alternatives/which.pl1.gz -> /usr/share/man/pl/man1/which.debianutils.1.gz
lrwxrwxrwx root   root         45 Fri, 2023-07-28 23:46:35 etc/alternatives/which.sl1.gz -> /usr/share/man/sl/man1/which.debianutils.1.gz
lrwxrwxrwx root   root         11 Wed, 2022-09-14 19:45:55 etc/alternatives/mt -> /bin/mt-gnu
lrwxrwxrwx root   root         31 Wed, 2022-09-14 19:45:55 etc/alternatives/mt.1.gz -> /usr/share/man/man1/mt-gnu.1.gz
lrwxrwxrwx root   root         19 Fri, 2021-08-20 11:41:53 etc/alternatives/nc -> /bin/nc.traditional
lrwxrwxrwx root   root         19 Fri, 2021-08-20 11:41:53 etc/alternatives/netcat -> /bin/nc.traditional
lrwxrwxrwx root   root         39 Fri, 2021-08-20 11:41:53 etc/alternatives/nc.1.gz -> /usr/share/man/man1/nc.traditional.1.gz
lrwxrwxrwx root   root         39 Fri, 2021-08-20 11:41:53 etc/alternatives/netcat.1.gz -> /usr/share/man/man1/nc.traditional.1.gz
lrwxrwxrwx root   root         22 Sat, 2023-02-25 12:24:47 etc/alternatives/traceroute -> /usr/bin/traceroute.db
lrwxrwxrwx root   root         22 Sat, 2023-02-25 12:24:47 etc/alternatives/traceroute.sbin -> /usr/bin/traceroute.db
lrwxrwxrwx root   root         38 Sat, 2023-02-25 12:24:47 etc/alternatives/traceroute.1.gz -> /usr/share/man/man1/traceroute.db.1.gz
lrwxrwxrwx root   root         23 Sat, 2023-02-25 12:24:47 etc/alternatives/traceroute6 -> /usr/bin/traceroute6.db
lrwxrwxrwx root   root         39 Sat, 2023-02-25 12:24:47 etc/alternatives/traceroute6.1.gz -> /usr/share/man/man1/traceroute6.db.1.gz
lrwxrwxrwx root   root         15 Sat, 2023-02-25 12:24:47 etc/alternatives/lft -> /usr/bin/lft.db
lrwxrwxrwx root   root         31 Sat, 2023-02-25 12:24:47 etc/alternatives/lft.1.gz -> /usr/share/man/man1/lft.db.1.gz
lrwxrwxrwx root   root         22 Sat, 2023-02-25 12:24:47 etc/alternatives/traceproto -> /usr/bin/traceproto.db
lrwxrwxrwx root   root         38 Sat, 2023-02-25 12:24:47 etc/alternatives/traceproto.1.gz -> /usr/share/man/man1/traceproto.db.1.gz
lrwxrwxrwx root   root         26 Sat, 2023-02-25 12:24:47 etc/alternatives/tcptraceroute -> /usr/sbin/tcptraceroute.db
lrwxrwxrwx root   root         41 Sat, 2023-02-25 12:24:47 etc/alternatives/tcptraceroute.8.gz -> /usr/share/man/man8/tcptraceroute.db.8.gz
lrwxrwxrwx root   root         13 Thu, 2024-09-05 04:33:57 etc/alternatives/pager -> /usr/bin/less
lrwxrwxrwx root   root         29 Thu, 2024-09-05 04:33:57 etc/alternatives/pager.1.gz -> /usr/share/man/man1/less.1.gz
lrwxrwxrwx root   root         11 Sun, 2023-02-12 20:22:50 etc/alternatives/lzma -> /usr/bin/xz
lrwxrwxrwx root   root         27 Sun, 2023-02-12 20:22:50 etc/alternatives/lzma.1.gz -> /usr/share/man/man1/xz.1.gz
lrwxrwxrwx root   root         13 Sun, 2023-02-12 20:22:50 etc/alternatives/unlzma -> /usr/bin/unxz
lrwxrwxrwx root   root         29 Sun, 2023-02-12 20:22:50 etc/alternatives/unlzma.1.gz -> /usr/share/man/man1/unxz.1.gz
lrwxrwxrwx root   root         14 Sun, 2023-02-12 20:22:50 etc/alternatives/lzcat -> /usr/bin/xzcat
lrwxrwxrwx root   root         30 Sun, 2023-02-12 20:22:50 etc/alternatives/lzcat.1.gz -> /usr/share/man/man1/xzcat.1.gz
lrwxrwxrwx root   root         15 Sun, 2023-02-12 20:22:50 etc/alternatives/lzmore -> /usr/bin/xzmore
lrwxrwxrwx root   root         31 Sun, 2023-02-12 20:22:50 etc/alternatives/lzmore.1.gz -> /usr/share/man/man1/xzmore.1.gz
lrwxrwxrwx root   root         15 Sun, 2023-02-12 20:22:50 etc/alternatives/lzless -> /usr/bin/xzless
lrwxrwxrwx root   root         31 Sun, 2023-02-12 20:22:50 etc/alternatives/lzless.1.gz -> /usr/share/man/man1/xzless.1.gz
lrwxrwxrwx root   root         15 Sun, 2023-02-12 20:22:50 etc/alternatives/lzdiff -> /usr/bin/xzdiff
lrwxrwxrwx root   root         31 Sun, 2023-02-12 20:22:50 etc/alternatives/lzdiff.1.gz -> /usr/share/man/man1/xzdiff.1.gz
lrwxrwxrwx root   root         14 Sun, 2023-02-12 20:22:50 etc/alternatives/lzcmp -> /usr/bin/xzcmp
lrwxrwxrwx root   root         30 Sun, 2023-02-12 20:22:50 etc/alternatives/lzcmp.1.gz -> /usr/share/man/man1/xzcmp.1.gz
lrwxrwxrwx root   root         15 Sun, 2023-02-12 20:22:50 etc/alternatives/lzgrep -> /usr/bin/xzgrep
lrwxrwxrwx root   root         31 Sun, 2023-02-12 20:22:50 etc/alternatives/lzgrep.1.gz -> /usr/share/man/man1/xzgrep.1.gz
lrwxrwxrwx root   root         16 Sun, 2023-02-12 20:22:50 etc/alternatives/lzegrep -> /usr/bin/xzegrep
lrwxrwxrwx root   root         32 Sun, 2023-02-12 20:22:50 etc/alternatives/lzegrep.1.gz -> /usr/share/man/man1/xzegrep.1.gz
lrwxrwxrwx root   root         16 Sun, 2023-02-12 20:22:50 etc/alternatives/lzfgrep -> /usr/bin/xzfgrep
lrwxrwxrwx root   root         32 Sun, 2023-02-12 20:22:50 etc/alternatives/lzfgrep.1.gz -> /usr/share/man/man1/xzfgrep.1.gz
lrwxrwxrwx root   root          9 Mon, 2024-05-06 06:10:01 etc/alternatives/editor -> /bin/nano
lrwxrwxrwx root   root         29 Mon, 2024-05-06 06:10:01 etc/alternatives/editor.1.gz -> /usr/share/man/man1/nano.1.gz
lrwxrwxrwx root   root          9 Mon, 2024-05-06 06:10:01 etc/alternatives/pico -> /bin/nano
lrwxrwxrwx root   root         29 Mon, 2024-05-06 06:10:01 etc/alternatives/pico.1.gz -> /usr/share/man/man1/nano.1.gz
lrwxrwxrwx root   root         25 Wed, 2023-08-23 10:01:39 etc/alternatives/telnet -> /usr/bin/inetutils-telnet
lrwxrwxrwx root   root         41 Wed, 2023-08-23 10:01:39 etc/alternatives/telnet.1.gz -> /usr/share/man/man1/inetutils-telnet.1.gz
lrwxrwxrwx root   root         17 Thu, 2023-05-04 10:24:44 etc/alternatives/ex -> /usr/bin/vim.tiny
lrwxrwxrwx root   root         28 Thu, 2023-05-04 10:24:44 etc/alternatives/ex.1.gz -> /usr/share/man/man1/vim.1.gz
lrwxrwxrwx root   root         31 Thu, 2023-05-04 10:24:44 etc/alternatives/ex.da.1.gz -> /usr/share/man/da/man1/vim.1.gz
lrwxrwxrwx root   root         31 Thu, 2023-05-04 10:24:44 etc/alternatives/ex.de.1.gz -> /usr/share/man/de/man1/vim.1.gz
lrwxrwxrwx root   root         31 Thu, 2023-05-04 10:24:44 etc/alternatives/ex.fr.1.gz -> /usr/share/man/fr/man1/vim.1.gz
lrwxrwxrwx root   root         31 Thu, 2023-05-04 10:24:44 etc/alternatives/ex.it.1.gz -> /usr/share/man/it/man1/vim.1.gz
lrwxrwxrwx root   root         31 Thu, 2023-05-04 10:24:44 etc/alternatives/ex.ja.1.gz -> /usr/share/man/ja/man1/vim.1.gz
lrwxrwxrwx root   root         31 Thu, 2023-05-04 10:24:44 etc/alternatives/ex.pl.1.gz -> /usr/share/man/pl/man1/vim.1.gz
lrwxrwxrwx root   root         31 Thu, 2023-05-04 10:24:44 etc/alternatives/ex.ru.1.gz -> /usr/share/man/ru/man1/vim.1.gz
lrwxrwxrwx root   root         31 Thu, 2023-05-04 10:24:44 etc/alternatives/ex.tr.1.gz -> /usr/share/man/tr/man1/vim.1.gz
lrwxrwxrwx root   root         17 Thu, 2023-05-04 10:24:44 etc/alternatives/rview -> /usr/bin/vim.tiny
lrwxrwxrwx root   root         17 Thu, 2023-05-04 10:24:44 etc/alternatives/vi -> /usr/bin/vim.tiny
lrwxrwxrwx root   root         28 Thu, 2023-05-04 10:24:44 etc/alternatives/vi.1.gz -> /usr/share/man/man1/vim.1.gz
lrwxrwxrwx root   root         31 Thu, 2023-05-04 10:24:44 etc/alternatives/vi.da.1.gz -> /usr/share/man/da/man1/vim.1.gz
lrwxrwxrwx root   root         31 Thu, 2023-05-04 10:24:44 etc/alternatives/vi.de.1.gz -> /usr/share/man/de/man1/vim.1.gz
lrwxrwxrwx root   root         31 Thu, 2023-05-04 10:24:44 etc/alternatives/vi.fr.1.gz -> /usr/share/man/fr/man1/vim.1.gz
lrwxrwxrwx root   root         31 Thu, 2023-05-04 10:24:44 etc/alternatives/vi.it.1.gz -> /usr/share/man/it/man1/vim.1.gz
lrwxrwxrwx root   root         31 Thu, 2023-05-04 10:24:44 etc/alternatives/vi.ja.1.gz -> /usr/share/man/ja/man1/vim.1.gz
lrwxrwxrwx root   root         31 Thu, 2023-05-04 10:24:44 etc/alternatives/vi.pl.1.gz -> /usr/share/man/pl/man1/vim.1.gz
lrwxrwxrwx root   root         31 Thu, 2023-05-04 10:24:44 etc/alternatives/vi.ru.1.gz -> /usr/share/man/ru/man1/vim.1.gz
lrwxrwxrwx root   root         31 Thu, 2023-05-04 10:24:44 etc/alternatives/vi.tr.1.gz -> /usr/share/man/tr/man1/vim.1.gz
lrwxrwxrwx root   root         17 Thu, 2023-05-04 10:24:44 etc/alternatives/view -> /usr/bin/vim.tiny
lrwxrwxrwx root   root         28 Thu, 2023-05-04 10:24:44 etc/alternatives/view.1.gz -> /usr/share/man/man1/vim.1.gz
lrwxrwxrwx root   root         31 Thu, 2023-05-04 10:24:44 etc/alternatives/view.da.1.gz -> /usr/share/man/da/man1/vim.1.gz
lrwxrwxrwx root   root         31 Thu, 2023-05-04 10:24:44 etc/alternatives/view.de.1.gz -> /usr/share/man/de/man1/vim.1.gz
lrwxrwxrwx root   root         31 Thu, 2023-05-04 10:24:44 etc/alternatives/view.fr.1.gz -> /usr/share/man/fr/man1/vim.1.gz
lrwxrwxrwx root   root         31 Thu, 2023-05-04 10:24:44 etc/alternatives/view.it.1.gz -> /usr/share/man/it/man1/vim.1.gz
lrwxrwxrwx root   root         31 Thu, 2023-05-04 10:24:44 etc/alternatives/view.ja.1.gz -> /usr/share/man/ja/man1/vim.1.gz
lrwxrwxrwx root   root         31 Thu, 2023-05-04 10:24:44 etc/alternatives/view.pl.1.gz -> /usr/share/man/pl/man1/vim.1.gz
lrwxrwxrwx root   root         31 Thu, 2023-05-04 10:24:44 etc/alternatives/view.ru.1.gz -> /usr/share/man/ru/man1/vim.1.gz
lrwxrwxrwx root   root         31 Thu, 2023-05-04 10:24:44 etc/alternatives/view.tr.1.gz -> /usr/share/man/tr/man1/vim.1.gz
lrwxrwxrwx root   root         20 Mon, 2021-11-29 13:07:27 etc/alternatives/open -> /usr/bin/run-mailcap
lrwxrwxrwx root   root         36 Mon, 2021-11-29 13:07:27 etc/alternatives/open.1.gz -> /usr/share/man/man1/run-mailcap.1.gz
lrwxrwxrwx root   root         23 Thu, 2024-09-05 04:34:19 etc/alternatives/ip6tables -> /usr/sbin/ip6tables-nft
lrwxrwxrwx root   root         31 Thu, 2024-09-05 04:34:19 etc/alternatives/ip6tables-restore -> /usr/sbin/ip6tables-nft-restore
lrwxrwxrwx root   root         28 Thu, 2024-09-05 04:34:19 etc/alternatives/ip6tables-save -> /usr/sbin/ip6tables-nft-save
lrwxrwxrwx root   root         23 Mon, 2023-01-16 13:44:37 etc/alternatives/arptables -> /usr/sbin/arptables-nft
lrwxrwxrwx root   root         31 Mon, 2023-01-16 13:44:37 etc/alternatives/arptables-restore -> /usr/sbin/arptables-nft-restore
lrwxrwxrwx root   root         28 Mon, 2023-01-16 13:44:37 etc/alternatives/arptables-save -> /usr/sbin/arptables-nft-save
lrwxrwxrwx root   root         22 Thu, 2024-09-05 04:34:19 etc/alternatives/iptables -> /usr/sbin/iptables-nft
lrwxrwxrwx root   root         30 Thu, 2024-09-05 04:34:19 etc/alternatives/iptables-restore -> /usr/sbin/iptables-nft-restore
lrwxrwxrwx root   root         27 Thu, 2024-09-05 04:34:19 etc/alternatives/iptables-save -> /usr/sbin/iptables-nft-save
lrwxrwxrwx root   root         22 Mon, 2023-01-16 13:44:37 etc/alternatives/ebtables -> /usr/sbin/ebtables-nft
lrwxrwxrwx root   root         30 Mon, 2023-01-16 13:44:37 etc/alternatives/ebtables-restore -> /usr/sbin/ebtables-nft-restore
lrwxrwxrwx root   root         27 Mon, 2023-01-16 13:44:37 etc/alternatives/ebtables-save -> /usr/sbin/ebtables-nft-save
lrwxrwxrwx root   root         24 Tue, 2022-10-18 14:52:33 etc/alternatives/pinentry -> /usr/bin/pinentry-curses
lrwxrwxrwx root   root         22 Sat, 2023-03-18 09:22:00 etc/alternatives/fakeroot -> /usr/bin/fakeroot-sysv
lrwxrwxrwx root   root         38 Sat, 2023-03-18 09:22:00 etc/alternatives/fakeroot.1.gz -> /usr/share/man/man1/fakeroot-sysv.1.gz
lrwxrwxrwx root   root         35 Sat, 2023-03-18 09:22:00 etc/alternatives/faked.1.gz -> /usr/share/man/man1/faked-sysv.1.gz
lrwxrwxrwx root   root         41 Sat, 2023-03-18 09:22:00 etc/alternatives/fakeroot.es.1.gz -> /usr/share/man/es/man1/fakeroot-sysv.1.gz
lrwxrwxrwx root   root         38 Sat, 2023-03-18 09:22:00 etc/alternatives/faked.es.1.gz -> /usr/share/man/es/man1/faked-sysv.1.gz
lrwxrwxrwx root   root         41 Sat, 2023-03-18 09:22:00 etc/alternatives/fakeroot.fr.1.gz -> /usr/share/man/fr/man1/fakeroot-sysv.1.gz
lrwxrwxrwx root   root         38 Sat, 2023-03-18 09:22:00 etc/alternatives/faked.fr.1.gz -> /usr/share/man/fr/man1/faked-sysv.1.gz
lrwxrwxrwx root   root         41 Sat, 2023-03-18 09:22:00 etc/alternatives/fakeroot.sv.1.gz -> /usr/share/man/sv/man1/fakeroot-sysv.1.gz
lrwxrwxrwx root   root         38 Sat, 2023-03-18 09:22:00 etc/alternatives/faked.sv.1.gz -> /usr/share/man/sv/man1/faked-sysv.1.gz
lrwxrwxrwx root   root         40 Tue, 2022-10-18 14:52:33 etc/alternatives/pinentry.1.gz -> /usr/share/man/man1/pinentry-curses.1.gz
lrwxrwxrwx root   root         12 Sun, 2023-01-08 09:05:59 etc/alternatives/cpp -> /usr/bin/cpp
lrwxrwxrwx root   root         12 Sun, 2023-01-08 09:05:59 etc/alternatives/cc -> /usr/bin/gcc
lrwxrwxrwx root   root         16 Tue, 2020-11-17 18:53:07 etc/alternatives/c89 -> /usr/bin/c89-gcc
lrwxrwxrwx root   root         32 Tue, 2020-11-17 18:53:07 etc/alternatives/c89.1.gz -> /usr/share/man/man1/c89-gcc.1.gz
lrwxrwxrwx root   root         16 Tue, 2020-11-17 18:53:07 etc/alternatives/c99 -> /usr/bin/c99-gcc
lrwxrwxrwx root   root         32 Sun, 2023-01-08 09:05:59 etc/alternatives/c99.1.gz -> /usr/share/man/man1/c99-gcc.1.gz
lrwxrwxrwx root   root         12 Sun, 2023-01-08 09:05:59 etc/alternatives/c++ -> /usr/bin/g++
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:33:47 etc/cron.d
-rw-r--r-- root   root        201 Sun, 2023-03-05 03:16:08 etc/cron.d/e2scrub_all
-rw-r--r-- root   root        102 Thu, 2023-03-02 07:33:55 etc/cron.d/.placeholder
-rw-r--r-- root   root        685 Sun, 2023-03-05 03:16:08 etc/e2scrub.conf
-rw-r--r-- root   root        782 Sun, 2023-03-05 03:16:08 etc/mke2fs.conf
-rw-r--r-- root   root        367 Wed, 2024-04-10 07:01:24 etc/bindresvport.blacklist
-rw-r--r-- root   root       2584 Fri, 2022-07-29 22:03:09 etc/gai.conf
-rw-r--r-- root   root         34 Wed, 2024-04-10 07:01:24 etc/ld.so.conf
drwxr-xr-x root   root          0 Tue, 2025-03-18 10:28:01 etc/ld.so.conf.d
-rw-r--r-- root   root         44 Wed, 2024-04-10 07:01:24 etc/ld.so.conf.d/libc.conf
-rw-r--r-- root   root        100 Thu, 2024-08-15 09:10:46 etc/ld.so.conf.d/x86_64-linux-gnu.conf
-rw-r--r-- root   root         38 Sat, 2023-03-18 09:22:00 etc/ld.so.conf.d/fakeroot-x86_64-linux-gnu.conf
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:34:19 etc/rc0.d
lrwxrwxrwx root   root         20 Thu, 2024-09-05 04:33:37 etc/rc0.d/K01hwclock.sh -> ../init.d/hwclock.sh
lrwxrwxrwx root   root         14 Thu, 2024-09-05 04:33:59 etc/rc0.d/K01udev -> ../init.d/udev
lrwxrwxrwx root   root         20 Thu, 2024-09-05 04:34:01 etc/rc0.d/K01networking -> ../init.d/networking
lrwxrwxrwx root   root         16 Thu, 2024-09-05 04:34:16 etc/rc0.d/K01chrony -> ../init.d/chrony
lrwxrwxrwx root   root         29 Thu, 2024-09-05 04:34:19 etc/rc0.d/K01unattended-upgrades -> ../init.d/unattended-upgrades
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:34:16 etc/rc1.d
lrwxrwxrwx root   root         16 Thu, 2024-09-05 04:34:16 etc/rc1.d/K01chrony -> ../init.d/chrony
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:34:19 etc/rc2.d
lrwxrwxrwx root   root         14 Thu, 2024-09-05 04:33:59 etc/rc2.d/S01cron -> ../init.d/cron
lrwxrwxrwx root   root         14 Thu, 2024-09-05 04:33:59 etc/rc2.d/S01dbus -> ../init.d/dbus
lrwxrwxrwx root   root         16 Thu, 2024-09-05 04:34:16 etc/rc2.d/S01chrony -> ../init.d/chrony
lrwxrwxrwx root   root         14 Thu, 2024-09-05 04:34:17 etc/rc2.d/S01sudo -> ../init.d/sudo
lrwxrwxrwx root   root         13 Thu, 2024-09-05 04:34:18 etc/rc2.d/S01ssh -> ../init.d/ssh
lrwxrwxrwx root   root         29 Thu, 2024-09-05 04:34:19 etc/rc2.d/S01unattended-upgrades -> ../init.d/unattended-upgrades
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:34:19 etc/rc3.d
lrwxrwxrwx root   root         14 Thu, 2024-09-05 04:33:59 etc/rc3.d/S01cron -> ../init.d/cron
lrwxrwxrwx root   root         14 Thu, 2024-09-05 04:33:59 etc/rc3.d/S01dbus -> ../init.d/dbus
lrwxrwxrwx root   root         16 Thu, 2024-09-05 04:34:16 etc/rc3.d/S01chrony -> ../init.d/chrony
lrwxrwxrwx root   root         14 Thu, 2024-09-05 04:34:17 etc/rc3.d/S01sudo -> ../init.d/sudo
lrwxrwxrwx root   root         13 Thu, 2024-09-05 04:34:18 etc/rc3.d/S01ssh -> ../init.d/ssh
lrwxrwxrwx root   root         29 Thu, 2024-09-05 04:34:19 etc/rc3.d/S01unattended-upgrades -> ../init.d/unattended-upgrades
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:34:19 etc/rc4.d
lrwxrwxrwx root   root         14 Thu, 2024-09-05 04:33:59 etc/rc4.d/S01cron -> ../init.d/cron
lrwxrwxrwx root   root         14 Thu, 2024-09-05 04:33:59 etc/rc4.d/S01dbus -> ../init.d/dbus
lrwxrwxrwx root   root         16 Thu, 2024-09-05 04:34:16 etc/rc4.d/S01chrony -> ../init.d/chrony
lrwxrwxrwx root   root         14 Thu, 2024-09-05 04:34:17 etc/rc4.d/S01sudo -> ../init.d/sudo
lrwxrwxrwx root   root         13 Thu, 2024-09-05 04:34:18 etc/rc4.d/S01ssh -> ../init.d/ssh
lrwxrwxrwx root   root         29 Thu, 2024-09-05 04:34:19 etc/rc4.d/S01unattended-upgrades -> ../init.d/unattended-upgrades
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:34:19 etc/rc5.d
lrwxrwxrwx root   root         14 Thu, 2024-09-05 04:33:59 etc/rc5.d/S01cron -> ../init.d/cron
lrwxrwxrwx root   root         14 Thu, 2024-09-05 04:33:59 etc/rc5.d/S01dbus -> ../init.d/dbus
lrwxrwxrwx root   root         16 Thu, 2024-09-05 04:34:16 etc/rc5.d/S01chrony -> ../init.d/chrony
lrwxrwxrwx root   root         14 Thu, 2024-09-05 04:34:17 etc/rc5.d/S01sudo -> ../init.d/sudo
lrwxrwxrwx root   root         13 Thu, 2024-09-05 04:34:18 etc/rc5.d/S01ssh -> ../init.d/ssh
lrwxrwxrwx root   root         29 Thu, 2024-09-05 04:34:19 etc/rc5.d/S01unattended-upgrades -> ../init.d/unattended-upgrades
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:34:19 etc/rc6.d
lrwxrwxrwx root   root         20 Thu, 2024-09-05 04:33:37 etc/rc6.d/K01hwclock.sh -> ../init.d/hwclock.sh
lrwxrwxrwx root   root         14 Thu, 2024-09-05 04:33:59 etc/rc6.d/K01udev -> ../init.d/udev
lrwxrwxrwx root   root         20 Thu, 2024-09-05 04:34:01 etc/rc6.d/K01networking -> ../init.d/networking
lrwxrwxrwx root   root         16 Thu, 2024-09-05 04:34:16 etc/rc6.d/K01chrony -> ../init.d/chrony
lrwxrwxrwx root   root         29 Thu, 2024-09-05 04:34:19 etc/rc6.d/K01unattended-upgrades -> ../init.d/unattended-upgrades
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:34:17 etc/rcS.d
lrwxrwxrwx root   root         20 Thu, 2024-09-05 04:33:37 etc/rcS.d/S01hwclock.sh -> ../init.d/hwclock.sh
lrwxrwxrwx root   root         14 Thu, 2024-09-05 04:33:57 etc/rcS.d/S01kmod -> ../init.d/kmod
lrwxrwxrwx root   root         14 Thu, 2024-09-05 04:33:59 etc/rcS.d/S01udev -> ../init.d/udev
lrwxrwxrwx root   root         16 Thu, 2024-09-05 04:33:59 etc/rcS.d/S01procps -> ../init.d/procps
lrwxrwxrwx root   root         20 Thu, 2024-09-05 04:34:01 etc/rcS.d/S01networking -> ../init.d/networking
lrwxrwxrwx root   root         18 Thu, 2024-09-05 04:34:17 etc/rcS.d/S01apparmor -> ../init.d/apparmor
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:33:58 etc/systemd
drwxr-xr-x root   root          0 Tue, 2025-03-18 10:28:57 etc/systemd/system
drwxr-xr-x root   root          0 Tue, 2025-03-18 10:28:58 etc/systemd/system/timers.target.wants
lrwxrwxrwx root   root         40 Thu, 2024-09-05 04:33:29 etc/systemd/system/timers.target.wants/dpkg-db-backup.timer -> /lib/systemd/system/dpkg-db-backup.timer
lrwxrwxrwx root   root         43 Thu, 2024-09-05 04:33:38 etc/systemd/system/timers.target.wants/apt-daily-upgrade.timer -> /lib/systemd/system/apt-daily-upgrade.timer
lrwxrwxrwx root   root         35 Thu, 2024-09-05 04:33:38 etc/systemd/system/timers.target.wants/apt-daily.timer -> /lib/systemd/system/apt-daily.timer
lrwxrwxrwx root   root         37 Thu, 2024-09-05 04:33:37 etc/systemd/system/timers.target.wants/e2scrub_all.timer -> /lib/systemd/system/e2scrub_all.timer
lrwxrwxrwx root   root         32 Thu, 2024-09-05 04:33:37 etc/systemd/system/timers.target.wants/fstrim.timer -> /lib/systemd/system/fstrim.timer
lrwxrwxrwx root   root         35 Thu, 2024-09-05 04:33:59 etc/systemd/system/timers.target.wants/logrotate.timer -> /lib/systemd/system/logrotate.timer
lrwxrwxrwx root   root         32 Thu, 2024-09-05 04:34:01 etc/systemd/system/timers.target.wants/man-db.timer -> /lib/systemd/system/man-db.timer
lrwxrwxrwx root   root         44 Tue, 2025-03-18 10:28:58 etc/systemd/system/timers.target.wants/borg-backup-client.timer -> /etc/systemd/system/borg-backup-client.timer
drwxr-xr-x root   root          0 Tue, 2025-03-18 10:26:31 etc/systemd/system/multi-user.target.wants
lrwxrwxrwx root   root         40 Thu, 2024-09-05 04:33:37 etc/systemd/system/multi-user.target.wants/e2scrub_reap.service -> /lib/systemd/system/e2scrub_reap.service
lrwxrwxrwx root   root         36 Thu, 2024-09-05 04:33:48 etc/systemd/system/multi-user.target.wants/remote-fs.target -> /lib/systemd/system/remote-fs.target
lrwxrwxrwx root   root         32 Thu, 2024-09-05 04:33:59 etc/systemd/system/multi-user.target.wants/cron.service -> /lib/systemd/system/cron.service
lrwxrwxrwx root   root         38 Thu, 2024-09-05 04:34:01 etc/systemd/system/multi-user.target.wants/networking.service -> /lib/systemd/system/networking.service
lrwxrwxrwx root   root         34 Thu, 2024-09-05 04:34:17 etc/systemd/system/multi-user.target.wants/chrony.service -> /lib/systemd/system/chrony.service
lrwxrwxrwx root   root         31 Thu, 2024-09-05 04:34:18 etc/systemd/system/multi-user.target.wants/ssh.service -> /lib/systemd/system/ssh.service
lrwxrwxrwx root   root         35 Thu, 2024-09-05 04:34:18 etc/systemd/system/multi-user.target.wants/rsyslog.service -> /lib/systemd/system/rsyslog.service
lrwxrwxrwx root   root         47 Thu, 2024-09-05 04:34:19 etc/systemd/system/multi-user.target.wants/unattended-upgrades.service -> /lib/systemd/system/unattended-upgrades.service
lrwxrwxrwx root   root         45 Thu, 2024-09-05 04:33:58 etc/systemd/system/dbus-org.freedesktop.timesync1.service -> /lib/systemd/system/systemd-timesyncd.service
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:33:48 etc/systemd/system/getty.target.wants
lrwxrwxrwx root   root         34 Thu, 2024-09-05 04:33:48 etc/systemd/system/getty.target.wants/getty@tty1.service -> /lib/systemd/system/getty@.service
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:34:17 etc/systemd/system/sysinit.target.wants
lrwxrwxrwx root   root         42 Thu, 2024-09-05 04:33:48 etc/systemd/system/sysinit.target.wants/systemd-pstore.service -> /lib/systemd/system/systemd-pstore.service
lrwxrwxrwx root   root         45 Thu, 2024-09-05 04:33:58 etc/systemd/system/sysinit.target.wants/systemd-timesyncd.service -> /lib/systemd/system/systemd-timesyncd.service
lrwxrwxrwx root   root         36 Thu, 2024-09-05 04:34:17 etc/systemd/system/sysinit.target.wants/apparmor.service -> /lib/systemd/system/apparmor.service
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:34:01 etc/systemd/system/network-online.target.wants
lrwxrwxrwx root   root         38 Thu, 2024-09-05 04:34:01 etc/systemd/system/network-online.target.wants/networking.service -> /lib/systemd/system/networking.service
lrwxrwxrwx root   root         34 Thu, 2024-09-05 04:34:17 etc/systemd/system/chronyd.service -> /lib/systemd/system/chrony.service
lrwxrwxrwx root   root         31 Thu, 2024-09-05 04:34:18 etc/systemd/system/sshd.service -> /lib/systemd/system/ssh.service
lrwxrwxrwx root   root         35 Thu, 2024-09-05 04:34:18 etc/systemd/system/syslog.service -> /lib/systemd/system/rsyslog.service
-rw-r--r-- root   root        568 Thu, 2024-09-05 04:34:56 etc/systemd/system/set-grub-install-device.service
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:34:56 etc/systemd/system/ssh.service.d
-rw-r--r-- root   root        234 Thu, 2024-09-05 04:34:56 etc/systemd/system/ssh.service.d/generate-ssh-keys.conf
-rw-r--r-- root   root        662 Tue, 2025-03-18 10:28:56 etc/systemd/system/borg-backup-client.service
-rw-r--r-- root   root        110 Tue, 2025-03-18 10:28:57 etc/systemd/system/borg-backup-client.timer
drwxr-xr-x root   root          0 Tue, 2025-03-18 10:28:02 etc/systemd/user
drwxr-xr-x root   root          0 Tue, 2025-03-18 10:28:04 etc/systemd/user/sockets.target.wants
lrwxrwxrwx root   root         46 Tue, 2025-03-18 10:28:02 etc/systemd/user/sockets.target.wants/gpg-agent-browser.socket -> /usr/lib/systemd/user/gpg-agent-browser.socket
lrwxrwxrwx root   root         44 Tue, 2025-03-18 10:28:02 etc/systemd/user/sockets.target.wants/gpg-agent-extra.socket -> /usr/lib/systemd/user/gpg-agent-extra.socket
lrwxrwxrwx root   root         42 Tue, 2025-03-18 10:28:02 etc/systemd/user/sockets.target.wants/gpg-agent-ssh.socket -> /usr/lib/systemd/user/gpg-agent-ssh.socket
lrwxrwxrwx root   root         38 Tue, 2025-03-18 10:28:03 etc/systemd/user/sockets.target.wants/gpg-agent.socket -> /usr/lib/systemd/user/gpg-agent.socket
lrwxrwxrwx root   root         36 Tue, 2025-03-18 10:28:04 etc/systemd/user/sockets.target.wants/dirmngr.socket -> /usr/lib/systemd/user/dirmngr.socket
-rw-r--r-- root   root       1282 Sun, 2024-08-25 17:35:39 etc/systemd/journald.conf
-rw-r--r-- root   root       1539 Sun, 2024-08-25 17:35:39 etc/systemd/logind.conf
drwxr-xr-x root   root          0 Sun, 2024-08-25 17:35:39 etc/systemd/network
-rw-r--r-- root   root        846 Mon, 2024-08-19 20:25:31 etc/systemd/networkd.conf
-rw-r--r-- root   root        670 Mon, 2024-08-19 20:25:31 etc/systemd/pstore.conf
-rw-r--r-- root   root        953 Mon, 2024-08-19 20:25:31 etc/systemd/sleep.conf
-rw-r--r-- root   root       2080 Sun, 2024-08-25 17:35:39 etc/systemd/system.conf
-rw-r--r-- root   root       1415 Sun, 2024-08-25 17:35:39 etc/systemd/user.conf
-rw-r--r-- root   root        864 Sun, 2024-08-25 17:35:39 etc/systemd/timesyncd.conf
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:33:36 etc/selinux
-rw-r--r-- root   root       2065 Sat, 2022-06-04 17:10:35 etc/selinux/semanage.conf
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:33:36 etc/terminfo
-rw-r--r-- root   root        212 Sun, 2023-05-07 14:33:47 etc/terminfo/README
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:33:37 etc/security
-rw-r--r-- root   root       4564 Thu, 2023-09-21 20:55:12 etc/security/access.conf
-rw-r--r-- root   root       2234 Thu, 2023-09-21 20:55:12 etc/security/faillock.conf
-rw-r--r-- root   root       3635 Thu, 2023-09-21 20:55:12 etc/security/group.conf
-rw-r--r-- root   root       2752 Thu, 2023-09-21 20:55:12 etc/security/limits.conf
drwxr-xr-x root   root          0 Thu, 2023-09-21 20:55:12 etc/security/limits.d
-rw-r--r-- root   root       1637 Thu, 2023-09-21 20:55:12 etc/security/namespace.conf
drwxr-xr-x root   root          0 Thu, 2023-09-21 20:55:12 etc/security/namespace.d
-rwxr-xr-x root   root       1016 Thu, 2023-09-21 20:55:12 etc/security/namespace.init
-rw-r--r-- root   root       2971 Thu, 2023-09-21 20:55:12 etc/security/pam_env.conf
-rw-r--r-- root   root        418 Thu, 2023-09-21 20:55:12 etc/security/sepermit.conf
-rw-r--r-- root   root       2179 Thu, 2023-09-21 20:55:12 etc/security/time.conf
-rw------- root   root          0 Thu, 2024-09-05 04:33:37 etc/security/opasswd
-rw-r--r-- root   root        552 Thu, 2023-09-21 20:55:12 etc/pam.conf
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:34:17 etc/pam.d
-rw-r--r-- root   root        520 Thu, 2023-09-21 20:55:12 etc/pam.d/other
-rw-r--r-- root   root       4126 Thu, 2023-03-23 12:40:50 etc/pam.d/login
-rw-r--r-- root   root        384 Fri, 2022-11-11 08:28:15 etc/pam.d/chfn
-rw-r--r-- root   root         92 Fri, 2022-11-11 08:28:15 etc/pam.d/chpasswd
-rw-r--r-- root   root        581 Fri, 2022-11-11 08:28:15 etc/pam.d/chsh
-rw-r--r-- root   root         92 Fri, 2022-11-11 08:28:15 etc/pam.d/newusers
-rw-r--r-- root   root         92 Fri, 2022-11-11 08:28:15 etc/pam.d/passwd
-rw-r--r-- root   root        143 Thu, 2024-03-28 09:52:12 etc/pam.d/runuser
-rw-r--r-- root   root        138 Thu, 2024-03-28 09:52:12 etc/pam.d/runuser-l
-rw-r--r-- root   root       2259 Thu, 2024-03-28 09:52:12 etc/pam.d/su
-rw-r--r-- root   root        137 Thu, 2024-03-28 09:52:12 etc/pam.d/su-l
-rw-r--r-- root   root       1208 Thu, 2024-09-05 04:33:59 etc/pam.d/common-account
-rw-r--r-- root   root       1620 Thu, 2024-09-05 04:33:59 etc/pam.d/common-password
-rw-r--r-- root   root       1146 Thu, 2024-09-05 04:33:59 etc/pam.d/common-session
-rw-r--r-- root   root       1154 Thu, 2024-09-05 04:33:59 etc/pam.d/common-session-noninteractive
-rw-r--r-- root   root        606 Sun, 2022-07-17 08:49:40 etc/pam.d/cron
-rw-r--r-- root   root       1214 Thu, 2024-09-05 04:33:59 etc/pam.d/common-auth
-rw-r--r-- root   root       2133 Sat, 2024-06-22 19:38:08 etc/pam.d/sshd
-rw-r--r-- root   root        185 Tue, 2023-06-27 11:45:00 etc/pam.d/sudo
-rw-r--r-- root   root        170 Tue, 2023-06-27 11:45:00 etc/pam.d/sudo-i
-rw-r--r-- root   root      12569 Fri, 2022-11-11 08:28:15 etc/login.defs
-rw-r--r-- root   root        248 Thu, 2024-09-05 04:33:57 etc/modules
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:34:18 etc/init.d
-rwxr-xr-x root   root       1748 Thu, 2024-03-28 09:52:12 etc/init.d/hwclock.sh
-rwxr-xr-x root   root       3059 Sun, 2022-07-17 08:49:40 etc/init.d/cron
-rwxr-xr-x root   root       3152 Sat, 2023-09-16 10:03:58 etc/init.d/dbus
-rwxr-xr-x root   root       4531 Mon, 2023-01-23 21:14:41 etc/init.d/networking
-rwxr-xr-x root   root       2063 Fri, 2022-12-09 22:58:43 etc/init.d/kmod
-rwxr-xr-x root   root        959 Mon, 2022-12-19 06:06:38 etc/init.d/procps
-rwxr-xr-x root   root       6871 Sun, 2024-08-25 17:30:20 etc/init.d/udev
-rwxr-xr-x root   root       1897 Mon, 2023-05-08 20:05:00 etc/init.d/chrony
-rwxr-xr-x root   root       4060 Tue, 2023-12-19 12:55:09 etc/init.d/ssh
-rwxr-xr-x root   root       1161 Tue, 2023-06-27 11:45:00 etc/init.d/sudo
-rwxr-xr-x root   root       3740 Tue, 2023-02-14 11:49:15 etc/init.d/apparmor
-rwxr-xr-x root   root       1391 Sat, 2022-12-31 20:59:00 etc/init.d/unattended-upgrades
-rw-r--r-- root   root         33 Tue, 2025-03-18 10:26:30 etc/machine-id
-rw-r--r-- root   root          7 Tue, 2025-03-18 10:26:46 etc/hostname
-rw-r--r-- root   root         37 Thu, 2024-09-05 04:33:28 etc/fstab.old
-rw-r--r-- root   root        769 Sat, 2021-04-10 20:00:00 etc/profile
-rw-r--r-- root   root        286 Wed, 2024-08-14 16:10:00 etc/motd
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:33:28 etc/opt
-rw-r--r-- root   root          8 Thu, 2024-09-05 04:33:36 etc/timezone
-rw-r--r-- root   root       5989 Thu, 2024-09-05 04:33:39 etc/ca-certificates.conf
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:33:38 etc/ca-certificates
drwxr-xr-x root   root          0 Sat, 2023-03-11 08:47:05 etc/ca-certificates/update.d
lrwxrwxrwx root   root         27 Thu, 2024-09-05 04:33:36 etc/localtime -> /usr/share/zoneinfo/Etc/UTC
-rw-r--r-- root   root        526 Thu, 2024-09-05 04:33:57 etc/nsswitch.conf
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:33:39 etc/ssl
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:33:40 etc/ssl/certs
-rw-r--r-- root   root     213777 Thu, 2024-09-05 04:33:40 etc/ssl/certs/ca-certificates.crt
lrwxrwxrwx root   root         48 Thu, 2024-09-05 04:33:39 etc/ssl/certs/ACCVRAIZ1.pem -> /usr/share/ca-certificates/mozilla/ACCVRAIZ1.crt
lrwxrwxrwx root   root         55 Thu, 2024-09-05 04:33:39 etc/ssl/certs/AC_RAIZ_FNMT-RCM.pem -> /usr/share/ca-certificates/mozilla/AC_RAIZ_FNMT-RCM.crt
lrwxrwxrwx root   root         74 Thu, 2024-09-05 04:33:39 etc/ssl/certs/AC_RAIZ_FNMT-RCM_SERVIDORES_SEGUROS.pem -> /usr/share/ca-certificates/mozilla/AC_RAIZ_FNMT-RCM_SERVIDORES_SEGUROS.crt
lrwxrwxrwx root   root         64 Thu, 2024-09-05 04:33:39 etc/ssl/certs/ANF_Secure_Server_Root_CA.pem -> /usr/share/ca-certificates/mozilla/ANF_Secure_Server_Root_CA.crt
lrwxrwxrwx root   root         69 Thu, 2024-09-05 04:33:39 etc/ssl/certs/Actalis_Authentication_Root_CA.pem -> /usr/share/ca-certificates/mozilla/Actalis_Authentication_Root_CA.crt
lrwxrwxrwx root   root         61 Thu, 2024-09-05 04:33:39 etc/ssl/certs/AffirmTrust_Commercial.pem -> /usr/share/ca-certificates/mozilla/AffirmTrust_Commercial.crt
lrwxrwxrwx root   root         61 Thu, 2024-09-05 04:33:39 etc/ssl/certs/AffirmTrust_Networking.pem -> /usr/share/ca-certificates/mozilla/AffirmTrust_Networking.crt
lrwxrwxrwx root   root         58 Thu, 2024-09-05 04:33:39 etc/ssl/certs/AffirmTrust_Premium.pem -> /usr/share/ca-certificates/mozilla/AffirmTrust_Premium.crt
lrwxrwxrwx root   root         62 Thu, 2024-09-05 04:33:39 etc/ssl/certs/AffirmTrust_Premium_ECC.pem -> /usr/share/ca-certificates/mozilla/AffirmTrust_Premium_ECC.crt
lrwxrwxrwx root   root         55 Thu, 2024-09-05 04:33:39 etc/ssl/certs/Amazon_Root_CA_1.pem -> /usr/share/ca-certificates/mozilla/Amazon_Root_CA_1.crt
lrwxrwxrwx root   root         55 Thu, 2024-09-05 04:33:39 etc/ssl/certs/Amazon_Root_CA_2.pem -> /usr/share/ca-certificates/mozilla/Amazon_Root_CA_2.crt
lrwxrwxrwx root   root         55 Thu, 2024-09-05 04:33:39 etc/ssl/certs/Amazon_Root_CA_3.pem -> /usr/share/ca-certificates/mozilla/Amazon_Root_CA_3.crt
lrwxrwxrwx root   root         55 Thu, 2024-09-05 04:33:39 etc/ssl/certs/Amazon_Root_CA_4.pem -> /usr/share/ca-certificates/mozilla/Amazon_Root_CA_4.crt
lrwxrwxrwx root   root         60 Thu, 2024-09-05 04:33:39 etc/ssl/certs/Atos_TrustedRoot_2011.pem -> /usr/share/ca-certificates/mozilla/Atos_TrustedRoot_2011.crt
lrwxrwxrwx root   root         96 Thu, 2024-09-05 04:33:39 etc/ssl/certs/Autoridad_de_Certificacion_Firmaprofesional_CIF_A62634068.pem -> /usr/share/ca-certificates/mozilla/Autoridad_de_Certificacion_Firmaprofesional_CIF_A62634068.crt
lrwxrwxrwx root   root         98 Thu, 2024-09-05 04:33:39 etc/ssl/certs/Autoridad_de_Certificacion_Firmaprofesional_CIF_A62634068_2.pem -> /usr/share/ca-certificates/mozilla/Autoridad_de_Certificacion_Firmaprofesional_CIF_A62634068_2.crt
lrwxrwxrwx root   root         64 Thu, 2024-09-05 04:33:39 etc/ssl/certs/Baltimore_CyberTrust_Root.pem -> /usr/share/ca-certificates/mozilla/Baltimore_CyberTrust_Root.crt
lrwxrwxrwx root   root         62 Thu, 2024-09-05 04:33:39 etc/ssl/certs/Buypass_Class_2_Root_CA.pem -> /usr/share/ca-certificates/mozilla/Buypass_Class_2_Root_CA.crt
lrwxrwxrwx root   root         62 Thu, 2024-09-05 04:33:39 etc/ssl/certs/Buypass_Class_3_Root_CA.pem -> /usr/share/ca-certificates/mozilla/Buypass_Class_3_Root_CA.crt
lrwxrwxrwx root   root         55 Thu, 2024-09-05 04:33:39 etc/ssl/certs/CA_Disig_Root_R2.pem -> /usr/share/ca-certificates/mozilla/CA_Disig_Root_R2.crt
lrwxrwxrwx root   root         51 Thu, 2024-09-05 04:33:39 etc/ssl/certs/CFCA_EV_ROOT.pem -> /usr/share/ca-certificates/mozilla/CFCA_EV_ROOT.crt
lrwxrwxrwx root   root         69 Thu, 2024-09-05 04:33:39 etc/ssl/certs/COMODO_Certification_Authority.pem -> /usr/share/ca-certificates/mozilla/COMODO_Certification_Authority.crt
lrwxrwxrwx root   root         73 Thu, 2024-09-05 04:33:39 etc/ssl/certs/COMODO_ECC_Certification_Authority.pem -> /usr/share/ca-certificates/mozilla/COMODO_ECC_Certification_Authority.crt
lrwxrwxrwx root   root         73 Thu, 2024-09-05 04:33:39 etc/ssl/certs/COMODO_RSA_Certification_Authority.pem -> /usr/share/ca-certificates/mozilla/COMODO_RSA_Certification_Authority.crt
lrwxrwxrwx root   root         56 Thu, 2024-09-05 04:33:39 etc/ssl/certs/Certainly_Root_E1.pem -> /usr/share/ca-certificates/mozilla/Certainly_Root_E1.crt
lrwxrwxrwx root   root         56 Thu, 2024-09-05 04:33:39 etc/ssl/certs/Certainly_Root_R1.pem -> /usr/share/ca-certificates/mozilla/Certainly_Root_R1.crt
lrwxrwxrwx root   root         47 Thu, 2024-09-05 04:33:39 etc/ssl/certs/Certigna.pem -> /usr/share/ca-certificates/mozilla/Certigna.crt
lrwxrwxrwx root   root         55 Thu, 2024-09-05 04:33:39 etc/ssl/certs/Certigna_Root_CA.pem -> /usr/share/ca-certificates/mozilla/Certigna_Root_CA.crt
lrwxrwxrwx root   root         55 Thu, 2024-09-05 04:33:39 etc/ssl/certs/Certum_EC-384_CA.pem -> /usr/share/ca-certificates/mozilla/Certum_EC-384_CA.crt
lrwxrwxrwx root   root         64 Thu, 2024-09-05 04:33:39 etc/ssl/certs/Certum_Trusted_Network_CA.pem -> /usr/share/ca-certificates/mozilla/Certum_Trusted_Network_CA.crt
lrwxrwxrwx root   root         66 Thu, 2024-09-05 04:33:39 etc/ssl/certs/Certum_Trusted_Network_CA_2.pem -> /usr/share/ca-certificates/mozilla/Certum_Trusted_Network_CA_2.crt
lrwxrwxrwx root   root         61 Thu, 2024-09-05 04:33:39 etc/ssl/certs/Certum_Trusted_Root_CA.pem -> /usr/share/ca-certificates/mozilla/Certum_Trusted_Root_CA.crt
lrwxrwxrwx root   root         63 Thu, 2024-09-05 04:33:39 etc/ssl/certs/Comodo_AAA_Services_root.pem -> /usr/share/ca-certificates/mozilla/Comodo_AAA_Services_root.crt
lrwxrwxrwx root   root         64 Thu, 2024-09-05 04:33:39 etc/ssl/certs/D-TRUST_BR_Root_CA_1_2020.pem -> /usr/share/ca-certificates/mozilla/D-TRUST_BR_Root_CA_1_2020.crt
lrwxrwxrwx root   root         64 Thu, 2024-09-05 04:33:39 etc/ssl/certs/D-TRUST_EV_Root_CA_1_2020.pem -> /usr/share/ca-certificates/mozilla/D-TRUST_EV_Root_CA_1_2020.crt
lrwxrwxrwx root   root         69 Thu, 2024-09-05 04:33:39 etc/ssl/certs/D-TRUST_Root_Class_3_CA_2_2009.pem -> /usr/share/ca-certificates/mozilla/D-TRUST_Root_Class_3_CA_2_2009.crt
lrwxrwxrwx root   root         72 Thu, 2024-09-05 04:33:39 etc/ssl/certs/D-TRUST_Root_Class_3_CA_2_EV_2009.pem -> /usr/share/ca-certificates/mozilla/D-TRUST_Root_Class_3_CA_2_EV_2009.crt
lrwxrwxrwx root   root         66 Thu, 2024-09-05 04:33:39 etc/ssl/certs/DigiCert_Assured_ID_Root_CA.pem -> /usr/share/ca-certificates/mozilla/DigiCert_Assured_ID_Root_CA.crt
lrwxrwxrwx root   root         66 Thu, 2024-09-05 04:33:39 etc/ssl/certs/DigiCert_Assured_ID_Root_G2.pem -> /usr/share/ca-certificates/mozilla/DigiCert_Assured_ID_Root_G2.crt
lrwxrwxrwx root   root         66 Thu, 2024-09-05 04:33:39 etc/ssl/certs/DigiCert_Assured_ID_Root_G3.pem -> /usr/share/ca-certificates/mozilla/DigiCert_Assured_ID_Root_G3.crt
lrwxrwxrwx root   root         62 Thu, 2024-09-05 04:33:39 etc/ssl/certs/DigiCert_Global_Root_CA.pem -> /usr/share/ca-certificates/mozilla/DigiCert_Global_Root_CA.crt
lrwxrwxrwx root   root         62 Thu, 2024-09-05 04:33:39 etc/ssl/certs/DigiCert_Global_Root_G2.pem -> /usr/share/ca-certificates/mozilla/DigiCert_Global_Root_G2.crt
lrwxrwxrwx root   root         62 Thu, 2024-09-05 04:33:39 etc/ssl/certs/DigiCert_Global_Root_G3.pem -> /usr/share/ca-certificates/mozilla/DigiCert_Global_Root_G3.crt
lrwxrwxrwx root   root         73 Thu, 2024-09-05 04:33:39 etc/ssl/certs/DigiCert_High_Assurance_EV_Root_CA.pem -> /usr/share/ca-certificates/mozilla/DigiCert_High_Assurance_EV_Root_CA.crt
lrwxrwxrwx root   root         68 Thu, 2024-09-05 04:33:39 etc/ssl/certs/DigiCert_TLS_ECC_P384_Root_G5.pem -> /usr/share/ca-certificates/mozilla/DigiCert_TLS_ECC_P384_Root_G5.crt
lrwxrwxrwx root   root         67 Thu, 2024-09-05 04:33:39 etc/ssl/certs/DigiCert_TLS_RSA4096_Root_G5.pem -> /usr/share/ca-certificates/mozilla/DigiCert_TLS_RSA4096_Root_G5.crt
lrwxrwxrwx root   root         63 Thu, 2024-09-05 04:33:39 etc/ssl/certs/DigiCert_Trusted_Root_G4.pem -> /usr/share/ca-certificates/mozilla/DigiCert_Trusted_Root_G4.crt
lrwxrwxrwx root   root         70 Thu, 2024-09-05 04:33:39 etc/ssl/certs/E-Tugra_Certification_Authority.pem -> /usr/share/ca-certificates/mozilla/E-Tugra_Certification_Authority.crt
lrwxrwxrwx root   root         68 Thu, 2024-09-05 04:33:39 etc/ssl/certs/E-Tugra_Global_Root_CA_ECC_v3.pem -> /usr/share/ca-certificates/mozilla/E-Tugra_Global_Root_CA_ECC_v3.crt
lrwxrwxrwx root   root         68 Thu, 2024-09-05 04:33:39 etc/ssl/certs/E-Tugra_Global_Root_CA_RSA_v3.pem -> /usr/share/ca-certificates/mozilla/E-Tugra_Global_Root_CA_RSA_v3.crt
lrwxrwxrwx root   root         80 Thu, 2024-09-05 04:33:39 etc/ssl/certs/Entrust.net_Premium_2048_Secure_Server_CA.pem -> /usr/share/ca-certificates/mozilla/Entrust.net_Premium_2048_Secure_Server_CA.crt
lrwxrwxrwx root   root         75 Thu, 2024-09-05 04:33:39 etc/ssl/certs/Entrust_Root_Certification_Authority.pem -> /usr/share/ca-certificates/mozilla/Entrust_Root_Certification_Authority.crt
lrwxrwxrwx root   root         81 Thu, 2024-09-05 04:33:39 etc/ssl/certs/Entrust_Root_Certification_Authority_-_EC1.pem -> /usr/share/ca-certificates/mozilla/Entrust_Root_Certification_Authority_-_EC1.crt
lrwxrwxrwx root   root         80 Thu, 2024-09-05 04:33:39 etc/ssl/certs/Entrust_Root_Certification_Authority_-_G2.pem -> /usr/share/ca-certificates/mozilla/Entrust_Root_Certification_Authority_-_G2.crt
lrwxrwxrwx root   root         80 Thu, 2024-09-05 04:33:39 etc/ssl/certs/Entrust_Root_Certification_Authority_-_G4.pem -> /usr/share/ca-certificates/mozilla/Entrust_Root_Certification_Authority_-_G4.crt
lrwxrwxrwx root   root         61 Thu, 2024-09-05 04:33:39 etc/ssl/certs/GDCA_TrustAUTH_R5_ROOT.pem -> /usr/share/ca-certificates/mozilla/GDCA_TrustAUTH_R5_ROOT.crt
lrwxrwxrwx root   root         55 Thu, 2024-09-05 04:33:39 etc/ssl/certs/GLOBALTRUST_2020.pem -> /usr/share/ca-certificates/mozilla/GLOBALTRUST_2020.crt
lrwxrwxrwx root   root         50 Thu, 2024-09-05 04:33:39 etc/ssl/certs/GTS_Root_R1.pem -> /usr/share/ca-certificates/mozilla/GTS_Root_R1.crt
lrwxrwxrwx root   root         50 Thu, 2024-09-05 04:33:39 etc/ssl/certs/GTS_Root_R2.pem -> /usr/share/ca-certificates/mozilla/GTS_Root_R2.crt
lrwxrwxrwx root   root         50 Thu, 2024-09-05 04:33:39 etc/ssl/certs/GTS_Root_R3.pem -> /usr/share/ca-certificates/mozilla/GTS_Root_R3.crt
lrwxrwxrwx root   root         50 Thu, 2024-09-05 04:33:39 etc/ssl/certs/GTS_Root_R4.pem -> /usr/share/ca-certificates/mozilla/GTS_Root_R4.crt
lrwxrwxrwx root   root         66 Thu, 2024-09-05 04:33:39 etc/ssl/certs/GlobalSign_ECC_Root_CA_-_R4.pem -> /usr/share/ca-certificates/mozilla/GlobalSign_ECC_Root_CA_-_R4.crt
lrwxrwxrwx root   root         66 Thu, 2024-09-05 04:33:39 etc/ssl/certs/GlobalSign_ECC_Root_CA_-_R5.pem -> /usr/share/ca-certificates/mozilla/GlobalSign_ECC_Root_CA_-_R5.crt
lrwxrwxrwx root   root         57 Thu, 2024-09-05 04:33:39 etc/ssl/certs/GlobalSign_Root_CA.pem -> /usr/share/ca-certificates/mozilla/GlobalSign_Root_CA.crt
lrwxrwxrwx root   root         62 Thu, 2024-09-05 04:33:39 etc/ssl/certs/GlobalSign_Root_CA_-_R3.pem -> /usr/share/ca-certificates/mozilla/GlobalSign_Root_CA_-_R3.crt
lrwxrwxrwx root   root         62 Thu, 2024-09-05 04:33:39 etc/ssl/certs/GlobalSign_Root_CA_-_R6.pem -> /usr/share/ca-certificates/mozilla/GlobalSign_Root_CA_-_R6.crt
lrwxrwxrwx root   root         58 Thu, 2024-09-05 04:33:39 etc/ssl/certs/GlobalSign_Root_E46.pem -> /usr/share/ca-certificates/mozilla/GlobalSign_Root_E46.crt
lrwxrwxrwx root   root         58 Thu, 2024-09-05 04:33:39 etc/ssl/certs/GlobalSign_Root_R46.pem -> /usr/share/ca-certificates/mozilla/GlobalSign_Root_R46.crt
lrwxrwxrwx root   root         58 Thu, 2024-09-05 04:33:39 etc/ssl/certs/Go_Daddy_Class_2_CA.pem -> /usr/share/ca-certificates/mozilla/Go_Daddy_Class_2_CA.crt
lrwxrwxrwx root   root         79 Thu, 2024-09-05 04:33:39 etc/ssl/certs/Go_Daddy_Root_Certificate_Authority_-_G2.pem -> /usr/share/ca-certificates/mozilla/Go_Daddy_Root_Certificate_Authority_-_G2.crt
lrwxrwxrwx root   root         66 Thu, 2024-09-05 04:33:39 etc/ssl/certs/HARICA_TLS_ECC_Root_CA_2021.pem -> /usr/share/ca-certificates/mozilla/HARICA_TLS_ECC_Root_CA_2021.crt
lrwxrwxrwx root   root         66 Thu, 2024-09-05 04:33:39 etc/ssl/certs/HARICA_TLS_RSA_Root_CA_2021.pem -> /usr/share/ca-certificates/mozilla/HARICA_TLS_RSA_Root_CA_2021.crt
lrwxrwxrwx root   root         98 Thu, 2024-09-05 04:33:39 etc/ssl/certs/Hellenic_Academic_and_Research_Institutions_ECC_RootCA_2015.pem -> /usr/share/ca-certificates/mozilla/Hellenic_Academic_and_Research_Institutions_ECC_RootCA_2015.crt
lrwxrwxrwx root   root         94 Thu, 2024-09-05 04:33:39 etc/ssl/certs/Hellenic_Academic_and_Research_Institutions_RootCA_2015.pem -> /usr/share/ca-certificates/mozilla/Hellenic_Academic_and_Research_Institutions_RootCA_2015.crt
lrwxrwxrwx root   root         57 Thu, 2024-09-05 04:33:39 etc/ssl/certs/HiPKI_Root_CA_-_G1.pem -> /usr/share/ca-certificates/mozilla/HiPKI_Root_CA_-_G1.crt
lrwxrwxrwx root   root         62 Thu, 2024-09-05 04:33:39 etc/ssl/certs/Hongkong_Post_Root_CA_1.pem -> /usr/share/ca-certificates/mozilla/Hongkong_Post_Root_CA_1.crt
lrwxrwxrwx root   root         62 Thu, 2024-09-05 04:33:39 etc/ssl/certs/Hongkong_Post_Root_CA_3.pem -> /usr/share/ca-certificates/mozilla/Hongkong_Post_Root_CA_3.crt
lrwxrwxrwx root   root         51 Thu, 2024-09-05 04:33:39 etc/ssl/certs/ISRG_Root_X1.pem -> /usr/share/ca-certificates/mozilla/ISRG_Root_X1.crt
lrwxrwxrwx root   root         51 Thu, 2024-09-05 04:33:39 etc/ssl/certs/ISRG_Root_X2.pem -> /usr/share/ca-certificates/mozilla/ISRG_Root_X2.crt
lrwxrwxrwx root   root         69 Thu, 2024-09-05 04:33:39 etc/ssl/certs/IdenTrust_Commercial_Root_CA_1.pem -> /usr/share/ca-certificates/mozilla/IdenTrust_Commercial_Root_CA_1.crt
lrwxrwxrwx root   root         72 Thu, 2024-09-05 04:33:39 etc/ssl/certs/IdenTrust_Public_Sector_Root_CA_1.pem -> /usr/share/ca-certificates/mozilla/IdenTrust_Public_Sector_Root_CA_1.crt
lrwxrwxrwx root   root         49 Thu, 2024-09-05 04:33:39 etc/ssl/certs/Izenpe.com.pem -> /usr/share/ca-certificates/mozilla/Izenpe.com.crt
lrwxrwxrwx root   root         69 Thu, 2024-09-05 04:33:39 etc/ssl/certs/Microsec_e-Szigno_Root_CA_2009.pem -> /usr/share/ca-certificates/mozilla/Microsec_e-Szigno_Root_CA_2009.crt
lrwxrwxrwx root   root         84 Thu, 2024-09-05 04:33:39 etc/ssl/certs/Microsoft_ECC_Root_Certificate_Authority_2017.pem -> /usr/share/ca-certificates/mozilla/Microsoft_ECC_Root_Certificate_Authority_2017.crt
lrwxrwxrwx root   root         84 Thu, 2024-09-05 04:33:39 etc/ssl/certs/Microsoft_RSA_Root_Certificate_Authority_2017.pem -> /usr/share/ca-certificates/mozilla/Microsoft_RSA_Root_Certificate_Authority_2017.crt
lrwxrwxrwx root   root         80 Thu, 2024-09-05 04:33:39 etc/ssl/certs/NAVER_Global_Root_Certification_Authority.pem -> /usr/share/ca-certificates/mozilla/NAVER_Global_Root_Certification_Authority.crt
lrwxrwxrwx root   root         79 Thu, 2024-09-05 04:33:39 etc/ssl/certs/NetLock_Arany_=Class_Gold=_Főtanúsítvány.pem -> /usr/share/ca-certificates/mozilla/NetLock_Arany_=Class_Gold=_Főtanúsítvány.crt
lrwxrwxrwx root   root         70 Thu, 2024-09-05 04:33:39 etc/ssl/certs/OISTE_WISeKey_Global_Root_GB_CA.pem -> /usr/share/ca-certificates/mozilla/OISTE_WISeKey_Global_Root_GB_CA.crt
lrwxrwxrwx root   root         70 Thu, 2024-09-05 04:33:39 etc/ssl/certs/OISTE_WISeKey_Global_Root_GC_CA.pem -> /usr/share/ca-certificates/mozilla/OISTE_WISeKey_Global_Root_GC_CA.crt
lrwxrwxrwx root   root         60 Thu, 2024-09-05 04:33:39 etc/ssl/certs/QuoVadis_Root_CA_1_G3.pem -> /usr/share/ca-certificates/mozilla/QuoVadis_Root_CA_1_G3.crt
lrwxrwxrwx root   root         57 Thu, 2024-09-05 04:33:39 etc/ssl/certs/QuoVadis_Root_CA_2.pem -> /usr/share/ca-certificates/mozilla/QuoVadis_Root_CA_2.crt
lrwxrwxrwx root   root         60 Thu, 2024-09-05 04:33:40 etc/ssl/certs/QuoVadis_Root_CA_2_G3.pem -> /usr/share/ca-certificates/mozilla/QuoVadis_Root_CA_2_G3.crt
lrwxrwxrwx root   root         57 Thu, 2024-09-05 04:33:40 etc/ssl/certs/QuoVadis_Root_CA_3.pem -> /usr/share/ca-certificates/mozilla/QuoVadis_Root_CA_3.crt
lrwxrwxrwx root   root         60 Thu, 2024-09-05 04:33:40 etc/ssl/certs/QuoVadis_Root_CA_3_G3.pem -> /usr/share/ca-certificates/mozilla/QuoVadis_Root_CA_3_G3.crt
lrwxrwxrwx root   root         82 Thu, 2024-09-05 04:33:40 etc/ssl/certs/SSL.com_EV_Root_Certification_Authority_ECC.pem -> /usr/share/ca-certificates/mozilla/SSL.com_EV_Root_Certification_Authority_ECC.crt
lrwxrwxrwx root   root         85 Thu, 2024-09-05 04:33:40 etc/ssl/certs/SSL.com_EV_Root_Certification_Authority_RSA_R2.pem -> /usr/share/ca-certificates/mozilla/SSL.com_EV_Root_Certification_Authority_RSA_R2.crt
lrwxrwxrwx root   root         79 Thu, 2024-09-05 04:33:40 etc/ssl/certs/SSL.com_Root_Certification_Authority_ECC.pem -> /usr/share/ca-certificates/mozilla/SSL.com_Root_Certification_Authority_ECC.crt
lrwxrwxrwx root   root         79 Thu, 2024-09-05 04:33:40 etc/ssl/certs/SSL.com_Root_Certification_Authority_RSA.pem -> /usr/share/ca-certificates/mozilla/SSL.com_Root_Certification_Authority_RSA.crt
lrwxrwxrwx root   root         54 Thu, 2024-09-05 04:33:40 etc/ssl/certs/SZAFIR_ROOT_CA2.pem -> /usr/share/ca-certificates/mozilla/SZAFIR_ROOT_CA2.crt
lrwxrwxrwx root   root         58 Thu, 2024-09-05 04:33:40 etc/ssl/certs/SecureSign_RootCA11.pem -> /usr/share/ca-certificates/mozilla/SecureSign_RootCA11.crt
lrwxrwxrwx root   root         53 Thu, 2024-09-05 04:33:40 etc/ssl/certs/SecureTrust_CA.pem -> /usr/share/ca-certificates/mozilla/SecureTrust_CA.crt
lrwxrwxrwx root   root         55 Thu, 2024-09-05 04:33:40 etc/ssl/certs/Secure_Global_CA.pem -> /usr/share/ca-certificates/mozilla/Secure_Global_CA.crt
lrwxrwxrwx root   root         73 Thu, 2024-09-05 04:33:40 etc/ssl/certs/Security_Communication_ECC_RootCA1.pem -> /usr/share/ca-certificates/mozilla/Security_Communication_ECC_RootCA1.crt
lrwxrwxrwx root   root         69 Thu, 2024-09-05 04:33:40 etc/ssl/certs/Security_Communication_RootCA2.pem -> /usr/share/ca-certificates/mozilla/Security_Communication_RootCA2.crt
lrwxrwxrwx root   root         69 Thu, 2024-09-05 04:33:40 etc/ssl/certs/Security_Communication_RootCA3.pem -> /usr/share/ca-certificates/mozilla/Security_Communication_RootCA3.crt
lrwxrwxrwx root   root         69 Thu, 2024-09-05 04:33:40 etc/ssl/certs/Security_Communication_Root_CA.pem -> /usr/share/ca-certificates/mozilla/Security_Communication_Root_CA.crt
lrwxrwxrwx root   root         59 Thu, 2024-09-05 04:33:40 etc/ssl/certs/Starfield_Class_2_CA.pem -> /usr/share/ca-certificates/mozilla/Starfield_Class_2_CA.crt
lrwxrwxrwx root   root         80 Thu, 2024-09-05 04:33:40 etc/ssl/certs/Starfield_Root_Certificate_Authority_-_G2.pem -> /usr/share/ca-certificates/mozilla/Starfield_Root_Certificate_Authority_-_G2.crt
lrwxrwxrwx root   root         89 Thu, 2024-09-05 04:33:40 etc/ssl/certs/Starfield_Services_Root_Certificate_Authority_-_G2.pem -> /usr/share/ca-certificates/mozilla/Starfield_Services_Root_Certificate_Authority_-_G2.crt
lrwxrwxrwx root   root         61 Thu, 2024-09-05 04:33:40 etc/ssl/certs/SwissSign_Gold_CA_-_G2.pem -> /usr/share/ca-certificates/mozilla/SwissSign_Gold_CA_-_G2.crt
lrwxrwxrwx root   root         63 Thu, 2024-09-05 04:33:40 etc/ssl/certs/SwissSign_Silver_CA_-_G2.pem -> /usr/share/ca-certificates/mozilla/SwissSign_Silver_CA_-_G2.crt
lrwxrwxrwx root   root         67 Thu, 2024-09-05 04:33:40 etc/ssl/certs/T-TeleSec_GlobalRoot_Class_2.pem -> /usr/share/ca-certificates/mozilla/T-TeleSec_GlobalRoot_Class_2.crt
lrwxrwxrwx root   root         67 Thu, 2024-09-05 04:33:40 etc/ssl/certs/T-TeleSec_GlobalRoot_Class_3.pem -> /usr/share/ca-certificates/mozilla/T-TeleSec_GlobalRoot_Class_3.crt
lrwxrwxrwx root   root         84 Thu, 2024-09-05 04:33:40 etc/ssl/certs/TUBITAK_Kamu_SM_SSL_Kok_Sertifikasi_-_Surum_1.pem -> /usr/share/ca-certificates/mozilla/TUBITAK_Kamu_SM_SSL_Kok_Sertifikasi_-_Surum_1.crt
lrwxrwxrwx root   root         58 Thu, 2024-09-05 04:33:40 etc/ssl/certs/TWCA_Global_Root_CA.pem -> /usr/share/ca-certificates/mozilla/TWCA_Global_Root_CA.crt
lrwxrwxrwx root   root         72 Thu, 2024-09-05 04:33:40 etc/ssl/certs/TWCA_Root_Certification_Authority.pem -> /usr/share/ca-certificates/mozilla/TWCA_Root_Certification_Authority.crt
lrwxrwxrwx root   root         61 Thu, 2024-09-05 04:33:40 etc/ssl/certs/TeliaSonera_Root_CA_v1.pem -> /usr/share/ca-certificates/mozilla/TeliaSonera_Root_CA_v1.crt
lrwxrwxrwx root   root         55 Thu, 2024-09-05 04:33:40 etc/ssl/certs/Telia_Root_CA_v2.pem -> /usr/share/ca-certificates/mozilla/Telia_Root_CA_v2.crt
lrwxrwxrwx root   root         53 Thu, 2024-09-05 04:33:40 etc/ssl/certs/TrustCor_ECA-1.pem -> /usr/share/ca-certificates/mozilla/TrustCor_ECA-1.crt
lrwxrwxrwx root   root         61 Thu, 2024-09-05 04:33:40 etc/ssl/certs/TrustCor_RootCert_CA-1.pem -> /usr/share/ca-certificates/mozilla/TrustCor_RootCert_CA-1.crt
lrwxrwxrwx root   root         61 Thu, 2024-09-05 04:33:40 etc/ssl/certs/TrustCor_RootCert_CA-2.pem -> /usr/share/ca-certificates/mozilla/TrustCor_RootCert_CA-2.crt
lrwxrwxrwx root   root         79 Thu, 2024-09-05 04:33:40 etc/ssl/certs/Trustwave_Global_Certification_Authority.pem -> /usr/share/ca-certificates/mozilla/Trustwave_Global_Certification_Authority.crt
lrwxrwxrwx root   root         88 Thu, 2024-09-05 04:33:40 etc/ssl/certs/Trustwave_Global_ECC_P256_Certification_Authority.pem -> /usr/share/ca-certificates/mozilla/Trustwave_Global_ECC_P256_Certification_Authority.crt
lrwxrwxrwx root   root         88 Thu, 2024-09-05 04:33:40 etc/ssl/certs/Trustwave_Global_ECC_P384_Certification_Authority.pem -> /usr/share/ca-certificates/mozilla/Trustwave_Global_ECC_P384_Certification_Authority.crt
lrwxrwxrwx root   root         55 Thu, 2024-09-05 04:33:40 etc/ssl/certs/TunTrust_Root_CA.pem -> /usr/share/ca-certificates/mozilla/TunTrust_Root_CA.crt
lrwxrwxrwx root   root         67 Thu, 2024-09-05 04:33:40 etc/ssl/certs/UCA_Extended_Validation_Root.pem -> /usr/share/ca-certificates/mozilla/UCA_Extended_Validation_Root.crt
lrwxrwxrwx root   root         57 Thu, 2024-09-05 04:33:40 etc/ssl/certs/UCA_Global_G2_Root.pem -> /usr/share/ca-certificates/mozilla/UCA_Global_G2_Root.crt
lrwxrwxrwx root   root         76 Thu, 2024-09-05 04:33:40 etc/ssl/certs/USERTrust_ECC_Certification_Authority.pem -> /usr/share/ca-certificates/mozilla/USERTrust_ECC_Certification_Authority.crt
lrwxrwxrwx root   root         76 Thu, 2024-09-05 04:33:40 etc/ssl/certs/USERTrust_RSA_Certification_Authority.pem -> /usr/share/ca-certificates/mozilla/USERTrust_RSA_Certification_Authority.crt
lrwxrwxrwx root   root         59 Thu, 2024-09-05 04:33:40 etc/ssl/certs/XRamp_Global_CA_Root.pem -> /usr/share/ca-certificates/mozilla/XRamp_Global_CA_Root.crt
lrwxrwxrwx root   root         55 Thu, 2024-09-05 04:33:40 etc/ssl/certs/certSIGN_ROOT_CA.pem -> /usr/share/ca-certificates/mozilla/certSIGN_ROOT_CA.crt
lrwxrwxrwx root   root         58 Thu, 2024-09-05 04:33:40 etc/ssl/certs/certSIGN_Root_CA_G2.pem -> /usr/share/ca-certificates/mozilla/certSIGN_Root_CA_G2.crt
lrwxrwxrwx root   root         60 Thu, 2024-09-05 04:33:40 etc/ssl/certs/e-Szigno_Root_CA_2017.pem -> /usr/share/ca-certificates/mozilla/e-Szigno_Root_CA_2017.crt
lrwxrwxrwx root   root         72 Thu, 2024-09-05 04:33:40 etc/ssl/certs/ePKI_Root_Certification_Authority.pem -> /usr/share/ca-certificates/mozilla/ePKI_Root_Certification_Authority.crt
lrwxrwxrwx root   root         62 Thu, 2024-09-05 04:33:40 etc/ssl/certs/emSign_ECC_Root_CA_-_C3.pem -> /usr/share/ca-certificates/mozilla/emSign_ECC_Root_CA_-_C3.crt
lrwxrwxrwx root   root         62 Thu, 2024-09-05 04:33:40 etc/ssl/certs/emSign_ECC_Root_CA_-_G3.pem -> /usr/share/ca-certificates/mozilla/emSign_ECC_Root_CA_-_G3.crt
lrwxrwxrwx root   root         58 Thu, 2024-09-05 04:33:40 etc/ssl/certs/emSign_Root_CA_-_C1.pem -> /usr/share/ca-certificates/mozilla/emSign_Root_CA_-_C1.crt
lrwxrwxrwx root   root         58 Thu, 2024-09-05 04:33:40 etc/ssl/certs/emSign_Root_CA_-_G1.pem -> /usr/share/ca-certificates/mozilla/emSign_Root_CA_-_G1.crt
lrwxrwxrwx root   root         56 Thu, 2024-09-05 04:33:40 etc/ssl/certs/vTrus_ECC_Root_CA.pem -> /usr/share/ca-certificates/mozilla/vTrus_ECC_Root_CA.crt
lrwxrwxrwx root   root         52 Thu, 2024-09-05 04:33:40 etc/ssl/certs/vTrus_Root_CA.pem -> /usr/share/ca-certificates/mozilla/vTrus_Root_CA.crt
lrwxrwxrwx root   root         37 Thu, 2024-09-05 04:33:40 etc/ssl/certs/d4dae3dd.0 -> D-TRUST_Root_Class_3_CA_2_EV_2009.pem
lrwxrwxrwx root   root         45 Thu, 2024-09-05 04:33:40 etc/ssl/certs/5e98733a.0 -> Entrust_Root_Certification_Authority_-_G4.pem
lrwxrwxrwx root   root         29 Thu, 2024-09-05 04:33:40 etc/ssl/certs/b433981b.0 -> ANF_Secure_Server_Root_CA.pem
lrwxrwxrwx root   root         38 Thu, 2024-09-05 04:33:40 etc/ssl/certs/5860aaa6.0 -> Security_Communication_ECC_RootCA1.pem
lrwxrwxrwx root   root         20 Thu, 2024-09-05 04:33:40 etc/ssl/certs/ce5e74ef.0 -> Amazon_Root_CA_1.pem
lrwxrwxrwx root   root         61 Thu, 2024-09-05 04:33:40 etc/ssl/certs/3bde41ac.0 -> Autoridad_de_Certificacion_Firmaprofesional_CIF_A62634068.pem
lrwxrwxrwx root   root         63 Thu, 2024-09-05 04:33:40 etc/ssl/certs/3bde41ac.1 -> Autoridad_de_Certificacion_Firmaprofesional_CIF_A62634068_2.pem
lrwxrwxrwx root   root         31 Thu, 2024-09-05 04:33:40 etc/ssl/certs/40193066.0 -> Certum_Trusted_Network_CA_2.pem
lrwxrwxrwx root   root         63 Thu, 2024-09-05 04:33:40 etc/ssl/certs/7719f463.0 -> Hellenic_Academic_and_Research_Institutions_ECC_RootCA_2015.pem
lrwxrwxrwx root   root         34 Thu, 2024-09-05 04:33:40 etc/ssl/certs/40547a79.0 -> COMODO_Certification_Authority.pem
lrwxrwxrwx root   root         31 Thu, 2024-09-05 04:33:40 etc/ssl/certs/b1159c4c.0 -> DigiCert_Assured_ID_Root_CA.pem
lrwxrwxrwx root   root         20 Thu, 2024-09-05 04:33:40 etc/ssl/certs/cd8c0d63.0 -> AC_RAIZ_FNMT-RCM.pem
lrwxrwxrwx root   root         27 Thu, 2024-09-05 04:33:40 etc/ssl/certs/54657681.0 -> Buypass_Class_2_Root_CA.pem
lrwxrwxrwx root   root         20 Thu, 2024-09-05 04:33:40 etc/ssl/certs/de6d66f3.0 -> Amazon_Root_CA_4.pem
lrwxrwxrwx root   root         27 Thu, 2024-09-05 04:33:40 etc/ssl/certs/e8de2f56.0 -> Buypass_Class_3_Root_CA.pem
lrwxrwxrwx root   root         31 Thu, 2024-09-05 04:33:40 etc/ssl/certs/9f727ac7.0 -> HARICA_TLS_RSA_Root_CA_2021.pem
lrwxrwxrwx root   root         49 Thu, 2024-09-05 04:33:40 etc/ssl/certs/bf53fb88.0 -> Microsoft_RSA_Root_Certificate_Authority_2017.pem
lrwxrwxrwx root   root         31 Thu, 2024-09-05 04:33:40 etc/ssl/certs/b0e59380.0 -> GlobalSign_ECC_Root_CA_-_R4.pem
lrwxrwxrwx root   root         20 Thu, 2024-09-05 04:33:40 etc/ssl/certs/fa5da96b.0 -> GLOBALTRUST_2020.pem
lrwxrwxrwx root   root         17 Thu, 2024-09-05 04:33:40 etc/ssl/certs/7a3adc42.0 -> vTrus_Root_CA.pem
lrwxrwxrwx root   root         32 Thu, 2024-09-05 04:33:40 etc/ssl/certs/1e09d511.0 -> T-TeleSec_GlobalRoot_Class_2.pem
lrwxrwxrwx root   root         26 Thu, 2024-09-05 04:33:40 etc/ssl/certs/3e44d2f7.0 -> TrustCor_RootCert_CA-2.pem
lrwxrwxrwx root   root         59 Thu, 2024-09-05 04:33:40 etc/ssl/certs/32888f65.0 -> Hellenic_Academic_and_Research_Institutions_RootCA_2015.pem
lrwxrwxrwx root   root         27 Thu, 2024-09-05 04:33:40 etc/ssl/certs/062cdee6.0 -> GlobalSign_Root_CA_-_R3.pem
lrwxrwxrwx root   root         37 Thu, 2024-09-05 04:33:40 etc/ssl/certs/ca6e4ad9.0 -> ePKI_Root_Certification_Authority.pem
lrwxrwxrwx root   root         25 Thu, 2024-09-05 04:33:40 etc/ssl/certs/e18bfb83.0 -> QuoVadis_Root_CA_3_G3.pem
lrwxrwxrwx root   root         27 Thu, 2024-09-05 04:33:40 etc/ssl/certs/4b718d9b.0 -> emSign_ECC_Root_CA_-_C3.pem
lrwxrwxrwx root   root         16 Thu, 2024-09-05 04:33:40 etc/ssl/certs/4042bcee.0 -> ISRG_Root_X1.pem
lrwxrwxrwx root   root         54 Thu, 2024-09-05 04:33:40 etc/ssl/certs/09789157.0 -> Starfield_Services_Root_Certificate_Authority_-_G2.pem
lrwxrwxrwx root   root         26 Thu, 2024-09-05 04:33:40 etc/ssl/certs/5cd81ad7.0 -> TeliaSonera_Root_CA_v1.pem
lrwxrwxrwx root   root         19 Thu, 2024-09-05 04:33:40 etc/ssl/certs/fe8a2cd8.0 -> SZAFIR_ROOT_CA2.pem
lrwxrwxrwx root   root         20 Thu, 2024-09-05 04:33:40 etc/ssl/certs/6d41d539.0 -> Amazon_Root_CA_2.pem
lrwxrwxrwx root   root         15 Thu, 2024-09-05 04:33:40 etc/ssl/certs/1001acf7.0 -> GTS_Root_R1.pem
lrwxrwxrwx root   root         44 Thu, 2024-09-05 04:33:40 etc/ssl/certs/cbf06781.0 -> Go_Daddy_Root_Certificate_Authority_-_G2.pem
lrwxrwxrwx root   root         23 Thu, 2024-09-05 04:33:40 etc/ssl/certs/2923b3f9.0 -> emSign_Root_CA_-_G1.pem
lrwxrwxrwx root   root         29 Thu, 2024-09-05 04:33:40 etc/ssl/certs/9ef4a08a.0 -> D-TRUST_BR_Root_CA_1_2020.pem
lrwxrwxrwx root   root         23 Thu, 2024-09-05 04:33:40 etc/ssl/certs/feffd413.0 -> GlobalSign_Root_E46.pem
lrwxrwxrwx root   root         23 Thu, 2024-09-05 04:33:40 etc/ssl/certs/406c9bb1.0 -> emSign_Root_CA_-_C1.pem
lrwxrwxrwx root   root         38 Thu, 2024-09-05 04:33:40 etc/ssl/certs/eed8c118.0 -> COMODO_ECC_Certification_Authority.pem
lrwxrwxrwx root   root         15 Thu, 2024-09-05 04:33:40 etc/ssl/certs/0a775a30.0 -> GTS_Root_R3.pem
lrwxrwxrwx root   root         20 Thu, 2024-09-05 04:33:40 etc/ssl/certs/9482e63a.0 -> Certum_EC-384_CA.pem
lrwxrwxrwx root   root         18 Thu, 2024-09-05 04:33:40 etc/ssl/certs/f39fc864.0 -> SecureTrust_CA.pem
lrwxrwxrwx root   root         23 Thu, 2024-09-05 04:33:40 etc/ssl/certs/f081611a.0 -> Go_Daddy_Class_2_CA.pem
lrwxrwxrwx root   root         20 Thu, 2024-09-05 04:33:40 etc/ssl/certs/8cb5ee0f.0 -> Amazon_Root_CA_3.pem
lrwxrwxrwx root   root         45 Thu, 2024-09-05 04:33:40 etc/ssl/certs/4bfab552.0 -> Starfield_Root_Certificate_Authority_-_G2.pem
lrwxrwxrwx root   root         22 Thu, 2024-09-05 04:33:40 etc/ssl/certs/76faf6c0.0 -> QuoVadis_Root_CA_3.pem
lrwxrwxrwx root   root         26 Thu, 2024-09-05 04:33:40 etc/ssl/certs/0f6fa695.0 -> GDCA_TrustAUTH_R5_ROOT.pem
lrwxrwxrwx root   root         45 Thu, 2024-09-05 04:33:40 etc/ssl/certs/aee5f10d.0 -> Entrust.net_Premium_2048_Secure_Server_CA.pem
lrwxrwxrwx root   root         27 Thu, 2024-09-05 04:33:40 etc/ssl/certs/dd8e9d41.0 -> DigiCert_Global_Root_G3.pem
lrwxrwxrwx root   root         50 Thu, 2024-09-05 04:33:40 etc/ssl/certs/06dc52d5.0 -> SSL.com_EV_Root_Certification_Authority_RSA_R2.pem
lrwxrwxrwx root   root         47 Thu, 2024-09-05 04:33:40 etc/ssl/certs/f0c70a8d.0 -> SSL.com_EV_Root_Certification_Authority_ECC.pem
lrwxrwxrwx root   root         22 Thu, 2024-09-05 04:33:40 etc/ssl/certs/90c5a3c8.0 -> HiPKI_Root_CA_-_G1.pem
lrwxrwxrwx root   root         27 Thu, 2024-09-05 04:33:40 etc/ssl/certs/607986c7.0 -> DigiCert_Global_Root_G2.pem
lrwxrwxrwx root   root         21 Thu, 2024-09-05 04:33:40 etc/ssl/certs/ed858448.0 -> vTrus_ECC_Root_CA.pem
lrwxrwxrwx root   root         31 Thu, 2024-09-05 04:33:40 etc/ssl/certs/1d3472b9.0 -> GlobalSign_ECC_Root_CA_-_R5.pem
lrwxrwxrwx root   root         23 Thu, 2024-09-05 04:33:40 etc/ssl/certs/5f618aec.0 -> certSIGN_Root_CA_G2.pem
lrwxrwxrwx root   root         20 Thu, 2024-09-05 04:33:40 etc/ssl/certs/b66938e9.0 -> Secure_Global_CA.pem
lrwxrwxrwx root   root         26 Thu, 2024-09-05 04:33:40 etc/ssl/certs/5d3033c5.0 -> TrustCor_RootCert_CA-1.pem
lrwxrwxrwx root   root         35 Thu, 2024-09-05 04:33:40 etc/ssl/certs/e73d606e.0 -> OISTE_WISeKey_Global_Root_GB_CA.pem
lrwxrwxrwx root   root         38 Thu, 2024-09-05 04:33:40 etc/ssl/certs/244b5494.0 -> DigiCert_High_Assurance_EV_Root_CA.pem
lrwxrwxrwx root   root         38 Thu, 2024-09-05 04:33:40 etc/ssl/certs/d6325660.0 -> COMODO_RSA_Certification_Authority.pem
lrwxrwxrwx root   root         41 Thu, 2024-09-05 04:33:40 etc/ssl/certs/fc5a8f99.0 -> USERTrust_RSA_Certification_Authority.pem
lrwxrwxrwx root   root         16 Thu, 2024-09-05 04:33:40 etc/ssl/certs/0b1b94ef.0 -> CFCA_EV_ROOT.pem
lrwxrwxrwx root   root         35 Thu, 2024-09-05 04:33:40 etc/ssl/certs/773e07ad.0 -> OISTE_WISeKey_Global_Root_GC_CA.pem
lrwxrwxrwx root   root         46 Thu, 2024-09-05 04:33:40 etc/ssl/certs/106f3e4d.0 -> Entrust_Root_Certification_Authority_-_EC1.pem
lrwxrwxrwx root   root         26 Thu, 2024-09-05 04:33:40 etc/ssl/certs/4f316efb.0 -> SwissSign_Gold_CA_-_G2.pem
lrwxrwxrwx root   root         34 Thu, 2024-09-05 04:33:40 etc/ssl/certs/c28a8a30.0 -> D-TRUST_Root_Class_3_CA_2_2009.pem
lrwxrwxrwx root   root         23 Thu, 2024-09-05 04:33:40 etc/ssl/certs/002c0b4f.0 -> GlobalSign_Root_R46.pem
lrwxrwxrwx root   root         20 Thu, 2024-09-05 04:33:40 etc/ssl/certs/fd64f3fc.0 -> TunTrust_Root_CA.pem
lrwxrwxrwx root   root         25 Thu, 2024-09-05 04:33:40 etc/ssl/certs/e36a6752.0 -> Atos_TrustedRoot_2011.pem
lrwxrwxrwx root   root         37 Thu, 2024-09-05 04:33:40 etc/ssl/certs/b7a5b843.0 -> TWCA_Root_Certification_Authority.pem
lrwxrwxrwx root   root         45 Thu, 2024-09-05 04:33:40 etc/ssl/certs/3fb36b73.0 -> NAVER_Global_Root_Certification_Authority.pem
lrwxrwxrwx root   root         12 Thu, 2024-09-05 04:33:40 etc/ssl/certs/e113c810.0 -> Certigna.pem
lrwxrwxrwx root   root         32 Thu, 2024-09-05 04:33:40 etc/ssl/certs/0f5dc4f3.0 -> UCA_Extended_Validation_Root.pem
lrwxrwxrwx root   root         31 Thu, 2024-09-05 04:33:40 etc/ssl/certs/7f3d5d1d.0 -> DigiCert_Assured_ID_Root_G3.pem
lrwxrwxrwx root   root         13 Thu, 2024-09-05 04:33:40 etc/ssl/certs/a94d09e5.0 -> ACCVRAIZ1.pem
lrwxrwxrwx root   root         33 Thu, 2024-09-05 04:33:40 etc/ssl/certs/9846683b.0 -> DigiCert_TLS_ECC_P384_Root_G5.pem
lrwxrwxrwx root   root         18 Thu, 2024-09-05 04:33:40 etc/ssl/certs/7aaf71c0.0 -> TrustCor_ECA-1.pem
lrwxrwxrwx root   root         21 Thu, 2024-09-05 04:33:40 etc/ssl/certs/7a780d93.0 -> Certainly_Root_R1.pem
lrwxrwxrwx root   root         44 Thu, 2024-09-05 04:33:40 etc/ssl/certs/988a38cb.0 -> NetLock_Arany_=Class_Gold=_Főtanúsítvány.pem
lrwxrwxrwx root   root         34 Thu, 2024-09-05 04:33:40 etc/ssl/certs/930ac5d2.0 -> Actalis_Authentication_Root_CA.pem
lrwxrwxrwx root   root         27 Thu, 2024-09-05 04:33:40 etc/ssl/certs/68dd7389.0 -> Hongkong_Post_Root_CA_3.pem
lrwxrwxrwx root   root         28 Thu, 2024-09-05 04:33:40 etc/ssl/certs/57bcb2da.0 -> SwissSign_Silver_CA_-_G2.pem
lrwxrwxrwx root   root         25 Thu, 2024-09-05 04:33:40 etc/ssl/certs/749e9e03.0 -> QuoVadis_Root_CA_1_G3.pem
lrwxrwxrwx root   root         27 Thu, 2024-09-05 04:33:40 etc/ssl/certs/dc4d6a89.0 -> GlobalSign_Root_CA_-_R6.pem
lrwxrwxrwx root   root         32 Thu, 2024-09-05 04:33:40 etc/ssl/certs/d52c538d.0 -> DigiCert_TLS_RSA4096_Root_G5.pem
lrwxrwxrwx root   root         34 Thu, 2024-09-05 04:33:40 etc/ssl/certs/8160b96c.0 -> Microsec_e-Szigno_Root_CA_2009.pem
lrwxrwxrwx root   root         28 Thu, 2024-09-05 04:33:40 etc/ssl/certs/75d1b2ed.0 -> DigiCert_Trusted_Root_G4.pem
lrwxrwxrwx root   root         20 Thu, 2024-09-05 04:33:40 etc/ssl/certs/8f103249.0 -> Telia_Root_CA_v2.pem
lrwxrwxrwx root   root         44 Thu, 2024-09-05 04:33:40 etc/ssl/certs/0bf05006.0 -> SSL.com_Root_Certification_Authority_ECC.pem
lrwxrwxrwx root   root         26 Thu, 2024-09-05 04:33:40 etc/ssl/certs/2b349938.0 -> AffirmTrust_Commercial.pem
lrwxrwxrwx root   root         34 Thu, 2024-09-05 04:33:40 etc/ssl/certs/ef954a4e.0 -> IdenTrust_Commercial_Root_CA_1.pem
lrwxrwxrwx root   root         22 Thu, 2024-09-05 04:33:40 etc/ssl/certs/d7e8dc79.0 -> QuoVadis_Root_CA_2.pem
lrwxrwxrwx root   root         22 Thu, 2024-09-05 04:33:40 etc/ssl/certs/5ad8a5d6.0 -> GlobalSign_Root_CA.pem
lrwxrwxrwx root   root         44 Thu, 2024-09-05 04:33:40 etc/ssl/certs/6fa5da56.0 -> SSL.com_Root_Certification_Authority_RSA.pem
lrwxrwxrwx root   root         14 Thu, 2024-09-05 04:33:40 etc/ssl/certs/cc450945.0 -> Izenpe.com.pem
lrwxrwxrwx root   root         20 Thu, 2024-09-05 04:33:40 etc/ssl/certs/2ae6433e.0 -> CA_Disig_Root_R2.pem
lrwxrwxrwx root   root         24 Thu, 2024-09-05 04:33:40 etc/ssl/certs/f387163d.0 -> Starfield_Class_2_CA.pem
lrwxrwxrwx root   root         21 Thu, 2024-09-05 04:33:40 etc/ssl/certs/8508e720.0 -> Certainly_Root_E1.pem
lrwxrwxrwx root   root         40 Thu, 2024-09-05 04:33:40 etc/ssl/certs/6b99d060.0 -> Entrust_Root_Certification_Authority.pem
lrwxrwxrwx root   root         39 Thu, 2024-09-05 04:33:40 etc/ssl/certs/b81b93f0.0 -> AC_RAIZ_FNMT-RCM_SERVIDORES_SEGUROS.pem
lrwxrwxrwx root   root         20 Thu, 2024-09-05 04:33:40 etc/ssl/certs/f51bb24c.0 -> Certigna_Root_CA.pem
lrwxrwxrwx root   root         29 Thu, 2024-09-05 04:33:40 etc/ssl/certs/48bec511.0 -> Certum_Trusted_Network_CA.pem
lrwxrwxrwx root   root         34 Thu, 2024-09-05 04:33:40 etc/ssl/certs/08063a00.0 -> Security_Communication_RootCA3.pem
lrwxrwxrwx root   root         35 Thu, 2024-09-05 04:33:40 etc/ssl/certs/5273a94c.0 -> E-Tugra_Certification_Authority.pem
lrwxrwxrwx root   root         53 Thu, 2024-09-05 04:33:40 etc/ssl/certs/d887a5bb.0 -> Trustwave_Global_ECC_P384_Certification_Authority.pem
lrwxrwxrwx root   root         49 Thu, 2024-09-05 04:33:40 etc/ssl/certs/ff34af3f.0 -> TUBITAK_Kamu_SM_SSL_Kok_Sertifikasi_-_Surum_1.pem
lrwxrwxrwx root   root         23 Thu, 2024-09-05 04:33:40 etc/ssl/certs/18856ac4.0 -> SecureSign_RootCA11.pem
lrwxrwxrwx root   root         27 Thu, 2024-09-05 04:33:40 etc/ssl/certs/3e45d192.0 -> Hongkong_Post_Root_CA_1.pem
lrwxrwxrwx root   root         31 Thu, 2024-09-05 04:33:40 etc/ssl/certs/9d04f354.0 -> DigiCert_Assured_ID_Root_G2.pem
lrwxrwxrwx root   root         27 Thu, 2024-09-05 04:33:40 etc/ssl/certs/9c8dfbd4.0 -> AffirmTrust_Premium_ECC.pem
lrwxrwxrwx root   root         27 Thu, 2024-09-05 04:33:40 etc/ssl/certs/14bc7599.0 -> emSign_ECC_Root_CA_-_G3.pem
lrwxrwxrwx root   root         25 Thu, 2024-09-05 04:33:40 etc/ssl/certs/e868b802.0 -> e-Szigno_Root_CA_2017.pem
lrwxrwxrwx root   root         27 Thu, 2024-09-05 04:33:40 etc/ssl/certs/3513523f.0 -> DigiCert_Global_Root_CA.pem
lrwxrwxrwx root   root         23 Thu, 2024-09-05 04:33:40 etc/ssl/certs/b727005e.0 -> AffirmTrust_Premium.pem
lrwxrwxrwx root   root         49 Thu, 2024-09-05 04:33:40 etc/ssl/certs/8d89cda1.0 -> Microsoft_ECC_Root_Certificate_Authority_2017.pem
lrwxrwxrwx root   root         53 Thu, 2024-09-05 04:33:40 etc/ssl/certs/9b5697b0.0 -> Trustwave_Global_ECC_P256_Certification_Authority.pem
lrwxrwxrwx root   root         34 Thu, 2024-09-05 04:33:40 etc/ssl/certs/cd58d51e.0 -> Security_Communication_RootCA2.pem
lrwxrwxrwx root   root         29 Thu, 2024-09-05 04:33:40 etc/ssl/certs/653b494a.0 -> Baltimore_CyberTrust_Root.pem
lrwxrwxrwx root   root         29 Thu, 2024-09-05 04:33:40 etc/ssl/certs/5931b5bc.0 -> D-TRUST_EV_Root_CA_1_2020.pem
lrwxrwxrwx root   root         31 Thu, 2024-09-05 04:33:40 etc/ssl/certs/ecccd8db.0 -> HARICA_TLS_ECC_Root_CA_2021.pem
lrwxrwxrwx root   root         34 Thu, 2024-09-05 04:33:40 etc/ssl/certs/f3377b1b.0 -> Security_Communication_Root_CA.pem
lrwxrwxrwx root   root         33 Thu, 2024-09-05 04:33:40 etc/ssl/certs/66445960.0 -> E-Tugra_Global_Root_CA_RSA_v3.pem
lrwxrwxrwx root   root         25 Thu, 2024-09-05 04:33:40 etc/ssl/certs/064e0aa9.0 -> QuoVadis_Root_CA_2_G3.pem
lrwxrwxrwx root   root         32 Thu, 2024-09-05 04:33:40 etc/ssl/certs/5443e9e3.0 -> T-TeleSec_GlobalRoot_Class_3.pem
lrwxrwxrwx root   root         15 Thu, 2024-09-05 04:33:40 etc/ssl/certs/a3418fda.0 -> GTS_Root_R4.pem
lrwxrwxrwx root   root         26 Thu, 2024-09-05 04:33:40 etc/ssl/certs/93bc0acc.0 -> AffirmTrust_Networking.pem
lrwxrwxrwx root   root         24 Thu, 2024-09-05 04:33:40 etc/ssl/certs/706f604c.0 -> XRamp_Global_CA_Root.pem
lrwxrwxrwx root   root         15 Thu, 2024-09-05 04:33:40 etc/ssl/certs/626dceaf.0 -> GTS_Root_R2.pem
lrwxrwxrwx root   root         26 Thu, 2024-09-05 04:33:40 etc/ssl/certs/e35234b1.0 -> Certum_Trusted_Root_CA.pem
lrwxrwxrwx root   root         41 Thu, 2024-09-05 04:33:40 etc/ssl/certs/f30dd6ad.0 -> USERTrust_ECC_Certification_Authority.pem
lrwxrwxrwx root   root         45 Thu, 2024-09-05 04:33:40 etc/ssl/certs/02265526.0 -> Entrust_Root_Certification_Authority_-_G2.pem
lrwxrwxrwx root   root         33 Thu, 2024-09-05 04:33:40 etc/ssl/certs/5a7722fb.0 -> E-Tugra_Global_Root_CA_ECC_v3.pem
lrwxrwxrwx root   root         22 Thu, 2024-09-05 04:33:40 etc/ssl/certs/c01eb047.0 -> UCA_Global_G2_Root.pem
lrwxrwxrwx root   root         28 Thu, 2024-09-05 04:33:40 etc/ssl/certs/ee64a828.0 -> Comodo_AAA_Services_root.pem
lrwxrwxrwx root   root         23 Thu, 2024-09-05 04:33:40 etc/ssl/certs/5f15c80c.0 -> TWCA_Global_Root_CA.pem
lrwxrwxrwx root   root         37 Thu, 2024-09-05 04:33:40 etc/ssl/certs/1e08bfd1.0 -> IdenTrust_Public_Sector_Root_CA_1.pem
lrwxrwxrwx root   root         20 Thu, 2024-09-05 04:33:40 etc/ssl/certs/8d86cdd1.0 -> certSIGN_ROOT_CA.pem
lrwxrwxrwx root   root         44 Thu, 2024-09-05 04:33:40 etc/ssl/certs/f249de83.0 -> Trustwave_Global_Certification_Authority.pem
lrwxrwxrwx root   root         16 Thu, 2024-09-05 04:33:40 etc/ssl/certs/0b9bc432.0 -> ISRG_Root_X2.pem
-rw-r--r-- root   root      12332 Thu, 2024-08-15 21:51:02 etc/ssl/openssl.cnf
drwx------ root   root          0 Thu, 2024-08-15 21:51:02 etc/ssl/private
lrwxrwxrwx root   root         21 Wed, 2024-08-14 16:10:00 etc/os-release -> ../usr/lib/os-release
-rw-r--r-- root   root        128 Thu, 2024-09-05 04:33:36 etc/shells
-rw-r--r-- root   root          0 Thu, 2024-09-05 04:33:37 etc/environment
-rw------- root   root          0 Thu, 2024-09-05 04:33:37 etc/.pwd.lock
lrwxrwxrwx root   root         19 Thu, 2024-09-05 04:33:48 etc/mtab -> ../proc/self/mounts
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:33:49 etc/dbus-1
drwxr-xr-x root   root          0 Sat, 2023-09-16 10:03:58 etc/dbus-1/session.d
drwxr-xr-x root   root          0 Sat, 2023-09-16 10:03:58 etc/dbus-1/system.d
-rw-r--r-- root   root       1156 Thu, 2024-09-05 04:34:18 etc/passwd-
-rw-r----- root   shadow      675 Thu, 2024-09-05 04:34:57 etc/shadow-
-rw-r--r-- root   root          0 Thu, 2024-09-05 04:33:38 etc/subgid-
-rw-r--r-- root   root       3310 Tue, 2025-03-18 10:28:08 etc/mailcap
lrwxrwxrwx root   root         13 Sat, 2024-01-20 09:27:07 etc/rmt -> /usr/sbin/rmt
-rw-r--r-- root   root         17 Thu, 2024-09-05 04:33:44 etc/kernel-img.conf
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:33:47 etc/cron.hourly
-rw-r--r-- root   root        102 Thu, 2023-03-02 07:33:55 etc/cron.hourly/.placeholder
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:33:47 etc/cron.monthly
-rw-r--r-- root   root        102 Thu, 2023-03-02 07:33:55 etc/cron.monthly/.placeholder
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:34:00 etc/cron.weekly
-rw-r--r-- root   root        102 Thu, 2023-03-02 07:33:55 etc/cron.weekly/.placeholder
-rwxr-xr-x root   root       1055 Sun, 2023-03-12 22:23:59 etc/cron.weekly/man-db
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:33:47 etc/cron.yearly
-rw-r--r-- root   root        102 Thu, 2023-03-02 07:33:55 etc/cron.yearly/.placeholder
-rw-r--r-- root   root       1042 Thu, 2023-03-02 07:33:55 etc/crontab
drwxr-xr-x root   root          0 Sun, 2024-08-25 17:35:39 etc/binfmt.d
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:33:48 etc/modules-load.d
lrwxrwxrwx root   root         10 Sun, 2024-08-25 17:35:39 etc/modules-load.d/modules.conf -> ../modules
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:33:59 etc/sysctl.d
lrwxrwxrwx root   root         14 Sun, 2024-08-25 17:35:39 etc/sysctl.d/99-sysctl.conf -> ../sysctl.conf
-rw-r--r-- root   root        798 Mon, 2022-12-19 06:06:38 etc/sysctl.d/README.sysctl
drwxr-xr-x root   root          0 Sun, 2024-08-25 17:35:39 etc/tmpfiles.d
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:33:48 etc/xdg
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:33:48 etc/xdg/systemd
lrwxrwxrwx root   root         18 Sun, 2024-08-25 17:35:39 etc/xdg/systemd/user -> ../../systemd/user
-rw-r--r-- root   root         34 Thu, 2024-09-05 04:35:00 etc/cloud-release
-rw-r--r-- root   root       1875 Tue, 2023-01-03 20:49:45 etc/inputrc
-rw-r----- root   shadow      524 Thu, 2024-09-05 04:34:57 etc/gshadow-
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:33:49 etc/perl
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:34:01 etc/perl/Net
-rw-r--r-- root   root        611 Wed, 2023-07-05 08:56:39 etc/perl/Net/libnet.cfg
drwxr-xr-x root   root          0 Tue, 2025-03-18 10:28:01 etc/python3.11
-rw-r--r-- root   root        155 Thu, 2024-05-02 11:59:08 etc/python3.11/sitecustomize.py
-rw-r--r-- root   root      73816 Sat, 2023-02-11 13:45:22 etc/mime.types
-rw-r--r-- root   root        449 Mon, 2021-11-29 13:07:27 etc/mailcap.order
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:33:51 etc/gss
drwxr-xr-x root   root          0 Mon, 2024-07-01 17:31:35 etc/gss/mech.d
-rw-r--r-- root   root        767 Thu, 2022-08-11 14:50:52 etc/netconfig
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:33:59 etc/iproute2
-rw-r--r-- root   root         85 Wed, 2022-12-14 17:42:22 etc/iproute2/bpf_pinning
-rw-r--r-- root   root         81 Wed, 2022-12-14 17:42:22 etc/iproute2/ematch_map
-rw-r--r-- root   root         31 Wed, 2022-12-14 17:42:22 etc/iproute2/group
-rw-r--r-- root   root        262 Wed, 2022-12-14 17:42:22 etc/iproute2/nl_protos
-rw-r--r-- root   root        331 Wed, 2022-12-14 17:42:22 etc/iproute2/rt_dsfield
-rw-r--r-- root   root        219 Wed, 2022-12-14 17:42:22 etc/iproute2/rt_protos
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:33:59 etc/iproute2/rt_protos.d
-rw-r--r-- root   root        144 Wed, 2022-12-14 17:42:22 etc/iproute2/rt_protos.d/README
-rw-r--r-- root   root        112 Wed, 2022-12-14 17:42:22 etc/iproute2/rt_realms
-rw-r--r-- root   root         92 Wed, 2022-12-14 17:42:22 etc/iproute2/rt_scopes
-rw-r--r-- root   root         87 Wed, 2022-12-14 17:42:22 etc/iproute2/rt_tables
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:33:59 etc/iproute2/rt_tables.d
-rw-r--r-- root   root        144 Wed, 2022-12-14 17:42:22 etc/iproute2/rt_tables.d/README
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:34:56 etc/network
-rw-r--r-- root   root        473 Tue, 2025-03-18 10:26:57 etc/network/interfaces
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:34:01 etc/network/if-down.d
-rwxr-xr-x root   root        759 Fri, 2022-12-09 20:37:03 etc/network/if-down.d/resolved
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:34:16 etc/network/if-post-down.d
-rwxr-xr-x root   root        145 Mon, 2023-05-08 20:05:00 etc/network/if-post-down.d/chrony
drwxr-xr-x root   root          0 Tue, 2023-01-24 13:07:32 etc/network/if-pre-up.d
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:34:16 etc/network/if-up.d
-rwxr-xr-x root   root       4663 Fri, 2022-12-09 20:37:03 etc/network/if-up.d/resolved
-rwxr-xr-x root   root        145 Mon, 2023-05-08 20:05:00 etc/network/if-up.d/chrony
drwxr-xr-x root   root          0 Tue, 2023-01-24 13:07:32 etc/network/interfaces.d
-rwxr-xr-x root   root       2024 Thu, 2024-09-05 04:34:56 etc/network/cloud-ifupdown-helper
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:34:17 etc/apparmor.d
-rw-r--r-- root   root       2622 Mon, 2023-05-08 20:05:00 etc/apparmor.d/usr.sbin.chronyd
-rw-r--r-- root   root       3461 Thu, 2023-03-30 09:02:41 etc/apparmor.d/sbin.dhclient
-rw-r--r-- root   root       3448 Sun, 2023-03-12 22:23:59 etc/apparmor.d/usr.bin.man
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:34:17 etc/apparmor.d/local
-rw-r--r-- root   root          0 Thu, 2024-09-05 04:34:00 etc/apparmor.d/local/sbin.dhclient
-rw-r--r-- root   root          0 Thu, 2024-09-05 04:34:00 etc/apparmor.d/local/usr.bin.man
-rw-r--r-- root   root       1110 Tue, 2023-02-14 11:49:15 etc/apparmor.d/local/README
-rw-r--r-- root   root          0 Thu, 2024-09-05 04:34:16 etc/apparmor.d/local/usr.sbin.chronyd
-rw-r--r-- root   root          0 Thu, 2024-09-05 04:34:17 etc/apparmor.d/local/lsb_release
-rw-r--r-- root   root          0 Thu, 2024-09-05 04:34:17 etc/apparmor.d/local/nvidia_modprobe
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:34:17 etc/apparmor.d/abi
-rw-r--r-- root   root       1925 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abi/3.0
-rw-r--r-- root   root       1633 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abi/kernel-5.4-outoftree-network
-rw-r--r-- root   root       1302 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abi/kernel-5.4-vanilla
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:34:17 etc/apparmor.d/abstractions
-rw-r--r-- root   root       1989 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/X
-rw-r--r-- root   root       1119 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/apache2-common
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:34:17 etc/apparmor.d/abstractions/apparmor_api
-rw-r--r-- root   root        420 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/apparmor_api/change_profile
-rw-r--r-- root   root        504 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/apparmor_api/examine
-rw-r--r-- root   root        518 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/apparmor_api/find_mountpoint
-rw-r--r-- root   root        503 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/apparmor_api/introspect
-rw-r--r-- root   root        656 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/apparmor_api/is_enabled
-rw-r--r-- root   root        412 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/aspell
-rw-r--r-- root   root       2035 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/audio
-rw-r--r-- root   root       1857 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/authentication
-rw-r--r-- root   root       6969 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/base
-rw-r--r-- root   root       1614 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/bash
-rw-r--r-- root   root        903 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/consoles
-rw-r--r-- root   root        840 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/crypto
-rw-r--r-- root   root        820 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/cups-client
-rw-r--r-- root   root        694 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/dbus
-rw-r--r-- root   root        745 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/dbus-accessibility
-rw-r--r-- root   root        760 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/dbus-accessibility-strict
-rw-r--r-- root   root       1403 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/dbus-network-manager-strict
-rw-r--r-- root   root        747 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/dbus-session
-rw-r--r-- root   root       1010 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/dbus-session-strict
-rw-r--r-- root   root        781 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/dbus-strict
-rw-r--r-- root   root        344 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/dconf
-rw-r--r-- root   root        675 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/dovecot-common
-rw-r--r-- root   root        542 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/dri-common
-rw-r--r-- root   root        392 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/dri-enumerate
-rw-r--r-- root   root       2220 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/enchant
-rw-r--r-- root   root       1921 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/exo-open
-rw-r--r-- root   root        558 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/fcitx
-rw-r--r-- root   root        821 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/fcitx-strict
-rw-r--r-- root   root       2278 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/fonts
-rw-r--r-- root   root       1147 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/freedesktop.org
-rw-r--r-- root   root       1546 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/gio-open
-rw-r--r-- root   root       3708 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/gnome
-rw-r--r-- root   root        459 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/gnupg
-rw-r--r-- root   root       1490 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/gtk
-rw-r--r-- root   root       1180 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/gvfs-open
-rw-r--r-- root   root        511 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/hosts_access
-rw-r--r-- root   root        992 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/ibus
-rw-r--r-- root   root       3170 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/kde
-rw-r--r-- root   root        413 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/kde-globals-write
-rw-r--r-- root   root        256 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/kde-icon-cache-write
-rw-r--r-- root   root        575 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/kde-language-write
-rw-r--r-- root   root       3699 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/kde-open5
-rw-r--r-- root   root       1281 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/kerberosclient
-rw-r--r-- root   root        856 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/ldapclient
-rw-r--r-- root   root        770 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/libpam-systemd
-rw-r--r-- root   root        595 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/likewise
-rw-r--r-- root   root        554 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/mdns
-rw-r--r-- root   root       1238 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/mesa
-rw-r--r-- root   root        694 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/mir
-rw-r--r-- root   root        573 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/mozc
-rw-r--r-- root   root        739 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/mysql
-rw-r--r-- root   root       3787 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/nameservice
-rw-r--r-- root   root        625 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/nis
-rw-r--r-- root   root       1248 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/nss-systemd
-rw-r--r-- root   root        817 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/nvidia
-rw-r--r-- root   root        370 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/opencl
-rw-r--r-- root   root        516 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/opencl-common
-rw-r--r-- root   root        672 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/opencl-intel
-rw-r--r-- root   root        636 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/opencl-mesa
-rw-r--r-- root   root        895 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/opencl-nvidia
-rw-r--r-- root   root       2912 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/opencl-pocl
-rw-r--r-- root   root        648 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/openssl
-rw-r--r-- root   root        197 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/orbit2
-rw-r--r-- root   root        999 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/p11-kit
-rw-r--r-- root   root        974 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/perl
-rw-r--r-- root   root       1128 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/php
-rw-r--r-- root   root        558 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/php-worker
-rw-r--r-- root   root        208 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/php5
-rw-r--r-- root   root       1356 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/postfix-common
-rw-r--r-- root   root       1660 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/private-files
-rw-r--r-- root   root       1212 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/private-files-strict
-rw-r--r-- root   root       1860 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/python
-rw-r--r-- root   root        863 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/qt5
-rw-r--r-- root   root        399 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/qt5-compose-cache-write
-rw-r--r-- root   root        514 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/qt5-settings-write
-rw-r--r-- root   root        466 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/recent-documents-write
-rw-r--r-- root   root       1008 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/ruby
-rw-r--r-- root   root       1244 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/samba
-rw-r--r-- root   root        817 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/samba-rpcd
-rw-r--r-- root   root        581 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/smbpass
-rw-r--r-- root   root       1613 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/snap_browsers
-rw-r--r-- root   root       1643 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/ssl_certs
-rw-r--r-- root   root        938 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/ssl_keys
-rw-r--r-- root   root       1760 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/svn-repositories
-rw-r--r-- root   root        821 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/ubuntu-bittorrent-clients
-rw-r--r-- root   root       1621 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/ubuntu-browsers
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:34:17 etc/apparmor.d/abstractions/ubuntu-browsers.d
-rw-r--r-- root   root       1018 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/ubuntu-browsers.d/chromium-browser
-rw-r--r-- root   root       3889 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/ubuntu-browsers.d/java
-rw-r--r-- root   root        265 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/ubuntu-browsers.d/kde
-rw-r--r-- root   root        339 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/ubuntu-browsers.d/mailto
-rw-r--r-- root   root       1414 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/ubuntu-browsers.d/multimedia
-rw-r--r-- root   root        351 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/ubuntu-browsers.d/plugins-common
-rw-r--r-- root   root        894 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/ubuntu-browsers.d/productivity
-rw-r--r-- root   root        672 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/ubuntu-browsers.d/text-editors
-rw-r--r-- root   root       1134 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/ubuntu-browsers.d/ubuntu-integration
-rw-r--r-- root   root        185 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/ubuntu-browsers.d/ubuntu-integration-xul
-rw-r--r-- root   root        935 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/ubuntu-browsers.d/user-files
-rw-r--r-- root   root        731 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/ubuntu-console-browsers
-rw-r--r-- root   root        718 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/ubuntu-console-email
-rw-r--r-- root   root       1087 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/ubuntu-email
-rw-r--r-- root   root        456 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/ubuntu-feed-readers
-rw-r--r-- root   root        300 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/ubuntu-gnome-terminal
-rw-r--r-- root   root       3866 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/ubuntu-helpers
-rw-r--r-- root   root        453 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/ubuntu-konsole
-rw-r--r-- root   root       2352 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/ubuntu-media-players
-rw-r--r-- root   root       2558 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/ubuntu-unity7-base
-rw-r--r-- root   root        311 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/ubuntu-unity7-launcher
-rw-r--r-- root   root        313 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/ubuntu-unity7-messaging
-rw-r--r-- root   root        346 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/ubuntu-xterm
-rw-r--r-- root   root        987 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/user-download
-rw-r--r-- root   root        944 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/user-mail
-rw-r--r-- root   root       1000 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/user-manpages
-rw-r--r-- root   root        760 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/user-tmp
-rw-r--r-- root   root        972 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/user-write
-rw-r--r-- root   root        231 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/video
-rw-r--r-- root   root       1085 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/vulkan
-rw-r--r-- root   root        645 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/wayland
-rw-r--r-- root   root        811 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/web-data
-rw-r--r-- root   root        882 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/winbind
-rw-r--r-- root   root        711 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/wutmp
-rw-r--r-- root   root        984 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/xad
-rw-r--r-- root   root        782 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/xdg-desktop
-rw-r--r-- root   root       2285 Tue, 2023-02-14 11:49:15 etc/apparmor.d/abstractions/xdg-open
drwxr-xr-x root   root          0 Tue, 2023-02-14 11:49:15 etc/apparmor.d/disable
drwxr-xr-x root   root          0 Tue, 2023-02-14 11:49:15 etc/apparmor.d/force-complain
-rw-r--r-- root   root       1379 Tue, 2023-02-14 11:49:15 etc/apparmor.d/lsb_release
-rw-r--r-- root   root       1189 Tue, 2023-02-14 11:49:15 etc/apparmor.d/nvidia_modprobe
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:34:17 etc/apparmor.d/tunables
-rw-r--r-- root   root        624 Tue, 2023-02-14 11:49:15 etc/apparmor.d/tunables/alias
-rw-r--r-- root   root        375 Tue, 2023-02-14 11:49:15 etc/apparmor.d/tunables/apparmorfs
-rw-r--r-- root   root        804 Tue, 2023-02-14 11:49:15 etc/apparmor.d/tunables/dovecot
-rw-r--r-- root   root       1077 Tue, 2023-02-14 11:49:15 etc/apparmor.d/tunables/etc
-rw-r--r-- root   root        759 Tue, 2023-02-14 11:49:15 etc/apparmor.d/tunables/global
-rw-r--r-- root   root        982 Tue, 2023-02-14 11:49:15 etc/apparmor.d/tunables/home
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:34:17 etc/apparmor.d/tunables/home.d
-rw-r--r-- root   root        337 Thu, 2024-09-05 04:34:17 etc/apparmor.d/tunables/home.d/ubuntu
-rw-r--r-- root   root        634 Tue, 2023-02-14 11:49:15 etc/apparmor.d/tunables/home.d/site.local
-rw-r--r-- root   root       1391 Tue, 2023-02-14 11:49:15 etc/apparmor.d/tunables/kernelvars
-rw-r--r-- root   root        630 Tue, 2023-02-14 11:49:15 etc/apparmor.d/tunables/multiarch
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:34:17 etc/apparmor.d/tunables/multiarch.d
-rw-r--r-- root   root        645 Tue, 2023-02-14 11:49:15 etc/apparmor.d/tunables/multiarch.d/site.local
-rw-r--r-- root   root        440 Tue, 2023-02-14 11:49:15 etc/apparmor.d/tunables/proc
-rw-r--r-- root   root         23 Tue, 2023-02-14 11:49:15 etc/apparmor.d/tunables/run
-rw-r--r-- root   root        405 Tue, 2023-02-14 11:49:15 etc/apparmor.d/tunables/securityfs
-rw-r--r-- root   root        819 Tue, 2023-02-14 11:49:15 etc/apparmor.d/tunables/share
-rw-r--r-- root   root        378 Tue, 2023-02-14 11:49:15 etc/apparmor.d/tunables/sys
-rw-r--r-- root   root        867 Tue, 2023-02-14 11:49:15 etc/apparmor.d/tunables/xdg-user-dirs
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:34:17 etc/apparmor.d/tunables/xdg-user-dirs.d
-rw-r--r-- root   root        730 Thu, 2024-09-05 04:34:17 etc/apparmor.d/tunables/xdg-user-dirs.d/site.local
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:34:00 etc/dhcp
-rw-r--r-- root   root       1426 Wed, 2022-02-23 09:28:51 etc/dhcp/debug
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:33:52 etc/dhcp/dhclient-enter-hooks.d
lrwxrwxrwx root   root          8 Mon, 2023-04-17 12:20:02 etc/dhcp/dhclient-enter-hooks.d/debug -> ../debug
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:34:16 etc/dhcp/dhclient-exit-hooks.d
-rw-r--r-- root   root       1756 Mon, 2023-04-17 12:20:02 etc/dhcp/dhclient-exit-hooks.d/rfc3442-classless-routes
lrwxrwxrwx root   root          8 Mon, 2023-04-17 12:20:02 etc/dhcp/dhclient-exit-hooks.d/debug -> ../debug
-rw-r--r-- root   root       1388 Sun, 2024-08-25 17:30:20 etc/dhcp/dhclient-exit-hooks.d/timesyncd
-rw-r--r-- root   root        623 Mon, 2023-05-08 20:05:00 etc/dhcp/dhclient-exit-hooks.d/chrony
-rw-r--r-- root   root       1735 Wed, 2022-02-23 09:28:51 etc/dhcp/dhclient.conf
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:34:56 etc/modprobe.d
-rw-r--r-- root   root         34 Thu, 2024-09-05 04:34:56 etc/modprobe.d/blacklist.conf
-rw-r--r-- root   root        494 Wed, 2022-12-14 18:16:50 etc/logrotate.conf
-rw-r--r-- root   root      11424 Mon, 2024-05-06 06:04:37 etc/nanorc
-rw-r--r-- root   root       1853 Mon, 2022-10-17 22:36:59 etc/ethertypes
-rw-r--r-- root   root       3144 Mon, 2022-10-17 22:19:05 etc/protocols
-rw-r--r-- root   root        911 Mon, 2022-10-17 22:24:22 etc/rpc
-rw-r--r-- root   root      12813 Sat, 2021-03-27 22:32:57 etc/services
-rwxr-xr-x root   root        243 Sat, 2023-09-16 05:47:15 etc/nftables.conf
-rw-r--r-- root   root       2355 Mon, 2022-12-19 06:06:38 etc/sysctl.conf
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:33:58 etc/udev
drwxr-xr-x root   root          0 Sun, 2024-08-25 17:35:39 etc/udev/hwdb.d
drwxr-xr-x root   root          0 Sun, 2024-08-25 17:35:39 etc/udev/rules.d
-rw-r--r-- root   root        305 Mon, 2024-08-19 20:25:31 etc/udev/udev.conf
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:33:59 etc/vim
-rw-r--r-- root   root       2553 Thu, 2023-05-04 10:24:44 etc/vim/vimrc
-rw-r--r-- root   root        662 Thu, 2023-05-04 10:24:44 etc/vim/vimrc.tiny
-rw-r--r-- root   root       1260 Fri, 2023-01-27 13:29:51 etc/ucf.conf
-rw-r--r-- root   root         45 Fri, 2020-01-24 22:23:54 etc/bash_completion
-rw-r--r-- root   root        111 Sat, 2023-01-28 18:17:20 etc/magic
-rw-r--r-- root   root        111 Sat, 2023-01-28 18:17:20 etc/magic.mime
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:33:59 etc/groff
-rw-r--r-- root   root       1312 Tue, 2023-03-07 09:38:03 etc/groff/man.local
-rw-r--r-- root   root       1042 Tue, 2023-03-07 09:38:03 etc/groff/mdoc.local
-rw-r--r-- root   root       9431 Tue, 2025-03-18 10:28:01 etc/locale.gen
-rw-r--r-- root   root       2996 Thu, 2024-08-15 09:10:46 etc/locale.alias
-rw-r--r-- root   root       5230 Sun, 2023-03-12 22:23:59 etc/manpath.config
drwxr-xr-x root   root          0 Tue, 2025-03-18 10:26:33 etc/ssh
-rw-r--r-- root   root       1650 Sat, 2024-06-22 19:38:08 etc/ssh/ssh_config
drwxr-xr-x root   root          0 Tue, 2025-03-18 10:28:49 etc/ssh/ssh_config.d
-rw-r--r-- root   root         53 Tue, 2025-03-18 10:28:48 etc/ssh/ssh_config.d/disable_kh.conf
-rw-r--r-- root   root     573928 Sat, 2024-06-22 19:38:08 etc/ssh/moduli
drwxr-xr-x root   root          0 Sat, 2024-06-22 19:38:08 etc/ssh/sshd_config.d
-rw-r--r-- root   root       3285 Thu, 2024-09-05 04:34:58 etc/ssh/sshd_config
-rw-r--r-- root   root       3221 Tue, 2025-03-18 10:26:32 etc/ssh/sshd_config.ucf-dist
-rw------- root   root       2602 Tue, 2025-03-18 10:26:33 etc/ssh/ssh_host_rsa_key
-rw-r--r-- root   root        567 Tue, 2025-03-18 10:26:33 etc/ssh/ssh_host_rsa_key.pub
-rw------- root   root        505 Tue, 2025-03-18 10:26:33 etc/ssh/ssh_host_ecdsa_key
-rw-r--r-- root   root        175 Tue, 2025-03-18 10:26:33 etc/ssh/ssh_host_ecdsa_key.pub
-rw------- root   root        399 Tue, 2025-03-18 10:26:33 etc/ssh/ssh_host_ed25519_key
-rw-r--r-- root   root         95 Tue, 2025-03-18 10:26:33 etc/ssh/ssh_host_ed25519_key.pub
-rw-r--r-- root   root       3229 Mon, 2022-10-24 20:25:58 etc/reportbug.conf
-rw-r--r-- root   root       4942 Sat, 2022-05-14 05:16:15 etc/wgetrc
-rw-r--r-- root   root        627 Thu, 2024-09-05 04:34:57 etc/group-
-rw-r--r-- root   root         60 Thu, 2024-09-05 04:33:59 etc/networks
-rw-r--r-- root   root        152 Tue, 2025-03-18 10:26:46 etc/hosts
-rw-r----- root   shadow      536 Thu, 2024-09-05 04:34:57 etc/gshadow
-rw-r--r-- root   root        280 Fri, 2014-06-20 06:23:50 etc/fuse.conf
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:34:02 etc/python3
-rw-r--r-- root   root         94 Thu, 2024-09-05 04:34:02 etc/python3/debian_config
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:34:16 etc/chrony
-rw-r--r-- root   root       1416 Mon, 2023-05-08 20:05:00 etc/chrony/chrony.conf
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:34:16 etc/chrony/conf.d
-rw-r--r-- root   root        422 Mon, 2023-05-08 20:05:00 etc/chrony/conf.d/README
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:34:16 etc/chrony/sources.d
-rw-r--r-- root   root        498 Mon, 2023-05-08 20:05:00 etc/chrony/sources.d/README
-rw-r----- root   _chrony      481 Mon, 2023-05-08 20:05:00 etc/chrony/chrony.keys
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:34:11 etc/ppp
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:34:16 etc/ppp/ip-down.d
-rwxr-xr-x root   root        416 Mon, 2023-05-08 20:05:00 etc/ppp/ip-down.d/chrony
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:34:16 etc/ppp/ip-up.d
-rwxr-xr-x root   root        402 Mon, 2023-05-08 20:05:00 etc/ppp/ip-up.d/chrony
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:34:11 etc/runit
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:34:11 etc/runit/runsvdir
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:34:18 etc/runit/runsvdir/default
lrwxrwxrwx root   root         11 Thu, 2024-09-05 04:34:18 etc/runit/runsvdir/default/ssh -> /etc/sv/ssh
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:34:11 etc/sv
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:34:17 etc/sv/ssh
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:34:17 etc/sv/ssh/.meta
-rw-r--r-- root   root          0 Sat, 2024-06-22 19:38:08 etc/sv/ssh/.meta/installed
-rwxr-xr-x root   root        470 Sat, 2024-06-22 19:38:08 etc/sv/ssh/finish
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:34:17 etc/sv/ssh/log
-rwxr-xr-x root   root        140 Sat, 2024-06-22 19:38:08 etc/sv/ssh/log/run
-rwxr-xr-x root   root        393 Sat, 2024-06-22 19:38:08 etc/sv/ssh/run
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:34:11 etc/ufw
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:34:17 etc/ufw/applications.d
-rw-r--r-- root   root        145 Sat, 2024-06-22 19:38:08 etc/ufw/applications.d/openssh-server
-rw-r--r-- root   root       4343 Tue, 2023-06-27 11:45:00 etc/sudo.conf
-rw-r--r-- root   root       9804 Tue, 2023-06-27 11:45:00 etc/sudo_logsrvd.conf
-r--r----- root   root       1714 Tue, 2023-06-27 11:45:00 etc/sudoers
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:34:57 etc/sudoers.d
-r--r----- root   root         31 Thu, 2024-09-05 04:34:57 etc/sudoers.d/vagrant
-r--r----- root   root       1096 Tue, 2023-06-27 11:45:00 etc/sudoers.d/README
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:34:17 etc/apparmor
-rw-r--r-- root   root       2396 Tue, 2023-02-14 11:49:15 etc/apparmor/parser.conf
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:34:19 etc/initramfs-tools
drwxr-xr-x root   root          0 Sat, 2024-08-24 23:14:50 etc/initramfs-tools/conf.d
drwxr-xr-x root   root          0 Sat, 2024-08-24 23:14:50 etc/initramfs-tools/hooks
-rw-r--r-- root   root       1583 Mon, 2022-06-20 20:54:17 etc/initramfs-tools/initramfs.conf
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:34:12 etc/initramfs-tools/scripts
drwxr-xr-x root   root          0 Sat, 2024-08-24 23:14:50 etc/initramfs-tools/scripts/init-bottom
drwxr-xr-x root   root          0 Sat, 2024-08-24 23:14:50 etc/initramfs-tools/scripts/init-premount
drwxr-xr-x root   root          0 Sat, 2024-08-24 23:14:50 etc/initramfs-tools/scripts/init-top
drwxr-xr-x root   root          0 Sat, 2024-08-24 23:14:50 etc/initramfs-tools/scripts/local-bottom
drwxr-xr-x root   root          0 Sat, 2024-08-24 23:14:50 etc/initramfs-tools/scripts/local-premount
drwxr-xr-x root   root          0 Sat, 2024-08-24 23:14:50 etc/initramfs-tools/scripts/local-top
drwxr-xr-x root   root          0 Sat, 2024-08-24 23:14:50 etc/initramfs-tools/scripts/nfs-bottom
drwxr-xr-x root   root          0 Sat, 2024-08-24 23:14:50 etc/initramfs-tools/scripts/nfs-premount
drwxr-xr-x root   root          0 Sat, 2024-08-24 23:14:50 etc/initramfs-tools/scripts/nfs-top
drwxr-xr-x root   root          0 Sat, 2024-08-24 23:14:50 etc/initramfs-tools/scripts/panic
-rw-r--r-- root   root        378 Mon, 2014-10-27 20:54:27 etc/initramfs-tools/update-initramfs.conf
-rw-r--r-- root   root        246 Thu, 2024-09-05 04:34:19 etc/initramfs-tools/modules
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:34:19 etc/grub.d
-rwxr-xr-x root   root      10046 Mon, 2023-10-02 14:11:34 etc/grub.d/00_header
-rwxr-xr-x root   root       6260 Mon, 2023-10-02 14:11:34 etc/grub.d/05_debian_theme
-rwxr-xr-x root   root      14123 Mon, 2023-10-02 14:11:34 etc/grub.d/10_linux
-rwxr-xr-x root   root      14180 Mon, 2023-10-02 14:11:34 etc/grub.d/20_linux_xen
-rwxr-xr-x root   root      12923 Mon, 2023-10-02 14:11:34 etc/grub.d/30_os-prober
-rwxr-xr-x root   root       1372 Mon, 2023-10-02 14:11:34 etc/grub.d/30_uefi-firmware
-rwxr-xr-x root   root        214 Mon, 2023-10-02 14:11:34 etc/grub.d/40_custom
-rwxr-xr-x root   root        215 Mon, 2023-10-02 14:11:34 etc/grub.d/41_custom
-rw-r--r-- root   root        483 Mon, 2023-10-02 14:11:34 etc/grub.d/README
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:34:16 etc/logcheck
drwxr-xr-x root   root          0 Tue, 2025-03-18 10:28:02 etc/logcheck/ignore.d.server
-rw-r--r-- root   root        656 Wed, 2023-02-22 19:43:00 etc/logcheck/ignore.d.server/rsyslog
-rw-r--r-- root   root       1347 Thu, 2022-09-01 22:08:12 etc/logcheck/ignore.d.server/gpg-agent
-rw-r--r-- root   root       1430 Wed, 2023-02-22 19:43:00 etc/rsyslog.conf
drwxr-xr-x root   root          0 Wed, 2023-02-22 19:43:00 etc/rsyslog.d
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:34:16 etc/pm
drwxr-xr-x root   root          0 Thu, 2024-09-05 04:34:18 etc/pm/sleep.d
-rwxr-xr-x root   root        672 Sat, 2022-12-31 20:59:00 etc/pm/sleep.d/10_unattended-upgrades-hibernate
-rw-r--r-- root   root        643 Thu, 2024-09-05 04:34:57 etc/group
-rw-r--r-- root   root        411 Thu, 2024-09-05 04:34:17 etc/hosts.allow
-rw-r--r-- root   root        711 Thu, 2024-09-05 04:34:17 etc/hosts.deny
-rw-r--r-- root   root         20 Tue, 2025-03-18 10:26:54 etc/resolv.conf
-rw-r----- root   shadow      704 Thu, 2024-09-05 04:34:57 etc/shadow
-rw-r--r-- root   root          7 Tue, 2025-03-18 10:26:46 etc/mailname
-rw-r--r-- root   root          0 Thu, 2024-09-05 04:33:38 etc/subuid-
-rw-r--r-- root   root        390 Tue, 2025-03-18 10:27:00 etc/fstab
-rw-r--r-- root   root       3886 Sat, 2023-01-14 17:24:22 etc/gprofng.rc
-rw-r--r-- root   root       1201 Thu, 2024-09-05 04:34:57 etc/passwd
-rw-r--r-- root   root         21 Thu, 2024-09-05 04:34:57 etc/subuid
-rw-r--r-- root   root         21 Thu, 2024-09-05 04:34:57 etc/subgid
drwxr-xr-x root   root          0 Tue, 2025-03-18 10:28:03 etc/fonts
drwxr-xr-x root   root          0 Tue, 2025-03-18 10:28:01 etc/fonts/conf.avail
-rw-r--r-- root   root        874 Sat, 2016-07-30 08:37:51 etc/fonts/conf.avail/20-unhint-small-dejavu-lgc-sans-mono.conf
-rw-r--r-- root   root        864 Sat, 2016-07-30 08:37:51 etc/fonts/conf.avail/20-unhint-small-dejavu-lgc-sans.conf
-rw-r--r-- root   root        866 Sat, 2016-07-30 08:37:51 etc/fonts/conf.avail/20-unhint-small-dejavu-lgc-serif.conf
-rw-r--r-- root   root        866 Sat, 2016-07-30 08:37:51 etc/fonts/conf.avail/20-unhint-small-dejavu-sans-mono.conf
-rw-r--r-- root   root        856 Sat, 2016-07-30 08:37:51 etc/fonts/conf.avail/20-unhint-small-dejavu-sans.conf
-rw-r--r-- root   root        858 Sat, 2016-07-30 08:37:51 etc/fonts/conf.avail/20-unhint-small-dejavu-serif.conf
-rw-r--r-- root   root       1201 Fri, 2023-03-10 08:35:35 etc/fonts/conf.avail/57-dejavu-sans-mono.conf
-rw-r--r-- root   root       1711 Fri, 2023-03-10 08:35:35 etc/fonts/conf.avail/57-dejavu-sans.conf
-rw-r--r-- root   root       1357 Fri, 2023-03-10 08:35:35 etc/fonts/conf.avail/57-dejavu-serif.conf
-rw-r--r-- root   root       1545 Sat, 2016-07-30 08:37:51 etc/fonts/conf.avail/58-dejavu-lgc-sans-mono.conf
-rw-r--r-- root   root       2063 Sat, 2016-07-30 08:37:51 etc/fonts/conf.avail/58-dejavu-lgc-sans.conf
-rw-r--r-- root   root       1689 Sat, 2016-07-30 08:37:51 etc/fonts/conf.avail/58-dejavu-lgc-serif.conf
drwxr-xr-x root   root          0 Tue, 2025-03-18 10:28:03 etc/fonts/conf.d
lrwxrwxrwx root   root         55 Fri, 2023-03-10 08:35:35 etc/fonts/conf.d/20-unhint-small-dejavu-lgc-sans-mono.conf -> ../conf.avail/20-unhint-small-dejavu-lgc-sans-mono.conf
lrwxrwxrwx root   root         50 Fri, 2023-03-10 08:35:35 etc/fonts/conf.d/20-unhint-small-dejavu-lgc-sans.conf -> ../conf.avail/20-unhint-small-dejavu-lgc-sans.conf
lrwxrwxrwx root   root         51 Fri, 2023-03-10 08:35:35 etc/fonts/conf.d/20-unhint-small-dejavu-lgc-serif.conf -> ../conf.avail/20-unhint-small-dejavu-lgc-serif.conf
lrwxrwxrwx root   root         51 Fri, 2023-03-10 08:35:35 etc/fonts/conf.d/20-unhint-small-dejavu-sans-mono.conf -> ../conf.avail/20-unhint-small-dejavu-sans-mono.conf
lrwxrwxrwx root   root         46 Fri, 2023-03-10 08:35:35 etc/fonts/conf.d/20-unhint-small-dejavu-sans.conf -> ../conf.avail/20-unhint-small-dejavu-sans.conf
lrwxrwxrwx root   root         47 Fri, 2023-03-10 08:35:35 etc/fonts/conf.d/20-unhint-small-dejavu-serif.conf -> ../conf.avail/20-unhint-small-dejavu-serif.conf
lrwxrwxrwx root   root         38 Fri, 2023-03-10 08:35:35 etc/fonts/conf.d/57-dejavu-sans-mono.conf -> ../conf.avail/57-dejavu-sans-mono.conf
lrwxrwxrwx root   root         33 Fri, 2023-03-10 08:35:35 etc/fonts/conf.d/57-dejavu-sans.conf -> ../conf.avail/57-dejavu-sans.conf
lrwxrwxrwx root   root         34 Fri, 2023-03-10 08:35:35 etc/fonts/conf.d/57-dejavu-serif.conf -> ../conf.avail/57-dejavu-serif.conf
lrwxrwxrwx root   root         42 Fri, 2023-03-10 08:35:35 etc/fonts/conf.d/58-dejavu-lgc-sans-mono.conf -> ../conf.avail/58-dejavu-lgc-sans-mono.conf
lrwxrwxrwx root   root         37 Fri, 2023-03-10 08:35:35 etc/fonts/conf.d/58-dejavu-lgc-sans.conf -> ../conf.avail/58-dejavu-lgc-sans.conf
lrwxrwxrwx root   root         38 Fri, 2023-03-10 08:35:35 etc/fonts/conf.d/58-dejavu-lgc-serif.conf -> ../conf.avail/58-dejavu-lgc-serif.conf
-rw-r--r-- root   root        979 Tue, 2023-01-31 22:24:08 etc/fonts/conf.d/README
lrwxrwxrwx root   root         59 Tue, 2023-01-31 22:24:08 etc/fonts/conf.d/10-scale-bitmap-fonts.conf -> /usr/share/fontconfig/conf.avail/10-scale-bitmap-fonts.conf
lrwxrwxrwx root   root         54 Tue, 2023-01-31 22:24:08 etc/fonts/conf.d/10-yes-antialias.conf -> /usr/share/fontconfig/conf.avail/10-yes-antialias.conf
lrwxrwxrwx root   root         58 Tue, 2023-01-31 22:24:08 etc/fonts/conf.d/11-lcdfilter-default.conf -> /usr/share/fontconfig/conf.avail/11-lcdfilter-default.conf
lrwxrwxrwx root   root         58 Tue, 2023-01-31 22:24:08 etc/fonts/conf.d/20-unhint-small-vera.conf -> /usr/share/fontconfig/conf.avail/20-unhint-small-vera.conf
lrwxrwxrwx root   root         55 Tue, 2023-01-31 22:24:08 etc/fonts/conf.d/30-metric-aliases.conf -> /usr/share/fontconfig/conf.avail/30-metric-aliases.conf
lrwxrwxrwx root   root         49 Tue, 2023-01-31 22:24:08 etc/fonts/conf.d/40-nonlatin.conf -> /usr/share/fontconfig/conf.avail/40-nonlatin.conf
lrwxrwxrwx root   root         48 Tue, 2023-01-31 22:24:08 etc/fonts/conf.d/45-generic.conf -> /usr/share/fontconfig/conf.avail/45-generic.conf
lrwxrwxrwx root   root         46 Tue, 2023-01-31 22:24:08 etc/fonts/conf.d/45-latin.conf -> /usr/share/fontconfig/conf.avail/45-latin.conf
lrwxrwxrwx root   root         48 Tue, 2023-01-31 22:24:08 etc/fonts/conf.d/48-spacing.conf -> /usr/share/fontconfig/conf.avail/48-spacing.conf
lrwxrwxrwx root   root         50 Tue, 2023-01-31 22:24:08 etc/fonts/conf.d/49-sansserif.conf -> /usr/share/fontconfig/conf.avail/49-sansserif.conf
lrwxrwxrwx root   root         45 Tue, 2023-01-31 22:24:08 etc/fonts/conf.d/50-user.conf -> /usr/share/fontconfig/conf.avail/50-user.conf
lrwxrwxrwx root   root         46 Tue, 2023-01-31 22:24:08 etc/fonts/conf.d/51-local.conf -> /usr/share/fontconfig/conf.avail/51-local.conf
lrwxrwxrwx root   root         48 Tue, 2023-01-31 22:24:08 etc/fonts/conf.d/60-generic.conf -> /usr/share/fontconfig/conf.avail/60-generic.conf
lrwxrwxrwx root   root         46 Tue, 2023-01-31 22:24:08 etc/fonts/conf.d/60-latin.conf -> /usr/share/fontconfig/conf.avail/60-latin.conf
lrwxrwxrwx root   root         54 Tue, 2023-01-31 22:24:08 etc/fonts/conf.d/65-fonts-persian.conf -> /usr/share/fontconfig/conf.avail/65-fonts-persian.conf
lrwxrwxrwx root   root         49 Tue, 2023-01-31 22:24:08 etc/fonts/conf.d/65-nonlatin.conf -> /usr/share/fontconfig/conf.avail/65-nonlatin.conf
lrwxrwxrwx root   root         48 Tue, 2023-01-31 22:24:08 etc/fonts/conf.d/69-unifont.conf -> /usr/share/fontconfig/conf.avail/69-unifont.conf
lrwxrwxrwx root   root         50 Tue, 2023-01-31 22:24:08 etc/fonts/conf.d/80-delicious.conf -> /usr/share/fontconfig/conf.avail/80-delicious.conf
lrwxrwxrwx root   root         50 Tue, 2023-01-31 22:24:08 etc/fonts/conf.d/90-synthetic.conf -> /usr/share/fontconfig/conf.avail/90-synthetic.conf
lrwxrwxrwx root   root         55 Tue, 2025-03-18 10:28:03 etc/fonts/conf.d/10-hinting-slight.conf -> /usr/share/fontconfig/conf.avail/10-hinting-slight.conf
lrwxrwxrwx root   root         51 Tue, 2025-03-18 10:28:03 etc/fonts/conf.d/70-no-bitmaps.conf -> /usr/share/fontconfig/conf.avail/70-no-bitmaps.conf
-rw-r--r-- root   root       2940 Tue, 2023-01-31 22:24:08 etc/fonts/fonts.conf
drwxr-xr-x root   root          0 Tue, 2025-03-18 10:27:55 etc/X11
drwxr-xr-x root   root          0 Tue, 2025-03-18 10:28:02 etc/X11/Xsession.d
-rw-r--r-- root   root        880 Thu, 2022-09-01 22:08:12 etc/X11/Xsession.d/90gpg-agent
drwxr-xr-x root   root          0 Tue, 2025-03-18 10:27:55 etc/apache2
drwxr-xr-x root   root          0 Tue, 2025-03-18 10:28:00 etc/apache2/conf-available
-rw-r--r-- root   root        127 Fri, 2020-12-18 19:33:11 etc/apache2/conf-available/javascript-common.conf
drwxr-xr-x root   root          0 Tue, 2025-03-18 10:28:00 etc/lighttpd
drwxr-xr-x root   root          0 Tue, 2025-03-18 10:28:00 etc/lighttpd/conf-available
-rw-r--r-- root   root         56 Sun, 2013-07-28 21:48:58 etc/lighttpd/conf-available/90-javascript-alias.conf
drwxr-xr-x root   root          0 Tue, 2025-03-18 10:28:00 etc/lighttpd/conf-enabled
lrwxrwxrwx root   root         42 Tue, 2025-03-18 10:28:00 etc/lighttpd/conf-enabled/90-javascript-alias.conf -> ../conf-available/90-javascript-alias.conf
-rw-r--r-- root   root      20015 Tue, 2025-03-18 10:28:06 etc/ld.so.cache
drwxr-xr-x root   root          0 Tue, 2025-03-18 16:49:44 etc/test
-rw-r--r-- root   root          0 Tue, 2025-03-18 16:49:44 etc/test/1
```
</details>

    
**Итак, видим что BORG работает и резервные копии папки /etc создаются.**

