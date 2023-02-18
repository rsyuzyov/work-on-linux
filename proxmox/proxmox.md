Установка: поверх debian | чистая  
Создание хранилищ: lvm | zfs  
Подключение хранилищ: Directory | lvm, lvm-thin | zfs  
Импорт ВМ  
Бэкапы, репликация  

### Заметки
Если при установке системы все валится и виснет с ошибками, связанными с usb - отключить в биосе usb 3.0 контроллер - материнка скорее всего "переходного периода"  
Где лежат конфиги ВМ: /etc/pve/qemu-server/10*.conf  
Где лежат конфиги LXC: /etc/pce/lxc/10*.conf  
Где лежат профили apparmor: /etc/apparmor.d/lxc/  
Разрешение mount прямо в контейнерах: в профиль lxc-container-default (файл lxc-default в конец перед закрывающей скобкой добавить:  
```
mount fstype=*,
mount option=(rw),
```
В случае проблем смотрим в dmesg сообщения apparmor (сообщения типа DENIED, смотрим тип фс и опции монтирования) и добавлем разрешения: 
```
dmesg | grep apparmor | tail
```
Для поднятия nfs-сервера в контейнере придется ставить его и на хост, чтобы поиметь /proc/nfsd. Также нужно разрешение mount ftype=nfsd.  

Компрессия qcow: `qemu-img convert -O qcow2 -c source-disk.qcow2 dest-disl.qcow2`  
Импорт дисков: `qm importdisk vm-id disk.vhd storagename --format=raw|qcow`  
Для zfs единственный доступный формат - raw, сжатие и все остальные плюшки zfs делает сама.  

Репликация выполняется на уровне хранилищ и доступна только на zfs, так как использует снимки и инкременты.  

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

Если случилось "cluster not ready - no quorum?":  
Если узлов больше одного и `pvecm status` говорит о проблемах с узлами, то:  
```
pve expected 1
```
иначе если узел один или 'pvecm status' говорит, что кластер не запущен, то смотрим ошибки:  
```
systemctl status pve*
journalctl -b
```
и действуем по обстановке. Например у меня после некорректной правки fstab ос грузилась в emergency mode... 
