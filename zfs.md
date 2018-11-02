**Внимание! Данная тема регулярно служит источником холиваров и граблей! Рекомендуется читать до конца перед применением!**  

Postgres, в отличие от MS SQL, не умеет сжатие данных.  
Зато сжатие отлично умеет zfs, файловая система и по совместительству менеджер хранилищ.
Сравнение zfs с другими фаловыми системами смотрим [здесь](https://ru.wikipedia.org/wiki/%D0%A1%D1%80%D0%B0%D0%B2%D0%BD%D0%B5%D0%BD%D0%B8%D0%B5_%D1%84%D0%B0%D0%B9%D0%BB%D0%BE%D0%B2%D1%8B%D1%85_%D1%81%D0%B8%D1%81%D1%82%D0%B5%D0%BC).  

## Предисловие
Важно! Неоптимальная конфигурация пула zfs может снизить быстродействие на порядок!  
Чтобы понять, насколько велика может быть разница, имеет смысл сделать хотя бы сферические замеры на ext4, а затем повторить их на zfs. Самый простой, хоть и не самый лучший, способ - создать раздел ext4 и выполнить dd с разным размером блока. Предположим у нас есть диск /dev/sdb, на который мы планируем положить zfs.  
Для начала форматируем его в ext4:  
```
sudo fdisk /dev/sdb
>>fdisk:
>>o
>>n
>>w
sudo mkfs.ext4 /dev/sdb1
mkdir ~/test
mount /dev/sdb1 ~/test
```
Делаем замеры (todo: такой замер оказался ни о чем, сделать [отсюда](https://habr.com/post/154235/):
```
cd ~/test
dd if=/dev/zero of=./largefile bs=1M count=1000 oflag=direct
dd if=/dev/zero of=./largefile bs=10K count=100000  oflag=direct
```
Результаты записываем и после создания тома zfs тесты повторяем. Практически все параметры томов zfs можно менять на лету: очень рекомендую попробовать поставить recordsize равным 4, 8, 16, 32, 64, 128k и сравнить результаты замеров. Так, при блоке 128k скорость записи примерно уравнивается с показателями ext4. Задрежки, конечно, при таких условиях будут ощутимо больше...  
Также можно сравнить производительность без сжатия, ну и сдругими параметрами поиграть.  
Общие рекомендации примерно такие: если надо писать большие файлы, либо писать много потоком - ставим 128k, много мелких фалов или сильно непоследовательно - в районе 4-16k. Оптимальное значение подбираем опытным путем.  
Для СУБД особая рекомендация: имеет смысл ставить размер записи равным (или большим) размеру блока, записываемого СУБД, в случае postgresql это 8k.  


## Общие принципы

## Установка и создание пула
Установка zfs:  
```
sudo apt update
sudo apt install -y linux-headers-$(uname -r)
sudo ln -s /bin/rm /usr/bin/rm
sudo apt install -y zfs-dkms zfsutils-linux
sudo modprobe zfs
sudo systemctl start zfs*
```

Создание пула для postgresql:  
```
sudo zpool create -o ashift=12 pgdata /dev/{sdx} -f
```
"/dev/{sdx}" - здесь может быть диск или раздел диска (посмотреть, что есть: ls /dev/sd*), или просто каталог в случае тестов, например ~/zfstst.  
Если пул создется на hdd, и есть немного места на ssd, можно на ssd создать раздел и включить его в пул в качестве кэша:
```
sudo zpool create -o ashift=12 pgdata /dev/{hdd} cache /dev/{ssd} -f
```
Устанавливаем и сразу проверяем параметры:
```
sudo zfs set recordsize=8k pgdata
sudo zfs set atime=off pgdata
sudo zfs set compression=lz4 pgdata
sudo zfs set sync=disabled pgdata
sudo zfs set primarycache=all pgdata
sudo zfs get atime,compression,primarycache,recordsize,sync,primarycache pgdata
```
Далее создаем /etc/modprobe.d/zfs.conf, в него записываем:
```
options zfs zfs_arc_max=4294967296
options zfs_prefetch_disable=1
```
Затем обновляем initramfs:
```
update-initramfs -u -k all
```

## Установка debian на zfs


## Заметки
Статистика ARC `cat /proc/spl/kstat/zfs/arcstats`  
iostat `zpool iostat -v`


## Ссылки
https://github.com/zfsonlinux/zfs/wiki/Debian  
http://open-zfs.org/wiki/Performance_tuning  
http://www.oug.org/files/presentations/zfszilsynchronicity.pdf  
