**Задача 1** 

*запустить контейнер с ubuntu, используя механизм LXC.*
- vboxuser@LINUX-VIRTUAL:~$ sudo su - root
- [sudo] пароль для vboxuser: 
- root@LINUX-VIRTUAL:~# 
- root@LINUX-VIRTUAL:~# lxc-create -n test-container -t ubuntu
- root@LINUX-VIRTUAL:~# lxc-ls -f
- NAME         		STATE   AUTOSTART   GROUPS   IPV4      IPV6 UNPRIVILEGED      
test-container STOPPED 0      	   -      -         -    false     

*ограничить контейнер 256 Мб ОЗУ и проверить, что ограничение работает*

- root@LINUX-VIRTUAL:~# vi /var/lib/lxc/test-container/config 
lxc.cgroup2.memory.max = 256M

*добавить автозапуск контейнеру, перезагрузить ОС и убедиться, что контейнер действительно запустился самостоятельно*
- NAME         		STATE   AUTOSTART   GROUPS   IPV4      IPV6 UNPRIVILEGED      
test-container STOPPED 0      	   -      -         -    false  

- root@LINUX-VIRTUAL:~# vi /var/lib/lxc/test-container/config
lxc.start.auto  =  1
- NAME         		STATE   AUTOSTART   GROUPS   IPV4      IPV6 UNPRIVILEGED      
test-container STOPPED 1      	   -      -         -    false  

**Задача 2** 

*Создать несколько процессов и вручную распределить их по разным контрольным группам, ограничив различные ресурсы как на семинаре*

*получаем root-права*
- vboxuser@LINUX-VIRTUAL:~$ sudo su

*создадим 2 java скрипта с одинаковым бесконечным циклом*
- root@LINUX-VIRTUAL:/home/avc# nano test1.java

class Main{

        public static void main(String[] args){

                int i = 0;

                while (true) {

                        i += 1;

                }

        }

}

- root@LINUX-VIRTUAL:/home/avc# nano test2.java

*Создаем два терминала PowerShell, подключаемся по ssh к Ubuntu: ssh vboxuser@192.168.0.172*

*Поочередно запускаем на них оба файла*
- root@LINUX-VIRTUAL:/home/avc# java test1.java
- root@LINUX-VIRTUAL:/home/avc# java test2.java

*открываем диспетчер процессов командой "htop" в терминале Ubuntu и видим 100% загрузку двух ядер процессора*

*в терминале Ubuntu заходим в каталог* **cgroup** 

*создаем контрольную группу homeWork2.slice*

- root@LINUX-VIRTUAL:/sys/fs/cgroup# mkdir homeWork2.slice

*присвоим группе homeWork2.slice значение cpuset = 0 (все процессы, относящиеся к этой группе будут выполняться только на ядре 0)*


- root@LINUX-VIRTUAL:/sys/fs/cgroup/homeWork2.slice# echo 0 > cpuset.cpus

*размещаем процессы 3806 и 3829, относящиеся к исполнению файлов java в группу homeWork2.slice*
- root@LINUX-VIRTUAL:/sys/fs/cgroup/homeWork2.slice# echo 3806 > cgroup.procs
- root@LINUX-VIRTUAL:/sys/fs/cgroup/homeWork2.slice# echo 3829 > cgroup.procs

*оба процесса загружают ядра 0 поровну (по 50%)*

*ограничиваем значение cpu.max = 50000 из максимального 100000*
- root@LINUX-VIRTUAL:/sys/fs/cgroup/homeWork2.slice# echo "50000 100000" > cpu.max

*и теперь ядро 0 загружено только на 50%*

*создаем две контрольные группы n1 и n2*
- root@LINUX-VIRTUAL:/sys/fs/cgroup# mkdir n1
- root@LINUX-VIRTUAL:/sys/fs/cgroup# mkdir n2

*зададим значение ограничения ресурса контейнеров*
- root@LINUX-VIRTUAL:/sys/fs/cgroup# echo 3 > n1/cpu.weight
- root@LINUX-VIRTUAL:/sys/fs/cgroup# echo 7 > n2/cpu.weight

*зададим cpuset = 0 для каждого контейнера*
- root@LINUX-VIRTUAL:/sys/fs/cgroup# echo 0 > n1/cpuset.cpus
- root@LINUX-VIRTUAL:/sys/fs/cgroup# echo 0 > n2/cpuset.cpus 

*размещаем процесс 3228 в группе n1, а процесс 3305 в группе n2*
- root@LINUX-VIRTUAL:/sys/fs/cgroup# echo 3228 > n1/cgroup.procs
- root@LINUX-VIRTUAL:/sys/fs/cgroup# echo 3305 > n2/cgroup.procs

*оба процесса исполняются ядром 0, распределение ресурсов в отношении 30/70%*