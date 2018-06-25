Установка: поверх debian | чистая  
Создание хранилищ: lvs | zfs  
Подключение хранилищ: Directory | lvm, lvm-thin | zfs  
Импорт ВМ  
Бэкапы, репликация  


### Заметки
Где лежат конфиги ВМ: /etc/pve/qemu-server/10*.conf  
Компрессия qcow: `qemu-img convert -O qcow2 -c source-disk.qcow2 dest-disl.qcow2`  
