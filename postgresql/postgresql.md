**Подотовка**  
Развернуть контейнер с debian  
Установить русскую локаль:  
```
sudo dpkg-reconfigure locales
```
Находим в списке локаль ru_RU.UTF-8, помечаем, затем выбираем в качестве основной и завершаем настройку.  


**Установка**  
Есть [вариант дистрибутива от 1С](https://releases.1c.ru/project/AddCompPostgre), но я им не пользуюсь.  
Берем отсюда [отсюда (pgpro)](http://1c.postgres.ru/): заполняем почту и получаем на почту инструкцию.  
Пример ддля 15 версии:
```
Вы получили это письмо, поскольку запрашивали инструкции по установке postgreSQL для 1с на сайте 1c.postgres.ru/.

Используйте инструкции для установки postgreSQL для 1с. Обратите внимание, что команды должны выполняться от пользователя с правами суперпользователя.

wget https://repo.postgrespro.ru/1c-15/keys/pgpro-repo-add.sh
sh pgpro-repo-add.sh
Если наш продукт единственный Postgres на вашей машине и вы хотите
сразу получить готовую к употреблению базу:


// Этот вариант нам не подходит, так как непоянтно, какие параметры при создании кластера будут установлены, а нам нужны правильные locale и encodind, да и checksums включить полезно
apt-get install postgrespro-1c-15 

Если у вас уже установлен другой Postgres и вы хотите чтобы он
продолжал работать параллельно (в том числе и для апгрейда с более
старой major-версии):


apt-get install postgrespro-1c-15-contrib
// Эту команду мы дополним нужными параметрами
/opt/pgpro/1c-15/bin/pg-setup initdb
/opt/pgpro/1c-15/bin/pg-setup service enable
/opt/pgpro/1c-15/bin/pg-setup service start
```

Итоговый вариант:  
```
wget https://repo.postgrespro.ru/1c-15/keys/pgpro-repo-add.sh
sh pgpro-repo-add.sh
apt-get install postgrespro-1c-15-contrib
/opt/pgpro/1c-15/bin/pg-setup initdb --encoding=UTF8 --locale=ru_RU.UTF-8 --data-checksums
/opt/pgpro/1c-15/bin/pg-setup service enable
/opt/pgpro/1c-15/bin/pg-setup service start
```

**Настройка**  
Прежде всего открыть доступ снаружи: /var/lib/pgpro/1c-10/data/pg_hba.conf: заменить 127.0.0.1/32 на 0.0.0.0/0  

Далее меняем пароль для пользователя postgres:  
```
su postgres
/opt/pgpro/1c-15/bin/psql
\password
\q
exit
```

Для удобства обращения к утилитам можно прописать каталог в PATH:
В ~/.bashrc добавить:
```
export PATH="$PATH:/opt/pgpro/1c-15/bin"
```

Затем необходимо задать базовые параметры, зависящие от типа дисков, количества памяти и ядер и т.п.  
На сайте https://pgtune.leopard.in.ua/ указываем параметры нашего окружения, нажимаем GENERATE.  
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
systemctl restart postgrespro-1c-15
```

По параметру "wal_sync_method":  
Запускаем /usr/lib/postgresql/{PG_VER}/bin/pg_test_fsync и смотрим на результаты: метод с наибольшими ops/sec будет оптимальным.
Для windows это обычно open_datasync, для linux - fdsatasync.  

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
