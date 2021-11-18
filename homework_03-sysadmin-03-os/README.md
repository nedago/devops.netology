## Домашнее задание к занятию "3.3. Операционные системы, лекция 1" - Денис Поляков

### Задание 1
Какой системный вызов делает команда `cd`? В прошлом ДЗ мы выяснили, что `cd` не является самостоятельной  программой, это `shell builtin`, поэтому запустить `strace` непосредственно на `cd` не получится. Тем не менее, вы можете запустить `strace` на `/bin/bash -c 'cd /tmp'`. В этом случае вы увидите полный список системных вызовов, которые делает сам `bash` при старте. Вам нужно найти тот единственный, который относится именно к `cd`.
```  
chdir("/tmp")                           = 0  
```  
### Задание 2
Попробуйте использовать команду `file` на объекты разных типов на файловой системе. Например:
```  
vagrant@netology1:~$ file /dev/tty  
/dev/tty: character special (5/0)  
vagrant@netology1:~$ file /dev/sda  
/dev/sda: block special (8/0)  
vagrant@netology1:~$ file /bin/bash  
/bin/bash: ELF 64-bit LSB shared object, x86-64  
```
Используя `strace` выясните, где находится база данных `file` на основании которой она делает свои догадки.
```  
vagrant@front:~$ strace -e openat file /dev/sda  
openat(AT_FDCWD, "/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3  
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libmagic.so.1", O_RDONLY|O_CLOEXEC) = 3  
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libc.so.6", O_RDONLY|O_CLOEXEC) = 3  
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/liblzma.so.5", O_RDONLY|O_CLOEXEC) = 3  
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libbz2.so.1.0", O_RDONLY|O_CLOEXEC) = 3  
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libz.so.1", O_RDONLY|O_CLOEXEC) = 3  
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libpthread.so.0", O_RDONLY|O_CLOEXEC) = 3  
openat(AT_FDCWD, "/usr/lib/locale/locale-archive", O_RDONLY|O_CLOEXEC) = 3  
openat(AT_FDCWD, "/etc/magic.mgc", O_RDONLY) = -1 ENOENT (No such file or directory)  
openat(AT_FDCWD, "/etc/magic", O_RDONLY) = 3  
openat(AT_FDCWD, "/usr/share/misc/magic.mgc", O_RDONLY) = 3  
openat(AT_FDCWD, "/usr/lib/x86_64-linux-gnu/gconv/gconv-modules.cache", O_RDONLY) = 3  
/dev/sda: block special (8/0)  
+++ exited with 0 +++  
```
Первыми строками идут обращения к библиотекам. Базы,  вероятно, хранятся тут: `/etc/magic` `/usr/share/misc/magic.mgc`
 
### Задание 3
Предположим, приложение пишет лог в текстовый файл. Этот файл оказался удален (deleted в lsof), однако возможности сигналом сказать приложению переоткрыть файлы или просто перезапустить приложение – нет. Так как приложение продолжает писать в удаленный файл, место на диске постепенно заканчивается. Основываясь на знаниях о перенаправлении потоков предложите способ обнуления открытого удаленного файла (чтобы освободить место на файловой системе).
```
ps aux | grep "приложение"  
#смотрим PID  
lsof -p PID | grep "deleted"  
#смотрив в выводе номер дескриптора к примеру "3"  
cat /proc/PID/fd/3 > /tmp/файл_приемник  
#сохраняем вывод в резервный файл  
cat /dev/null > proc/PID/fd/3  
```

### Задание 4
Занимают ли зомби-процессы какие-то ресурсы в ОС (CPU, RAM, IO)?
> Зомби- Процесс при завершении (как нормальном, так и в результате не обрабатываемого сигнала) освобождает все свои ресурсы и становится «зомби» — пустой записью в таблице процессов, хранящей статус завершения, предназначенный для чтения родительским процессом.
- Зомби не занимают ресурсов в ОС.

### Задание 5
В iovisor BCC есть утилита `opensnoop`:
```bash  
root@vagrant:~# dpkg -L bpfcc-tools | grep sbin/opensnoop  
/usr/sbin/opensnoop-bpfcc  
```  
На какие файлы вы увидели вызовы группы `open` за первую секунду работы утилиты? Воспользуйтесь пакетом `bpfcc-tools` для Ubuntu 20.04. Дополнительные [сведения по установке](https://github.com/iovisor/bcc/blob/master/INSTALL.md).
```  
root@front:/home/vagrant# opensnoop-bpfcc -d1  
PID    COMM               FD ERR PATH  
796    vminfo              4   0 /var/run/utmp  
588    dbus-daemon        -1   2 /usr/local/share/dbus-1/system-services  
588    dbus-daemon        18   0 /usr/share/dbus-1/system-services  
588    dbus-daemon        -1   2 /lib/dbus-1/system-services  
588    dbus-daemon        18   0 /var/lib/snapd/dbus-1/system-services/  
1      systemd            12   0 /proc/394/cgroup  
root@front:/home/vagrant#   
```
### Задание 6
Какой системный вызов использует `uname -a`? Приведите цитату из man по этому системному вызову, где описывается альтернативное местоположение в `/proc`, где можно узнать версию ядра и релиз ОС.
```  
uname({sysname="Linux", nodename="front", ...}) = 0  
uname({sysname="Linux", nodename="front", ...}) = 0  
write(1, "Linux front 5.4.0-58-generic #64"..., 103Linux front 5.4.0-58-generic #64-Ubuntu SMP Wed Dec 9 08:16:25 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux  
```
>   
> DESCRIPTION          
>       uname() returns system information in the structure pointed to by  
>       buf.    
>
```
vagrant@front:~$ sudo cat /proc/version  
Linux version 5.4.0-58-generic (buildd@lcy01-amd64-004) (gcc version 9.3.0 (Ubuntu 9.3.0-17ubuntu1~20.04)) #64-Ubuntu SMP Wed Dec 9 08:16:25 UTC 2020  
vagrant@front:~$   
```

### Задание 7
Чем отличается последовательность команд через `;` и через `&&` в bash? Например:
```bash  
root@netology1:~# test -d /tmp/some_dir; echo Hi  
Hi  
root@netology1:~# test -d /tmp/some_dir && echo Hi  
root@netology1:~#  
```
Есть ли смысл использовать в bash `&&`, если применить `set -e`?
- Есть, так как при даной конструкции оболочка не видит ненулевого статуса и не завершает работу.

### Задание 8
Из каких опций состоит режим bash `set -euxo pipefail` и почему его хорошо было бы использовать в сценариях?
> -e указывает оболочке выйти, если команда дает ненулевой статус выхода.  
> -u обрабатывает неустановленные или неопределенные переменные, за исключением специальных параметров, таких как подстановочные знаки (*) или «@», как ошибки во время раскрытия параметра  
> -x печатает аргументы команды во время выполнения  
> - -o pipefail если какая то команда в скрипте завершится кодом ошибки то этот код будет присвоен всему скрипту, иначе скрипт получает код последней выполненной команды  
- Вероятно, хорошая практика для использования ее в сценариях - мы не подвесим сервер неправильно созданным скриптом.

### Задание 9
Используя `-o stat` для `ps`, определите, какой наиболее часто встречающийся статус у процессов в системе. В `man ps` ознакомьтесь (`/PROCESS STATE CODES`) что значат дополнительные к основной заглавной буквы статуса процессов. Его можно не учитывать при расчете (считать S, Ss или Ssl равнозначными).
```
vagrant@front:~$ ps -o stat
STAT  
Ss  
R+  
```  
S - Процесс ожидает выполнения  
R -Процесс выполняется в данный момент  
R+  Процесс выполняется в данный момент не в фоновом режиме  
 
