# Развернуть mysql и подкинуть нужную конфигурацию через ansible

Для разворачивания mysql, использовать плейбук *deb_mysql.yml*

# Посмотреть файл my.cnf, как его тюнить.

Посмотрел.
Пример конфигурации для машинки, с 2Гб оперативной памяти.
```
[client]
port                        = 3306
socket                      = /var/run/mysqld/mysqld.sock

[mysqld_safe]
socket                      = /var/run/mysqld/mysqld.sock
nice                        = 0

[mysqld]
user                        = mysql
pid-file                    = /var/run/mysqld/mysqld.pid
socket                      = /var/run/mysqld/mysqld.sock
port                        = 3306
basedir                     = /usr
datadir                     = /var/lib/mysql
tmpdir                      = /tmp
language                    = /usr/share/mysql/english
old_passwords               = 0
bind-address                = 127.0.0.1

skip-external-locking

max_allowed_packet          = 16M
key_buffer_size             = 16M
innodb_buffer_pool_size     = 1024M
innodb_file_per_table       = 1
innodb_flush_method         = O_DIRECT
innodb_flush_log_at_trx_commit  = 0

max_connections             = 136

		query_cache_size            = 0

slow_query_log              = /var/log/mysql/mysql-slow.log
long_query_time             = 1

expire_logs_days            = 10
max_binlog_size             = 100M

[mysqldump]
quick
quote-names
max_allowed_packet          = 16M
```

# Дополнить таблицу с корзиной в базе данных shop, с которой работали на уроке


# Посчитать суммарную корзину покупок для каждого из пользователей и отсортировать по порядку


# *Развернуть через ansible mysql master и слейву с репликацией(1 мастер и 2 слейва) В ansible делаем все через роли, можно посмотреть готовую роли в интернете.