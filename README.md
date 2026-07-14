# Apache-Guacamole Server Debian13

**ansible-playbook -i inventory.yml install_guacamole.yml -k -K**

# 🖥️ Ansible Playbook для установки Apache Guacamole на Debian 13

## 📌 Что такое Apache Guacamole?

**Apache Guacamole** — это клиент удаленного доступа с открытым исходным кодом, который работает **без необходимости установки клиентского ПО**. Доступ к удаленным рабочим столам (RDP, VNC, SSH, Telnet) осуществляется **через обычный веб-браузер** с использованием HTML5. Guacamole состоит из двух основных компонентов:

- **guacamole-server** — серверная часть, проксирующая подключения к удаленным системам.
- **guacamole-client** — веб-приложение (WAR-файл), которое запускается в контейнере сервлетов (Tomcat) и предоставляет веб-интерфейс.

Основные возможности:
- Поддержка RDP (с RemoteApp), VNC, SSH, Telnet, Kubernetes.
- Запись сессий, TOTP-аутентификация, ограничения доступа.
- Хранение подключений и пользователей в базе данных (PostgreSQL/MySQL).

---

## 🎯 Основная задача плейбука

Этот Ansible-плейбук **полностью автоматизирует установку и настройку Apache Guacamole 1.6.0** на сервере с Debian 13 (Trixie). Он выполняет:

- Подготовку ОС и установку всех зависимостей (включая `freerdp2`, а не `freerdp3`).
- Сборку `guacamole-server` и `guacamole-client` из исходников.
- Настройку веб-сервера Nginx как фронтенда к Tomcat с SSL.
- Интеграцию с PostgreSQL для хранения пользователей и подключений.
- Установку расширений: запись сессий, TOTP, ограничения доступа.

---

## 📁 Структура плейбука и назначение файлов

| Файл | Описание |
|------|----------|
| **`inventory.yml`** | Инвентарь Ansible: определяет хост `guacamole-server`, задает переменные (версии, пути, учетные данные PostgreSQL). |
| **`ansible.cfg`** | Конфигурация Ansible: включает удобный вывод (`unixy`), профилирование задач и отключает проверку host-ключей. |
| **`install_guacamole.yml`** | Главный плейбук, который последовательно импортирует все остальные плейбуки в нужном порядке. |
| **`01-prepare-os.yml`** | Подготовка ОС: обновление репозиториев (включая `bookworm`), установка пакетов сборки, Tomcat, FreeRDP2, создание пользователя Tomcat, отключение IPv6. |
| **`02-download-sources.yml`** | Скачивание исходных кодов `guacamole-server` и `guacamole-client` с официального сайта Apache и их распаковка. |
| **`03-build-and-install.yml`** | Сборка и установка `guacamole-server` (через `./configure && make`), сборка `guacamole-client` через Maven, конвертация WAR-файла под Jakarta, настройка systemd-сервисов `guacd` и Tomcat. |
| **`04-deploy-webapp.yml`** | Настройка Tomcat на прослушивание только localhost, установка Nginx, генерация самоподписного SSL-сертификата, создание прокси-конфигурации Nginx с перенаправлением 80→443. |
| **`05-setup-sql-base.yml`** | Установка PostgreSQL, создание БД и пользователя, инициализация схемы, загрузка и установка JDBC-расширения и драйвера PostgreSQL, настройка `guacamole.properties`. |
| **`06-plugin-deploy.yml`** | Установка расширений: запись сессий, TOTP, ограничения доступа. Создание каталогов для записей и файлового обмена, настройка прав и добавление параметров в `guacamole.properties`. |

---

## ⚠️ Важные особенности

- **FreeRDP2 используется вместо FreeRDP3** — из-за проблем с RemoteApp в версии 1.6.0.
- **Tomcat 11** — поддержка Jakarta EE (конвертация WAR через `javax2jakarta`).
- **Отключение IPv6** — чтобы избежать проблем с сетевым доступом к Tomcat.
- **Самоподписной SSL-сертификат** — создается автоматически с указанными DNS и IP-альтернативами.
- **Хранение записей сессий** — настраивается отдельная папка с правами для Tomcat и daemon.
- **Использование PostgreSQL** — все данные (пользователи, подключения, настройки) хранятся в БД.

---

## 🔧 Переменные, требующие внимания

Перед запуском обязательно измените или проверьте следующие переменные в `inventory.yml`:

- `ansible_host` — IP-адрес сервера.
- `postgresql_password` — пароль пользователя БД.
- `server_AltNameDNS` — доменное имя для SSL-сертификата.
- `postgresql_jdbc_driver_url` — URL для загрузки JDBC-драйвера PostgreSQL.

---

# 🖥️ Ansible Playbook for Installing Apache Guacamole on Debian 13

## 📌 What is Apache Guacamole?

**Apache Guacamole** is an open-source remote access client that works **without requiring any client software installation**. Access to remote desktops (RDP, VNC, SSH, Telnet) is provided **through a regular web browser** using HTML5. Guacamole consists of two main components:

- **guacamole-server** — the server-side component that proxies connections to remote systems.
- **guacamole-client** — a web application (WAR file) that runs in a servlet container (Tomcat) and provides the web interface.

Key features:
- Support for RDP (including RemoteApp), VNC, SSH, Telnet, Kubernetes.
- Session recording, TOTP authentication, access restrictions.
- Storage of connections and users in a database (PostgreSQL/MySQL).

---

## 🎯 Main Purpose of This Playbook

This Ansible playbook **fully automates the installation and configuration of Apache Guacamole 1.6.0** on a Debian 13 (Trixie) server. It performs:

- OS preparation and installation of all dependencies (including `freerdp2` instead of `freerdp3`).
- Building `guacamole-server` and `guacamole-client` from source.
- Configuring Nginx as a frontend to Tomcat with SSL.
- Integration with PostgreSQL for storing users and connections.
- Installing extensions: session recording, TOTP, and connection restrictions.

---

## 📁 Playbook Structure and File Descriptions

| File | Description |
|------|-------------|
| **`inventory.yml`** | Ansible inventory: defines the `guacamole-server` host and sets variables (versions, paths, PostgreSQL credentials). |
| **`ansible.cfg`** | Ansible configuration: enables user-friendly output (`unixy`), task profiling, and disables host key checking. |
| **`install_guacamole.yml`** | Main playbook that sequentially imports all other playbooks in the correct order. |
| **`01-prepare-os.yml`** | OS preparation: updates repositories (including `bookworm`), installs build packages, Tomcat, FreeRDP2, creates Tomcat user, disables IPv6. |
| **`02-download-sources.yml`** | Downloads the source code of `guacamole-server` and `guacamole-client` from the official Apache website and extracts them. |
| **`03-build-and-install.yml`** | Builds and installs `guacamole-server` (via `./configure && make`), builds `guacamole-client` with Maven, converts the WAR file for Jakarta, configures systemd services for `guacd` and Tomcat. |
| **`04-deploy-webapp.yml`** | Configures Tomcat to listen only on localhost, installs Nginx, generates a self-signed SSL certificate, creates an Nginx proxy configuration with 80→443 redirection. |
| **`05-setup-sql-base.yml`** | Installs PostgreSQL, creates the database and user, initializes the schema, downloads and installs the JDBC extension and PostgreSQL driver, configures `guacamole.properties`. |
| **`06-plugin-deploy.yml`** | Installs extensions: session recording, TOTP, and access restrictions. Creates directories for recordings and file sharing, sets permissions, and adds parameters to `guacamole.properties`. |

---

## ⚠️ Important Notes

- **FreeRDP2 is used instead of FreeRDP3** — due to RemoteApp issues in version 1.6.0.
- **Tomcat 11** — supports Jakarta EE (WAR conversion via `javax2jakarta`).
- **IPv6 is disabled** — to avoid network access issues with Tomcat.
- **Self-signed SSL certificate** — is generated automatically with the specified DNS and IP alternatives.
- **Session recording storage** — a separate directory is configured with appropriate permissions for Tomcat and daemon.
- **PostgreSQL is used** — all data (users, connections, settings) is stored in the database.

---

## 🔧 Variables That Require Attention

Before running, make sure to change or verify the following variables in `inventory.yml`:

- `ansible_host` — server IP address.
- `postgresql_password` — database user password.
- `server_AltNameDNS` — domain name for the SSL certificate.
- `postgresql_jdbc_driver_url` — URL for downloading the PostgreSQL JDBC driver.

---

