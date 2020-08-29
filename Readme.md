# OTUS ДЗ 8 Работа с ZFS
-----------------------------------------------------------------------
### Домашнее задание

1. Определить алгоритм с наилучшим сжатием
   Зачем:
   Отрабатываем навыки работы с созданием томов и установкой параметров. Находим наилучшее сжатие.
Шаги:
*  определить какие алгоритмы сжатия поддерживает zfs (gzip gzip-N, zle lzjb, lz4)
*  создать 4 файловых системы на каждой применить свой алгоритм сжатия
   Для сжатия использовать либо текстовый файл либо группу файлов: скачать файл “Война и мир” и расположить на файловой системе
wget -O War_and_Peace.txt http://www.gutenberg.org/ebooks/2600.txt.utf-8
либо скачать файл ядра распаковать и расположить на файловой системе 
Результат:
список команд которыми получен результат с их выводами
вывод команды из которой видно какой из алгоритмов лучше


2.  Определить настройки pool’a
    Зачем:
Для переноса дисков между системами используется функция export/import. Отрабатываем навыки работы с файловой системой ZFS
Шаги:
Загрузить архив с файлами локально. 
https://drive.google.com/open?id=1KRBNW33QWqbvbVHa3hLJivOAt60yukkg 
Распаковать. 
С помощью команды zfs import собрать pool ZFS.
Командами zfs определить настройки
размер хранилища
тип pool
значение recordsize
какое сжатие используется
какая контрольная сумма используется 
Результат:
список команд которыми восстановили pool . Желательно с  Output команд.
файл с описанием настроек settings

3.  Найти сообщение от преподавателей 
Зачем:
для бэкапа используются технологии snapshot. Snapshot можно передавать между хостами и восстанавливать с помощью send/receive. Отрабатываем навыки восстановления snapshot и переноса файла.
Шаги:
Скопировать файл из удаленной директории.   https://drive.google.com/file/d/1gH8gCL9y7Nd5Ti3IRmplZPF1XjzxeRAG/view?usp=sharing 
    Файл был получен командой
zfs send otus/storage@task2 > otus_task2.file
Восстановить его локально. zfs receive
Найти зашифрованное сообщение в файле secret_message
Результат:
список шагов которыми восстанавливали 
зашифрованное сообщение


## Ход работ ##

1.  Сразу подготовим вагрант-файл, в конфигурацию включим побольше дисков, кроме того сразу пропишем необходимые команды для автоустановки ZFS в виртуалку.Т.о уже при старте ВМ можем работать с ZFS.
> [vagrant@ZFS ~]$ sudo zpool list  
>  no pools available  

Посмотрим на диски в системе  

``` 
[vagrant@ZFS ~]$ lsblk 
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0   40G  0 disk 
└─sda1   8:1    0   40G  0 part /
sdb      8:16   0    1G  0 disk 
├─sdb1   8:17   0 1014M  0 part 
└─sdb9   8:25   0    8M  0 part 
sdc      8:32   0    1G  0 disk 
├─sdc1   8:33   0 1014M  0 part 
└─sdc9   8:41   0    8M  0 part 
sdd      8:48   0    1G  0 disk 
sde      8:64   0    1G  0 disk 
sdf      8:80   0    1G  0 disk 
sdg      8:96   0    1G  0 disk 
```  
Создадим пул.  
```
[vagrant@ZFS ~]$ sudo zpool create -f mypool1 raidz2 sdb sdc sdd sde
[vagrant@ZFS ~]$ sudo zpool list
NAME      SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT
mypool1  3.75G   296K  3.75G        -         -     0%     0%  1.00x    ONLINE  -
```
Создадим файловые системы с различными степенями сжатия
```
[vagrant@ZFS ~]$ sudo zfs create mypool1/gzip
[vagrant@ZFS ~]$ sudo zfs create mypool1/lzjb
[vagrant@ZFS ~]$ sudo zfs create mypool1/zle
[vagrant@ZFS ~]$ df -h
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        912M     0  912M   0% /dev
tmpfs           919M     0  919M   0% /dev/shm
tmpfs           919M  8.6M  911M   1% /run
tmpfs           919M     0  919M   0% /sys/fs/cgroup
/dev/sda1        40G  3.1G   37G   8% /
tmpfs           184M     0  184M   0% /run/user/1000
tmpfs           184M     0  184M   0% /run/user/0
mypool1         1.8G  128K  1.8G   1% /mypool1
mypool1/lz4     1.8G  128K  1.8G   1% /mypool1/lz4
mypool1/gzip    1.8G  128K  1.8G   1% /mypool1/gzip
mypool1/lzjb    1.8G  128K  1.8G   1% /mypool1/lzjb
mypool1/zle     1.8G  128K  1.8G   1% /mypool1/zle
[vagrant@ZFS ~]$ sudo zfs set compression=lz4 mypool1/lz4
[vagrant@ZFS ~]$ sudo zfs set compression=zle mypool1/zle
[vagrant@ZFS ~]$ sudo zfs set compression=lzjb mypool1/lzjb
[vagrant@ZFS ~]$ sudo zfs set compression=gzip mypool1/gzip
```
Скачаем в mypool1 и во все вложенные папки,с включенной компрессией томик войны и мир размером 3,8 мб.
```
vagrant@ZFS mypool1]$ sudo wget http://tolstoy.ru/upload/iblock/b22/voina-i-mir.docx
--2020-08-29 15:09:15--  http://tolstoy.ru/upload/iblock/b22/voina-i-mir.docx
Resolving tolstoy.ru (tolstoy.ru)... 89.188.109.70
Connecting to tolstoy.ru (tolstoy.ru)|89.188.109.70|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 4021035 (3.8M) [application/vnd.openxmlformats-officedocument.wordprocessingml.document,charset=utf-8]
[vagrant@ZFS mypool1]$ zfs get compressratio
NAME          PROPERTY       VALUE  SOURCE
mypool1       compressratio  1.00x  -
mypool1/gzip  compressratio  1.01x  -
mypool1/lz4   compressratio  1.01x  -
mypool1/lzjb  compressratio  1.00x  -
mypool1/zle   compressratio  1.01x  -
sudo zfs list 
NAME           USED  AVAIL     REFER  MOUNTPOINT
mypool1       19.7M  1.72G     3.91M  /mypool1
mypool1/gzip  3.86M  1.72G     3.86M  /mypool1/gzip
mypool1/lz4   3.86M  1.72G     3.86M  /mypool1/lz4
mypool1/lzjb  3.88M  1.72G     3.88M  /mypool1/lzjb
mypool1/zle   3.87M  1.72G     3.87M  /mypool1/zle
```
На примере этого небольшого файла видно преимущество gzip (а ведь это был еще не самый высокий из 9 уровней) и современного используемого по дефолту lz4. Старый lzjb уступил всем, zle лучше проявляет себя на несжимаемых данных типа фото или видео.На большем кол-ве данных разбег был бы больше

2.  Имортируем пул.  
```
vagrant@ZFS /]$ sudo zpool import -d zfs_task1/ otus
[vagrant@ZFS /]$ sudo zpool status otus
  pool: otus
 state: ONLINE
  scan: none requested
config:

	NAME                  STATE     READ WRITE CKSUM
	otus                  ONLINE       0     0     0
	  mirror-0            ONLINE       0     0     0
	    /zfs_task1/filea  ONLINE       0     0     0
	    /zfs_task1/fileb  ONLINE       0     0     0

errors: No known data errors  

[vagrant@ZFS /]$ sudo zpool list | grep otus
otus      480M  2.09M   478M        -         -     0%     0%  1.00x    ONLINE  -

