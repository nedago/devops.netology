## Домашнее задание к занятию "3.4. Операционные системы, лекция 2" - Денис Поляков

### Задание №1
На лекции мы познакомились с [node_exporter](https://github.com/prometheus/node_exporter/releases). В демонстрации его исполняемый файл запускался в background. Этого достаточно для демо, но не для настоящей production-системы, где процессы должны находиться под внешним управлением. Используя знания из лекции по systemd, создайте самостоятельно простой [unit-файл](https://www.freedesktop.org/software/systemd/man/systemd.service.html) для node_exporter:

    * поместите его в автозагрузку,
    * предусмотрите возможность добавления опций к запускаемому процессу через внешний файл (посмотрите, например, на `systemctl cat cron`),
    * удостоверьтесь, что с помощью systemctl процесс корректно стартует, завершается, а после перезагрузки автоматически поднимается.

[Статья: "Systemd за пять минут"](https://habr.com/ru/company/southbridge/blog/255845/)   
Помещаю юнит в `/etc/systemd/system`
```
[Unit]
Description=Node Exporter
After=syslog.target
After=network.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
#ExecStart=/usr/local/bin/node_exporter --collector.nfs --collector.nfsd
ExecStart=/bin/sh -c '/usr/local/bin/node_exporter $OPTIONS_NODE_EXPORTER'

Restart=always

[Install]
WantedBy=multi-user.target
```

Перегружаю systemd:
```
vagrant@front:/etc/systemd/system$ systemctl daemon-reload
==== AUTHENTICATING FOR org.freedesktop.systemd1.reload-daemon ===
Authentication is required to reload the systemd state.
Authenticating as: vagrant,,, (vagrant)
Password:
==== AUTHENTICATION COMPLETE ===
```
Проверяю:
```
vagrant@front:/etc/systemd/system$ systemctl status node_exporter
● node_exporter.service - MyUnit_node_exporter
     Loaded: loaded (/etc/systemd/system/node_exporter.service; disabled; vendor preset: enabled)
     Active: inactive (dead)`
Включить/выключить автозапуск юнита при загрузке системы:  
```
Помещаю юнит в автозагрузку:
```
vagrant@front:/etc/systemd/system$ sudo systemctl enable node_exporter
Created symlink /etc/systemd/system/multi-user.target.wants/node_exporter.service → /etc/systemd/system/node_exporter.service.
```
Смотрю корректность работы:
```
vagrant@front:/etc/systemd/system$ systemctl status node_exporter
● node_exporter.service - MyUnit_node_exporter
     Loaded: loaded (/etc/systemd/system/node_exporter.service; enabled; vendor preset: enabled)
     Active: active (running) since Sun 2021-11-21 09:24:36 UTC; 1h 54min ago
   Main PID: 1543 (node_exporter)
      Tasks: 4 (limit: 1073)
     Memory: 2.4M
     CGroup: /system.slice/node_exporter.service
             └─1543 /usr/local/bin/node_exporter

Nov 21 09:24:36 front node_exporter[1543]: ts=2021-11-21T09:24:36.812Z caller=node_exporter.go:115 level=info collector=thermal_zone
Nov 21 09:24:36 front node_exporter[1543]: ts=2021-11-21T09:24:36.812Z caller=node_exporter.go:115 level=info collector=time
Nov 21 09:24:36 front node_exporter[1543]: ts=2021-11-21T09:24:36.812Z caller=node_exporter.go:115 level=info collector=timex
Nov 21 09:24:36 front node_exporter[1543]: ts=2021-11-21T09:24:36.812Z caller=node_exporter.go:115 level=info collector=udp_queues
Nov 21 09:24:36 front node_exporter[1543]: ts=2021-11-21T09:24:36.812Z caller=node_exporter.go:115 level=info collector=uname
Nov 21 09:24:36 front node_exporter[1543]: ts=2021-11-21T09:24:36.812Z caller=node_exporter.go:115 level=info collector=vmstat
Nov 21 09:24:36 front node_exporter[1543]: ts=2021-11-21T09:24:36.812Z caller=node_exporter.go:115 level=info collector=xfs
Nov 21 09:24:36 front node_exporter[1543]: ts=2021-11-21T09:24:36.812Z caller=node_exporter.go:115 level=info collector=zfs
Nov 21 09:24:36 front node_exporter[1543]: ts=2021-11-21T09:24:36.812Z caller=node_exporter.go:199 level=info msg="Listening on" address=:9100
Nov 21 09:24:36 front node_exporter[1543]: ts=2021-11-21T09:24:36.815Z caller=tls_config.go:195 level=info msg="TLS is disabled." http2=false
vagrant@front:/etc/systemd/system$
```
###  Задание 2
Ознакомьтесь с опциями node_exporter и выводом `/metrics` по-умолчанию. Приведите несколько опций, которые вы бы выбрали для базового мониторинга хоста по CPU, памяти, диску и сети.
Примеры для постороения графиrов в Grafana:
```
**CPU: node_cpu_seconds_total**
100 - (avg(rate(node_cpu_seconds_total{instance=~"$node",mode="idle"}[$interval])) * 100)
**RAM: node_memory_MemAvailable_bytes**
avg(rate(node_cpu_seconds_total{instance=~"$node",mode="iowait"}[$interval])) * 100
**Disk: node_filesystem_free_bytes**
(node_filesystem_size_bytes{instance=~'$node',fstype=~"ext.*|xfs",mountpoint="$maxmount"}-node_filesystem_free_bytes{instance=~'$node',fstype=~"ext.*|xfs",mountpoint="$maxmount"})*100 /(node_filesystem_avail_bytes {instance=~'$node',fstype=~"ext.*|xfs",mountpoint="$maxmount"}+(node_filesystem_size_bytes{instance=~'$node',fstype=~"ext.*|xfs",mountpoint="$maxmount"}-node_filesystem_free_bytes{instance=~'$node',fstype=~"ext.*|xfs",mountpoint="$maxmount"}))
**Network: node_network_receive_bytes_total ;node_network_transmit_bytes_total**
(1 - ((node_memory_SwapFree_bytes{instance=~"$node"} + 1)/ (node_memory_SwapTotal_bytes{instance=~"$node"} + 1))) * 100
```
### Задание 3
Установите в свою виртуальную машину [Netdata](https://github.com/netdata/netdata). Воспользуйтесь [готовыми пакетами](https://packagecloud.io/netdata/netdata/install) для установки (`sudo apt install -y netdata`). После успешной установки:
    * в конфигурационном файле `/etc/netdata/netdata.conf` в секции [web] замените значение с localhost на `bind to = 0.0.0.0`,
    * добавьте в Vagrantfile проброс порта Netdata на свой локальный компьютер и сделайте `vagrant reload`:

    ```bash
    config.vm.network "forwarded_port", guest: 19999, host: 19999
    ```

    После успешной перезагрузки в браузере *на своем ПК* (не в виртуальной машине) вы должны суметь зайти на `localhost:19999`. Ознакомьтесь с метриками, которые по умолчанию собираются Netdata и с комментариями, которые даны к этим метрикам.

- Netdata: ![Netdata](img/homework_03_01.png)

### Задание 4
Можно ли по выводу `dmesg` понять, осознает ли ОС, что загружена не на настоящем оборудовании, а на системе виртуализации?
```
vagrant@front:~$ dmesg -T | grep "virt"
[Sun Nov 21 12:43:42 2021] CPU MTRRs all blank - virtualized system.
[Sun Nov 21 12:43:42 2021] Booting paravirtualized kernel on KVM
[Sun Nov 21 12:43:42 2021] Performance Events: PMU not available due to virtualization, using software events only.
[Sun Nov 21 12:43:48 2021] systemd[1]: Detected virtualization oracle.
```
### Задание 5
Как настроен sysctl `fs.nr_open` на системе по-умолчанию? Узнайте, что означает этот параметр. Какой другой существующий лимит не позволит достичь такого числа (`ulimit --help`)?
```
vagrant@front:~$ /sbin/sysctl -n fs.nr_open
1048576
```
fs.nr_open - системное ограничение кол-ва возможных открытых файлов. Так же можно проверить командой ulimit -aH (жесткие огранияения)
ulimit -aH или ulimit -aS (жесткие или мягкие ограничения для shell)
```
vagrant@front:~$ ulimit -aS | grep "open file"
open files                      (-n) 1024
vagrant@front:~$ ulimit -aH | grep "open file"
open files                      (-n) 1048576
vagrant@front:~$
```
### Задание 6
Запустите любой долгоживущий процесс (не `ls`, который отработает мгновенно, а, например, `sleep 1h`) в отдельном неймспейсе процессов; покажите, что ваш процесс работает под PID 1 через `nsenter`. Для простоты работайте в данном задании под root (`sudo -i`). Под обычным пользователем требуются дополнительные опции (`--map-root-user`) и т.д.



12. Найдите информацию о том, что такое `:(){ :|:& };:`. Запустите эту команду в своей виртуальной машине Vagrant с Ubuntu 20.04 (**это важно, поведение в других ОС не проверялось**). Некоторое время все будет "плохо", после чего (минуты) – ОС должна стабилизироваться. Вызов `dmesg` расскажет, какой механизм помог автоматической стабилизации. Как настроен этот механизм по-умолчанию, и как изменить число процессов, которое можно создать в сессии?

 
 ---

## Как сдавать задания

Обязательными к выполнению являются задачи без указания звездочки. Их выполнение необходимо для получения зачета и диплома о профессиональной переподготовке.

Задачи со звездочкой (*) являются дополнительными задачами и/или задачами повышенной сложности. Они не являются обязательными к выполнению, но помогут вам глубже понять тему.

Домашнее задание выполните в файле readme.md в github репозитории. В личном кабинете отправьте на проверку ссылку на .md-файл в вашем репозитории.

Также вы можете выполнить задание в [Google Docs](https://docs.google.com/document/u/0/?tgif=d) и отправить в личном кабинете на проверку ссылку на ваш документ.
Название файла Google Docs должно содержать номер лекции и фамилию студента. Пример названия: "1.1. Введение в DevOps — Сусанна Алиева".

Если необходимо прикрепить дополнительные ссылки, просто добавьте их в свой Google Docs.

Перед тем как выслать ссылку, убедитесь, что ее содержимое не является приватным (открыто на комментирование всем, у кого есть ссылка), иначе преподаватель не сможет проверить работу. Чтобы это проверить, откройте ссылку в браузере в режиме инкогнито.

[Как предоставить доступ к файлам и папкам на Google Диске](https://support.google.com/docs/answer/2494822?hl=ru&co=GENIE.Platform%3DDesktop)

[Как запустить chrome в режиме инкогнито ](https://support.google.com/chrome/answer/95464?co=GENIE.Platform%3DDesktop&hl=ru)

[Как запустить  Safari в режиме инкогнито ](https://support.apple.com/ru-ru/guide/safari/ibrw1069/mac)

Любые вопросы по решению задач задавайте в чате Slack.

---
