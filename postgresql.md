Подготовка в случае linux: используем [ранее подготовленный образ](install-debian.md), добавляем диск; после установки переносим main. Либо просто поднимаем контейнер debian 9 из turnkey.  
В случае windows просто рядом ставим на систему с ms sql, выделив память и проц, либо готовим ВМ, как для ms sql.  

**Установка**  
Ставим [отсюда (1С)](https://releases.1c.ru/project/AddCompPostgre), а лучше [отсюда (pgpro)](https://postgrespro.ru/products/archive/1c). 
Я предпочитаю бесплатные сборки для 1C от невероятно крутых ребят из Postgres Professional, поэтому в тексте некоторые несущественные нюансы будут свойственны именно этим сборкам.  

ВНИМАНИЕ! Каталоги исполняемых и конфигурационных файлов по-умолчанию для PG9 и PG10 отличаются:  
###### 
| Данные | 9.6 | 10 и выше |
| --- | --- | --- |
| Конфиги | /etc/postgresql/9.6/main/  | /var/lib/pgpro/1c-10/data/ |
| Бинари | /usr/lib/postgresql/9.6/bin/  | /opt/pgpro/1c-10/bin/ |
| Базы | /var/ | /var/lib/pgpro/1c-10/data/base/ |

Если ОС без русской локали (например готовые lxc-контейнеры turnkey в proxmox), предварительно нужно ее установить:  
```
sudo dpkg-reconfigure -plow locales
```
Находим в списке локаль ru_RU.UTF-8, помечаем и завершаем настройку.  
Далее при инициализации инстанса необходимо указать encoding и locale, для:
```
/opt/pgpro/1c-10/bin/pg-setup initdb --encoding=UTF8 --locale=ru_RU.UTF-8
```

После установки:  
```
sudo -u postgres psql
\password
\q
```

в ~/.bashrc добавить:
```
export PATH="$PATH:/usr/lib/postgresql/9.6/bin"
```
либо
```
export PATH="$PATH:/opt/pgpro/1c-10/bin"
```

**Настройка**  
Прежде всего необходимо задать базовые параметры, зависящие от типа дисков, количества памяти и ядер и т.п.  
На сайте https://pgtune.leopard.in.ua/ указываем параметры нашего окружения, нажимаем GENERATE.  
Результаты вносим в /etc/postgresql/{PG_VER}/main/postgresql.conf.  
Также выполняем [рекомендации для 1С](https://postgrespro.ru/docs/postgrespro/10/config-one-c)  
Дополнительно находим и изменяем параметры:  
```
synchronous_commit = off
wal_sync_method = fdatasync 
```
Применяем настройки:  
```
sudo -u postgres psql
select pg_reload_conf();
\q
```
либо перезапускаем службу:  
```
systemctl restart postgrespro-1c-10
```

По параметру "wal_sync_method":  
Запускаем /usr/lib/postgresql/{PG_VER}/bin/pg_test_fsync и смотрим на результаты: метод с наибольшими ops/sec будет оптимальным.
Для windows это обычно open_datasync, для linux - fdsatasync.  

**Заметки**  
Бинари pg9: /usr/lib/postgresql/9.6/bin
Бинари pg10: /opt/pgpro/1c-10/bin
postgresql.conf: `SHOW config_file;`  для pg10 это `/var/lib/pgpro/1c-10/data/postgresql.conf`  

**Бэкапы**  
https://postgrespro.ru/docs/postgrespro/9.6/continuous-archiving  
https://pgbackrest.org  
https://bucardo.org  
И даже windows-gui для pg_dump:  
https://postgresql-backup.com/  
http://www.microolap.com/products/database/pagodump/  
http://www.microolap.com/products/database/pagorestore/  

Совсем простой вариант, когда PITR (восстановление на момент времени) не нужно:  
```
sudo apt install autopostgresqlbackup  
```
Конфиг здесь: `/etc/default/autopostgresqlbackup`, бэкапы кладутся сюда `/var/lib/autopostgresqlbackup"`  
Не забываем, что бэкапы не стоит хранить вместе с базами! Как минимум делаем nfs или smb ресурс и симлинк на него.  

**Реплики**  
https://wiki.postgresql.org/wiki/Replication,_Clustering,_and_Connection_Pooling  
https://postgrespro.ru/docs/postgrespro/9.6/different-replication-solutions  
https://postgrespro.ru/docs/postgrespro/9.6/continuous-archiving  

**Мониторинг**  
https://github.com/dalibo/pgbadger  
https://pgmetrics.io/  
https://github.com/okbob/pspg  
https://postgrespro.ru/docs/enterprise/9.6/mamonsu  

**Настройка**  
https://pgtune.leopard.in.ua/  
https://its.1c.ru/db/metod8dev/content/4650/hdoc  
https://its.1c.ru/db/metod8dev/content/5866/hdoc  

Чек-лист по параметрам:  
```
#Параметры, не зависящие от окружения:
max_connections = 1000
min_wal_size = 2GB
max_wal_size = 4GB
full_page_writes = on
wal_compression = on
temp_buffers = 256MB
synchronous_commit = off
commit_delay = 1000
commit_siblings = 5
row_security = off
max_files_per_process = 1000
standard_conforming_strings = off
escape_string_warning = off
max_locks_per_transaction = 256
plantuner.fix_empty_table = 'on'
online_analyze.table_type = 'temporary'
online_analyze.verbose = 'off'

#Параметры, зависящие от диска - на примере ssd:
checkpoint_completion_target = 0.9
effective_io_concurrency = 100
random_page_cost = 1.2

#Параметры, зависящие от оперативной памяти - на примере 8Gb:
effective_cache_size = 6GB
shared_buffers = 2GB
maintenance_work_mem = 512MB
wal_buffers = 8MB
default_statistics_target = 100
work_mem = 128MB

#Параметры, зависящие от колиества ядер - на примере 4 ядер:
max_worker_processes = 4
max_parallel_workers_per_gather = 2
max_parallel_workers = 4

#По результатам выполнения pg_test_fsync:
wal_sync_method = <метод>
```
