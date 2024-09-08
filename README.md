# Настройка мониторинга Zabbix с помощью Docker Compose

Этот репозиторий содержит настройки Docker Compose для развертывания системы мониторинга Zabbix с использованием PostgreSQL в качестве базы данных. В настройке включены следующие сервисы:

- **PostgreSQL**: База данных для Zabbix.
- **Zabbix Server**: Основное ядро системы мониторинга Zabbix.
- **Веб-интерфейс Zabbix**: Веб-консоль для управления и просмотра данных Zabbix.
- **Агент Zabbix**: Агент для мониторинга хостовой системы.

## Предварительные требования

- Установленные Docker и Docker Compose на вашей системе.

## Обзор сервисов

### 1. База данных PostgreSQL (`db`)

- **Образ**: `postgres:15-alpine`
- **Тома**: Настроено сохранение данных в `db-data:/var/lib/postgresql/data`.
- **Сети**: Подключен к `zabbix-network`.
- **Переменные окружения**:
  - `POSTGRES_DB`: Имя базы данных для Zabbix (`zabbix`).
  - `POSTGRES_USER`: Пользователь базы данных (`zabbix`).
  - `POSTGRES_PASSWORD`: Пароль для пользователя базы данных Zabbix (`zabbix_pass`).

### 2. Zabbix Server (`zabbix-server`)

- **Образ**: `zabbix/zabbix-server-pgsql:alpine-6.4.5`
- **Зависимость**: Сервис зависит от `db` (PostgreSQL должен быть запущен первым).
- **Тома**: Настроено сохранение данных в `zbx-data:/var/lib/zabbix`.
- **Порты**: Открыт порт `10051` для связи с агентами Zabbix.
- **Переменные окружения**:
  - `POSTGRES_DB`, `POSTGRES_USER`, `POSTGRES_PASSWORD`: Учетные данные для подключения к базе данных PostgreSQL.
  - `ZBX_ENABLE_SNMP_TRAPS`: Установлено значение `false` (включите это, если нужны ловушки SNMP).

### 3. Веб-интерфейс Zabbix (`zabbix-web`)

- **Образ**: `zabbix/zabbix-web-nginx-pgsql:alpine-6.4.5`
- **Зависимость**: Сервис зависит от `zabbix-server`.
- **Порты**: Открыт порт `8080` для доступа к веб-интерфейсу.
- **Переменные окружения**:
  - `POSTGRES_DB`, `POSTGRES_USER`, `POSTGRES_PASSWORD`: Учетные данные для подключения к базе данных PostgreSQL.
  - `ZBX_SERVER_HOST`: Имя хоста сервера Zabbix (`zabbix-server`).
  - `PHP_TZ`: Часовой пояс для PHP (`Europe/Moscow`).

### 4. Агент Zabbix (`zabbix-agent`)

- **Образ**: `zabbix/zabbix-agent:alpine-6.4.5`
- **Зависимость**: Сервис зависит от `zabbix-server`.
- **Переменные окружения**:
  - `ZBX_SERVER_HOST`: IP-адрес сервера Zabbix.
  - `ZBX_SERVER_PORT`: Номер порта сервера Zabbix (`10051`).

## Как использовать

1. **Клонируйте репозиторий**:
   ``` bash
   git clone https://your-repo-url.git
   cd your-repo-directory
   ```
2. **Измените конфигурацию для агента в сервисе zabbix-agent на фактический ip zabbix-server, ZBX_SERVER_HOST**:
3. **Запустите compose файл**
   ``` bash
   docker-compose up -d
   ```
4. **Доступ к web интерфейсу**
   - Откройте браузер и перейдите по адресу http://localhost:8080.
   - Учетные данные по умолчанию:
	- Логин: Admin
	- Пароль: zabbix
5. **Заметки**
   - Настройте часовой пояс (PHP_TZ) в сервисе zabbix-web в соответствии с вашим местоположением. 
   - Убедитесь, что заменили ZBX_SERVER_HOST в сервисе zabbix-agent на фактический IP-адрес сервера Zabbix.
