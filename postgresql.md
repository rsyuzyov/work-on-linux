Подготовка в случае linux: используем [ранее подготовленный образ](install-debian.md), добавляем диск; после установки переносим main.  
В случае windows просто рядом ставим на систему с ms sql, выделив память и проц, либо готовим ВМ, как для ms sql.  

**Установка**  
Ставим [отсюда (1С)](https://releases.1c.ru/project/AddCompPostgre), а лучше [отсюда (pgpro)](https://postgrespro.ru/products/archive/1c). 
Я предпочитаю бесплатные сборки для 1C от невероятно крутых ребят из Postgres Professional, поэтому в тексте некоторые несущественные нюансы будут свойственны именно этим сборкам.  

ВНИМАНИЕ! Каталоги исполняемых и конфигурационных файлов по-умолчанию для PG9 и PG10 отличаются:  
###### 
| Данные | 9.6 | 10 и выше |
| --- | --- | --- |
| Конфиги | /etc/postgresql/9.6/main/  |   |
| Бинари | ...  | /opt |
| Базы | /var | /opt |

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
https://github.com/NikolayS/postgres_dba  
http://powa.readthedocs.io/en/latest/  
https://pgtune.leopard.in.ua/#/  

