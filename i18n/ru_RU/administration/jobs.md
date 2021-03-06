# Задания

Задания предназначены для выполнения задач в фоновом режиме. Они обрабатывают такие операции, как отправка уведомлений, массовая рассылка, синхронизация, очистка и т.д.

## Планировщик заданий

Планировщик заданий предназначен для выполнения повторяющихся заданий. Планирование для конкретного задания может быть настроено администратором с использованием записи crontab.

```
* * * * *
| | | | |
| | | | |
| | | | +---- Day of the Week   (range: 1-7, 1 standing for Monday)
| | | +------ Month of the Year (range: 1-12)
| | +-------- Day of the Month  (range: 1-31)
| +---------- Hour              (range: 0-23)
```

Если вы хотите, чтобы работа выполнялась как можно чаще, вам нужно установить расписание на `* * * * *`.

## Настройка

Есть два способа обработки заданий: crontab или daemon.

### Cron

Cron прост в настройке. Он поддерживается большинством хостинг-провайдеров. 

Смотрите, как настроить cron [здесь](server-configuration.md#setup-a-crontab).

В системах Unix cron должен запускаться не чаще, чем раз в минуту. Это ограничение можно обойти с помощью следующего трюка.

Добавьте несколько строк в crontab с задержками в секундах:

```
* * * * * /usr/bin/php -f /var/www/html/espocrm/cron.php > /dev/null 2>&1
* * * * * sleep 15; /usr/bin/php -f /var/www/html/espocrm/cron.php > /dev/null 2>&1
* * * * * sleep 30; /usr/bin/php -f /var/www/html/espocrm/cron.php > /dev/null 2>&1
* * * * * sleep 45; /usr/bin/php -f /var/www/html/espocrm/cron.php > /dev/null 2>&1
```

Обратите внимание, что команда, которая запускает cron, отличается в зависимости от среды сервера.

### Daemon

Команда для запуска daemon с помощью nohup:

```
nohup php /path/to/espocrm/daemon.php &
```

Конфигурация для использования systemd:

```
[Unit]
Description=EspoCRM Daemon Service
Requires=mysql.service
After=mysql.service

[Service]
Type=simple
Restart=always
RestartSec=5
StartLimitInterval=0
User=www-data
ExecStart=/usr/bin/php /path/to/espocrm/daemon.php
StandardError=/path/to/espocrm/data/logs/daemon.log

[Install]
WantedBy=default.target
```

Требуются расширения pcntl и posix, php 7.1 или новее. Windows не поддерживается.

Рекомендуется включить обработку заданий в параллельных процессах: Администрирование > Планировщик заданий > Задания (в правом верхнем углу) > Настройки (в верхнем правом углу) > Задания, выполняемые параллельно.

## Параллельное выполнение заданий

По умолчанию задания выполняются одно за другим, что может вызвать ситуации, когда одно задание блокирует выполнение следующего задания на некоторое время (обычно это не более одной минуты). Чтобы избежать этого, можно запускать параллельное виполнение заданий. Параметр доступен в разделе Администрирование > Планировщик заданий > Задания в правом верхнем углу > Настройки в верхнем правом углу.

Доступно начиная с версии 5.5.0.

Требуются расширения pcntl и posix, php 7.1 или новее. Некоторые конфигурации серверов могут ограничивать возможность запуска дочерних процессов.

Windows не поддерживается.

## Смотрите также

[Creating custom scheduled job](../development/scheduled-job.md)
