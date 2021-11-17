##  Домашнее задание к занятию "3.2. Работа в терминале, лекция 2" - Денис Поляков

### Задание 1  
Какого типа команда `cd`? Попробуйте объяснить, почему она именно такого типа; опишите ход своих мыслей, если считаете что она могла бы быть другого типа.
```
vagrant@vagrant:~$ type cd  
cd is a shell builtin  
```
Это встроенная команда shell, меняет текущую папку только для оболочки, в которой выполняется.

### Задание 2
Какая альтернатива без pipe команде `grep <some_string> <some_file> | wc -l`? `man grep` поможет в ответе на этот вопрос. Ознакомьтесь с [документом](http://www.smallo.ruhr.de/award.html) о других подобных некорректных вариантах использования pipe.
```
vagrant@vagrant:~$ grep "server {" default.conf | wc -l  
3  
  
vagrant@vagrant:~$ grep "server {" default.conf -c  
3  
```
Замена без использования pipe -команда grep c ключом -с: `grep "something" example.txt -c .` так же покажет кол-во строк с искомым параметром, но не требует использования чего либо  помимо grep.

### Задание 3  
Какой процесс с PID `1` является родителем для всех процессов в вашей виртуальной машине Ubuntu 20.04?
```
vagrant@vagrant:~$ ps -p 1  
    PID TTY          TIME CMD  
      1 ?        00:00:01 systemd  
```
Родителем всех процессов явлется `systemd`

### Задание 4  
Как будет выглядеть команда, которая перенаправит вывод stderr `ls` на другую сессию терминала?
```  
vagrant@vagrant:~$ vagrant@vagrant:~$ who  
vagrant  pts/0        2021-11-16 07:40 (10.0.2.2)  
vagrant  pts/1        2021-11-17 06:42 (10.0.2.2)  
vagrant@vagrant:~$ who am i  
vagrant  pts/0        2021-11-16 07:40 (10.0.2.2)  
vagrant@vagrant:~$ ls *.md 2> /dev/pts/1  
  
Вывод в терминальном окне pts/1:  
ls: cannot access '*.md': No such file or directory  
```
stderr -дескрипитор 2 . Для перенаправления ошибок используем конструкцию `2>куда перенаправляем вывод`. Командой `who` смотрим открытые терминальные сессии. Перенаправление в другое окно терминала ошибок команды ls выглядело бы так `ls  2> /dev/pts/1`

### Задание 5  
Получится ли одновременно передать команде файл на stdin и вывести ее stdout в другой файл? Приведите работающий пример.
```  
vagrant@vagrant:~$ cat 1.txt  
test1  
test2  
vagrant@vagrant:~$ cat < 1.txt > 2.txt  
vagrant@vagrant:~$ cat 2.txt  
test1  
test2  
vagrant@vagrant:~$  
```

### Задание 6  
Получится ли вывести находясь в графическом режиме данные из PTY в какой-либо из эмуляторов TTY? Сможете ли вы наблюдать выводимые данные?
```
vagrant@vagrant:~$echo "Проверка перенаправления" > /dev/tty1   
```
Да. В терминале tty1

### Задание 7  
Выполните команду `bash 5>&1`. К чему она приведет? Что будет, если вы выполните `echo netology > /proc/$$/fd/5`? Почему так происходит?
```  
vagrant@vagrant:~$ bash 5>&1  
vagrant@vagrant:~$ echo netology > /proc/$$/fd/5  
netology  
vagrant@vagrant:~$ ls -lh /proc/$$/fd/5  
lrwx------ 1 vagrant vagrant 64 Nov 16 08:43 /proc/1188/fd/5 -> /dev/pts/0  
```
В случае с  bash мы создаем дискриптор `5` и перенаправлем его на stdout. 
во второй же команде мы перенапрвляем вывод команды  echo на дескрипотр 5 который ранее был перенаправлен на stdout


### Задание 8  
Получится ли в качестве входного потока для pipe использовать только stderr команды, не потеряв при этом отображение stdout на pty? Напоминаем: по умолчанию через pipe передается только stdout команды слева от `|` на stdin команды справа.
Это можно сделать, поменяв стандартные потоки местами через промежуточный новый дескриптор, который вы научились создавать в предыдущем вопросе.
```
vagrant@vagrant:~$ mkdir -v test 5>&1 1>&2 2>&5 | grep "test"  
mkdir: создан каталог «test»  
vagrant@vagrant:~$ mkdir -v test 5>&1 1>&2 2>&5 | grep "test"  
mkdir: невозможно создать каталог «test»: Файл существует  
vagrant@vagrant:~$
```

### Задание 9  
Что выведет команда `cat /proc/$$/environ`? Как еще можно получить аналогичный по содержанию вывод?
- Команда выводит значение переменных окружения, его так же можно посмотреть командой `env`

### Задание 10  
Используя `man`, опишите что доступно по адресам `/proc/<PID>/cmdline`, `/proc/<PID>/exe`.
```  
    /proc/[pid]/cmdline  
              This  read-only file holds the complete command line for the process, unless the process is a zombie.  
              In the latter case, there is nothing in this file: that is, a read on this file will return 0 charac‐  
              ters.   The  command-line  arguments  appear in this file as a set of strings separated by null bytes  
              ('\0'), with a further null byte after the last string.  
```
```  
   /proc/[pid]/exe  
              Under  Linux  2.2 and later, this file is a symbolic link containing the actual pathname of the exe‐  
              cuted command.  This symbolic link can be dereferenced normally; attempting to open it will open the  
              executable.  
```
### Задание 11
Узнайте, какую наиболее старшую версию набора инструкций SSE поддерживает ваш процессор с помощью `/proc/cpuinfo`.
```  
cat /proc/cpuinfo | grep "sse"  
flags           : fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ht syscall nx mmxext fxsr_opt rdtscp lm constant_tsc rep_good nopl nonstop_tsc cpuid extd_apicid tsc_known_freq pni pclmulqdq monitor ssse3 cx16 sse4_1 sse4_2 x2apic popcnt aes xsave avx hypervisor lahf_lm cr8_legacy abm sse4a misalig  
sse 3dnowprefetch ssbd vmmcall arat  
```
Процессор поддерживает sse4.2
```  
SSE4.2 added STTNI (String and Text New Instructions),[10] several new instructions that perform character searches and comparison on two operands of 16 bytes at a time. These were designed (among other things) to speed up the parsing of XML documents.[11] It also added a CRC32 instruction to compute cyclic redundancy checks as used in certain data transfer protocols. These instructions were first implemented in the Nehalem-based Intel Core i7 product line and complete the SSE4 instruction set. Support is indicated via the CPUID.01H:ECX.SSE42[Bit 20] flag.  
```

### Задание 12
При открытии нового окна терминала и `vagrant ssh` создается новая сессия и выделяется pty. Это можно подтвердить командой `tty`, которая упоминалась в лекции 3.2. Однако:

```  
	vagrant@netology1:~$ ssh localhost 'tty'  
	not a tty  
```
Почитайте, почему так происходит, и как изменить поведение.

```    
vagrant@vagrant:~$ ssh localhost -t 'tty'  
vagrant@localhost's password:  
/dev/pts/2  
Connection to localhost closed.  
```
Для выполнения  команды   tty на хосте localhost нужно команде ssh  добавить ключ -t


### Задание 13
Бывает, что есть необходимость переместить запущенный процесс из одной сессии в другую. Попробуйте сделать это, воспользовавшись `reptyr`. Например, так можно перенести в `screen` процесс, который вы запустили по ошибке в обычной SSH-сессии.
1. Запускаю `top` в первой сесии:
```  
vagrant@vagrant:~$ tty  
/dev/pts/1  
vagrant@vagrant:~$ ps aux | grep top  
vagrant     3669  0.2  0.3  11820  3980 pts/0    S+   08:12   0:00 top  
vagrant     3670  0.0  0.0   8900   740 pts/1    S+   08:12   0:00 grep --color=auto top  
vagrant@vagrant:~$  
```  
Устанавливаю `reptyr`, проверяю работу пытаясь вытянуть top в мою консоль.  
```  
sudo apt-get install -y reptyr  
vagrant@vagrant:~$ reptyr 2586  
Unable to attach to pid 2586: Operation not permitted  
The kernel denied permission while attaching. If your uid matches  
the target's, check the value of /proc/sys/kernel/yama/ptrace_scope.  
For more information, see /etc/sysctl.d/10-ptrace.conf  
```
в man reptyr нахожу решение:
```  
# echo 0 > /proc/sys/kernel/yama/ptrace_scope  
```
3. Подключаюсь к другой сессии, стартую там `screen`, и перетягиваю процесс в него командой `reptyr`:
```  
vagrant@vagrant:~$screen  
проваливаюсь в окно screen  
vagrant@vagrant:~$  
vagrant@vagrant:~$ ps aux | grep top  
vagrant     3669  0.0  0.4  11820  4072 pts/1    S+   09:31   0:00 top  
vagrant     3671  0.0  0.0   9032   740 pts/0    S+   09:31   0:00 grep --color=auto top  
vagrant@vagrant:~$ reptyr 3669  
```
Получаю вывод рабочий программы в моем окне screen. Можно выходить из него и отключаться от ssh. Программа останется работать в фоне. При следующем ssh подключении: Смотрю запущенные окна `screen -ls`. Подключаюсь к тому где была запущенна программ: `screen -r 3194`. Вижу работу этой программы.

### Задание 14
`sudo echo string > /root/new_file` не даст выполнить перенаправление под обычным пользователем, так как перенаправлением занимается процесс shell'а, который запущен без `sudo` под вашим пользователем. Для решения данной проблемы можно использовать конструкцию `echo string | sudo tee /root/new_file`. Узнайте что делает команда `tee` и почему в отличие от `sudo echo` команда с `sudo tee` будет работать.

- Команда `tee` записывает вывод любой команды в один или несколько файлов.
В этой конструкции `sudo echo string > /root/new_file` под sudo запускается только первая команда echo string  а перенаправление ее вывода идет от обчного пользователя которому запрещен доступ к рутовой папке. 

- В второй конструкции `echo string | sudo tee /root/new_file` мы под своим пользователем запускаем команду echo, а вот уже перенаправление, вывод команды tee в рутовый раздел, идет под sudo
