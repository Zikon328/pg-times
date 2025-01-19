### Исходные данные

```
- Astra Linux 1.8.1.12     (not free - test)
- PostgresPro-1C-16        (free)  
- pg_probackup-16  2.5.15  (free)
```

###  Установка для  версии  Astra Linux  1.8

- Регистрируем репозиторий

```
sudo -- sh -c "echo 'deb https://repo.postgrespro.ru/pg_probackup/deb bookworm main-bookworm' > /etc/apt/sources.list.d/pg_probackup.list"
Загружаем ключ GPG, преобразовываем и записываем в реестр   (  wget, gpg   -  должны быть установлены )

wget --progress=bar:force https://repo.postgrespro.ru/pg_probackup/keys/GPG-KEY-PG-PROBACKUP  -P /tmp/
sudo -- sh -c "gpg --dearmor < /tmp/GPG-KEY-PG-PROBACKUP > /etc/apt/trusted.gpg.d/pg_probackup.gpg"
sudo apt update
```

- Установка нужной версии программы ( для каждой версии PostgreSQL - своя программ pg_probackup )

```
sudo apt install pg-probackup-16
( при установке пакета первый "дефис" -  но при запуске программы  pg_probackup-16   !!!! )
( программа сразу установилась в каталог /usr/bin )
```

### Настройка - Создание пользователя админа бэкапов

- создадим пользователя  probackup  и выдадим права  в базе postgres ( для PG15+ )

```
sudo su - postgres
psql

BEGIN;
CREATE ROLE backup_admin WITH LOGIN REPLICATION password 'AbraBraa';
GRANT USAGE ON SCHEMA pg_catalog TO backup_admin;
GRANT EXECUTE ON FUNCTION pg_catalog.current_setting(text) TO backup_admin;
GRANT EXECUTE ON FUNCTION pg_catalog.set_config(text, text, boolean) TO backup_admin;
GRANT EXECUTE ON FUNCTION pg_catalog.pg_is_in_recovery() TO backup_admin;
GRANT EXECUTE ON FUNCTION pg_catalog.pg_backup_start(text, boolean) TO backup_admin;
GRANT EXECUTE ON FUNCTION pg_catalog.pg_backup_stop(boolean) TO backup_admin;
GRANT EXECUTE ON FUNCTION pg_catalog.pg_create_restore_point(text) TO backup_admin;
GRANT EXECUTE ON FUNCTION pg_catalog.pg_switch_wal() TO backup_admin;
GRANT EXECUTE ON FUNCTION pg_catalog.pg_last_wal_replay_lsn() TO backup_admin;
GRANT EXECUTE ON FUNCTION pg_catalog.txid_current() TO backup_admin;
GRANT EXECUTE ON FUNCTION pg_catalog.txid_current_snapshot() TO backup_admin;
GRANT EXECUTE ON FUNCTION pg_catalog.txid_snapshot_xmax(txid_snapshot) TO backup_admin;
GRANT EXECUTE ON FUNCTION pg_catalog.pg_control_checkpoint() TO backup_admin;
COMMIT;
```

- добавляем в файл ~postgres/.pgpass (права  обязательно  0600)  

```
echo "*:*:*:backup_admin:AbraBraa" >> ~/.pgpass
chmod 600 ~/.pgpass
```

- добавляем в файл pg_hba.conf - следующие строки - желательно в начало правил

```
local   all             backup_admin                      md5
local   replication     backup_admin                      md5
```

- если предусматривается удаленный запуск pg_probackup - ещё добавляем ( для текущей сети )

```
host    all             backup_admin     samenet          md5
host    replication     backup_admin     samenet          md5
```

- перезагрузить конфиг незабываем

```
psql
select pg_reload_config();
```

### Настройка программы pg_probackup

- создаём каталог для бэкапов ( или монтируем сетевой ресурс ) и назначаем переменную BACKUP_PATH

```
mkdir /backups/test
echo "export BACKUP_PATH=/backups/test" >> ~/.profile
source ~/.profile
```


