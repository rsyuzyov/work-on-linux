**Внимание! Данная тема регулярно служит источником холиваров и граблей! Рекомендуется читать до конца перед применением!**  

## Зачем вот это вот все  
Postgres (ванильный), в отличие от MS SQL, не умеет сжатие данных.  
Зато сжатие отлично умеет zfs, файловая система и по совместительству менеджер хранилищ.  
TODO: COW - кратко плюсы и минусы  
Аналоги: btrfs, почему все же не она.  
Большое сравнение файловых систем смотрим [здесь](https://ru.wikipedia.org/wiki/%D0%A1%D1%80%D0%B0%D0%B2%D0%BD%D0%B5%D0%BD%D0%B8%D0%B5_%D1%84%D0%B0%D0%B9%D0%BB%D0%BE%D0%B2%D1%8B%D1%85_%D1%81%D0%B8%D1%81%D1%82%D0%B5%D0%BC).  

## Что не так с zfs
Как считают многие linux-разработчики и ментейнеры дистрибутивов, "проект мог бы быть прекрасен и мог бы претендовать на роль лучшей ФС будущего", но имеет несовместимую с ядром linux лицензиюю... В общем, лицензионные перипетии ужасно изговняли всю эту историю, и это очень-очень печально (за подробностями - в google).
Несмотря на это, debian, ubuntu, arch, suse и другие дистрибутивы не только поддерживают zfs, но и имеют воможность установки системы на zfs.
Если хочется zfs прям из коробки, например для создания обособленного хранилища, имеет смысл посмотреть в сторону freebsd. Небольшой курьез - в виду очень бодрого развития проекта zfs-on-linux freebsd [планирует пересесть на него](https://lists.freebsd.org/pipermail/freebsd-current/2018-December/072422.html) (upd: пересел)   

## О производительности
**Важно!** Неоптимальная конфигурация пула zfs может снизить быстродействие на порядок!  
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
Делаем замеры:
```
cd ~/test
dd if=/dev/zero of=./largefile bs=1M count=1000 oflag=direct
dd if=/dev/zero of=./largefile bs=10K count=100000  oflag=direct
```
TODO: такой замер оказался ни о чем, сделать [отсюда](https://habr.com/post/154235/)  
Результаты записываем и после создания тома zfs тесты повторяем. Практически все параметры томов zfs можно менять на лету: очень рекомендую попробовать поставить recordsize равным 4, 8, 16, 32, 64, 128k и сравнить результаты замеров. Так, при блоке 128k скорость записи примерно уравнивается с показателями ext4. Задрежки, конечно, при таких условиях будут несколько больше...  
Также можно сравнить производительность без сжатия, ну и сдругими параметрами поиграть.  
Кратко общие рекомендации в интернетах: если надо писать большие файлы, либо писать много потоком - ставим 128k, много мелких файлов или сильно непоследовательно - в районе 4-16k. Оптимальное значение подбираем опытным путем.  
Рекомендации в интернетах для postgresql: имеет смысл ставить размер записи равным (или большим) размеру блока, записываемого СУБД, в случае postgresql это 8k.  

Рекомендации от сердца, выстраданные в результате долгих экспериментов и последующих наблюдений в проде:  
- Ставить 128k и не париться. Оно так по-умолчанию и стоит, к слову ;)  
- Сжатие lz4 обязательно, ибо по факту одни плюсы в выхлопе, минусов ни одного найти не удалось  
- Использовать zfs на гипервизоре для размещения виртуалок, в виртуалках использовать ext4  
- Избегать "вложенности" в случае виртуальных машин, когда zfs на гипервизоре и zfs в виртуалке  
- Для БД идеальной получилась схема с размещением на гипервизоре proxmox в lxc-контейнере, контейнер размещен на zfs - быстродействие сопоставимо с работой на хосте в пределах погрешностей  

**sync**: Про sync настоятельно рекомендую все изучить и взвесить самостоятельно. Если кратко, то в случае внезапно краха системы ФС останется абсолютно живой и здоровой (хвала COW), но будет потеряно до 30 секунд работы (как правило 2-5, получено путем примерно 100 экспериментов с жестким отключением питания на обычном ПК). В случае БД теоретически можно получить несогласованные записи (например документ есть, а движений нет), но на практике еще я лично не сталкивался. Лично я для себя принял решение ставить sync=disabled.

Ну и напоследок про бэкапы... Ну их ведь все и всегда регулярно делают и проверяют, да? Мы вот еще и реплики всех виртуалок и контейнеров держим, для особо важных данных - еще и не по одной, благо дисковое пространство сегодня дешево...

## Установка и создание пула
Установка zfs:  
```
apt update
apt install -y linux-headers-$(uname -r)
ln -s /bin/rm /usr/bin/rm
apt install -y zfs-dkms zfsutils-linux
modprobe zfs
systemctl start zfs*
```

Создание пула для postgresql:  
```
sudo zpool create -o ashift=12 pgdata /dev/{sdx} -f
```
"/dev/{sdx}" - здесь может быть диск или раздел диска (посмотреть, что есть: ls /dev/sd*), или просто каталог в случае тестов, например ~/zfstst.  
Если пул создется на hdd, и есть немного места на ssd, можно на ssd создать раздел и включить его в пул в качестве кэша:
```
zpool create -o ashift=12 pgdata /dev/{hdd} cache /dev/{ssd} -f
```
Устанавливаем и сразу проверяем параметры:
```
zfs set recordsize=128k pgdata
zfs set atime=off pgdata
zfs set compression=lz4 pgdata
zfs set sync=disabled pgdata
zfs set primarycache=all pgdata
zfs get atime,compression,primarycache,recordsize,sync,primarycache pgdata
```

## Ограничение потребления памяти
Создаем /etc/modprobe.d/zfs.conf, в него записываем:
```
options zfs zfs_arc_max=4294967296
```
Отключение "умного" кэширования (только при необходимости, например на хосте proxmox):  
```
options zfs zfs_prefetch_disable=1

```
Затем обновляем initramfs:
```
update-initramfs -u -k all
```

Чтобы не ждать перезагрузку, можно менять параметры на лету. Например:
```
echo 4294967296 > /sys/module/zfs/parameters/zfs_arc_max
cat /sys/module/zfs/parameters/zfs_arc_max
```

## Заметки
[Параметры](https://openzfs.github.io/openzfs-docs/Performance%20and%20Tuning/Module%20Parameters.html): `cat /sys/module/zfs/parameters/PARAMETER`  
Статистика ARC: `cat /proc/spl/kstat/zfs/arcstats`  
iostat: `zpool iostat -v`

## Создание тонких копий
Предположим нужна копия сервера БД для разработчика.  
Создаем новый контейнер с диском минимального размера, затем удаляем диск и вместо него создаем клон с продуктивного сервера БД.  
Чтобы не размещать копии на продуктивном сервере, имеет смысл настроить реплику датасета на сервер разработки и делать клоны с нее.  
Предположим, что продуктивный сервер БД размещен в контейнере (или виртуалке) с id 103, нужно сделать клон в контейнер с id 108:  
```
zfs destroy zpool1/subvol-108-disk-0 -r
zfs snapshot zpool1/subvol-103-disk-0@forclone
zfs clone zpool1/subvol-103-disk-0@forclone zpool1/subvol-108-disk-0
```

## Ссылки
https://openzfs.github.io/openzfs-docs/Getting%20Started/Debian/index.html  
https://openzfs.github.io/openzfs-docs/Performance%20and%20Tuning/Workload%20Tuning.html  
https://openzfs.github.io/openzfs-docs/Performance%20and%20Tuning/Module%20Parameters.html  
