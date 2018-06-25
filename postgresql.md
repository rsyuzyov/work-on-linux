Подготовка в случае linux: используем [ранее подготовленный образ](install-debian.md), добавляем диск; после установки переносим main.  
В случае windows просто рядом ставим на систему с ms sql, выделив память и проц, либо готовим ВМ, как для ms sql.  

**Установка**  
Ставим [отсюда](https://releases.1c.ru/project/AddCompPostgre) или [отсюда](https://postgrespro.ru/products/archive/1c).  
После установки:  
```
sudo -u postgres psql
\password
\q
```


**Бэкапы**  
https://postgrespro.ru/docs/postgrespro/9.6/continuous-archiving  
https://pgbackrest.org  
https://bucardo.org  
И даже windows-gui для pg_dump:  
https://postgresql-backup.com/  
http://www.microolap.com/products/database/pagodump/  
http://www.microolap.com/products/database/pagorestore/  

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

