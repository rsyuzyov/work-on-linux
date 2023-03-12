**Подотовка**  
Развернуть контейнер с debian  
Установить русскую локаль:  
```
sudo dpkg-reconfigure locales
```
Находим в списке локаль ru_RU.UTF-8, помечаем, затем выбираем в качестве основной и завершаем настройку.  


**Установка**  
Есть [вариант дистрибутива от 1С](https://releases.1c.ru/project/AddCompPostgre), но я им не пользуюсь.  
Идем за дистрибутивом от Postgres Professional [сюда](http://1c.postgres.ru/), заполняем почту и получаем на почту инструкцию.  
Пример для 15 версии:  
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
Прежде всего открыть доступ снаружи: в /var/lib/pgpro/1c-15/data/pg_hba.conf заменить 127.0.0.1/32 на 0.0.0.0/0  

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
