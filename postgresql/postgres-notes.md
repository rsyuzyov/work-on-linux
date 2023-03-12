**Где бинари и конфиги**  
Бинари pg9: `/usr/lib/postgresql/9.6/bin`  
Бинари pg15: `/opt/pgpro/1c-15/bin`  
postgresql.conf: `SHOW config_file;`  
для pg15 это `/var/lib/pgpro/1c-15/data/postgresql.conf`  

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
https://postgrespro.ru/docs/postgrespro/10/config-one-c  
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
track_activity_query_size = 4096

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

