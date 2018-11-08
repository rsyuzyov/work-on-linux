Установка: поверх debian | чистая  
Создание хранилищ: lvs | zfs  
Подключение хранилищ: Directory | lvm, lvm-thin | zfs  
Импорт ВМ  
Бэкапы, репликация  


### Заметки
Где лежат конфиги ВМ: /etc/pve/qemu-server/10*.conf  
Компрессия qcow: `qemu-img convert -O qcow2 -c source-disk.qcow2 dest-disl.qcow2`  
Импорт дисков: `qm importdisk vm-id disk.vhd storagename --format=raw|qcow`  
Для zfs единственный доступный формат - raw, сжатие и все остальные плюшки zfs делает сама

Пересоздание кластера pve:  
```
rm -rf /etc/pve
mkdir /etc/pve
systemctl start pve-cluster
```

Пересоздание lxcfs:  
```
rm -rf /var/lib/lxcfs/*
systemctl start lxcfs
```
