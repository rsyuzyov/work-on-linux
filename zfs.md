Postgres, в отличие от MS SQL, не умеет сжатие данных.  
Зато сжатие отлично умеет zfs, файловая система и по совместительству менеджер хранилищ.
Сравнение zfs с другими фаловыми системами смотрим [здесь](https://ru.wikipedia.org/wiki/%D0%A1%D1%80%D0%B0%D0%B2%D0%BD%D0%B5%D0%BD%D0%B8%D0%B5_%D1%84%D0%B0%D0%B9%D0%BB%D0%BE%D0%B2%D1%8B%D1%85_%D1%81%D0%B8%D1%81%D1%82%D0%B5%D0%BC).  
Важно! Неоптимальная конфигурация пула zfs может снизить быстродействие на порядок!  

## Общие принципы

## Установка и создание пула
https://github.com/zfsonlinux/zfs/wiki/Debian
```
sudo apt install linux-headers-$(uname -r)
sudo apt-get install zfs-dkms zfsutils-linux
```

Создание пула для postgresql:  
```
sudo zpool create pgpool /dev/sdX
sudo zfs create pgpool/pgdata -o mountpoint=/pgdata
sudo zfs set ashift=12 pgpool/pgdata
sudo zfs set recordsize=8k pgpool/pgdata
sudo zfs set atime=off pgpool/pgdata
sudo zfs set compression=lz4 pgpool/pgdata
sudo zfs primarycache=metadata pgpool/pgdata
```

```
sudo zfs get atime,compression,primarycache,recordsize pgpool/pgdata
```
http://open-zfs.org/wiki/Performance_tuning  

## Установка debian на zfs
