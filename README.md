# Docker Lab

## Лабораторная работа по посвящена работе с docker
Работа посвящена изучению технологии работы с контейнерами

## Задание лабораторной работы

Создаем переменные окружения
```bash
$ export GITHUB_USERNAME=<имя_пользователя>
$ export GIST_TOKEN=<сохраненный_токен>
$ alias edit=<nano|vi|vim|subl>
```
Клонируем репозиторий лабораторной работы №6 в данный репозиторий 
```sh
$ git clone https://github.com/${GITHUB_USERNAME}/lab06 projects/lab_docker
$ cd projects/lab_docker
$ git remote remove origin
$ git remote add origin https://github.com/${GITHUB_USERNAME}/lab_docker
```
Установка Docker
```sh
# Debian
$ sudo apt-get update
$ sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
Создание файла `main.py`
```sh
$ cat >> main.py <<EOF
print("Hello, Docker!")
EOF
```
Создание `requirements.txt`
```sh
$ cat >> requirements.txt <<EOF
flask
requests
EOF
```
Создание `Dockerfile`
```sh
$ cat >> Dockerfile <<EOF
FROM python:3.9-slim

WORKDIR /app

RUN apt-get update && apt-get install -y \
    build-essential 

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY . .

CMD ["python", "main.py"]
EOF
```
Сборка и запуск Docker образа
```sh
$ sudo docker build -t lab-docker .
```
<details><summary>Вывод</summary>

```sh
[+] Building 1.2s (11/11) FINISHED                               docker:default
 => [internal] load build definition from Dockerfile                       0.0s
 => => transferring dockerfile: 463B                                       0.0s
 => [internal] load metadata for docker.io/library/python:3.9-slim         0.8s
 => [internal] load .dockerignore                                          0.0s
 => => transferring context: 2B                                            0.0s
 => [stage-1 1/6] FROM docker.io/library/python:3.9-slim@sha256:2d97f6910  0.0s
 => => resolve docker.io/library/python:3.9-slim@sha256:2d97f6910b16bd338  0.0s
 => [internal] load build context                                          0.0s
 => => transferring context: 5.43kB                                        0.0s
 => CACHED [stage-1 2/6] WORKDIR /app                                      0.0s
 => CACHED [stage-1 3/6] RUN apt-get update && apt-get install -y     bui  0.0s
 => CACHED [stage-1 4/6] COPY requirements.txt .                           0.0s
 => CACHED [stage-1 5/6] RUN pip install --no-cache-dir -r requirements.t  0.0s
 => [stage-1 6/6] COPY . .                                                 0.0s
 => exporting to image                                                     0.2s
 => => exporting layers                                                    0.1s
 => => exporting manifest sha256:ece79b870731f56151d66c1e4dac43242e9a8af8  0.0s
 => => exporting config sha256:955e32ebc07cbae901e8448ee7cce80f755dbf8e00  0.0s
 => => exporting attestation manifest sha256:ffceb468535ca5716667f3306552  0.0s
 => => exporting manifest list sha256:183b8155ce11d82c07809c8fd4735d5fde7  0.0s
 => => naming to docker.io/library/lab-docker:latest                       0.0s
 => => unpacking to docker.io/library/lab-docker:latest                    0.0s
```
</details>

```sh
$ sudo docker run --rm -it lab-docker
```
И видим:
```sh
Hello, Docker!
```

### Docker compose

Создание `docker-compose.yml`
```sh
$ cat >> docker-compose.yml <<EOF
version: '3.8'

services:
  app:
    build: . 
    container_name: lab_docker
    depends_on:
      db:
        condition: service_healthy
    environment:
      - DB_HOST=$DB_HOST
      - DB_USER=$DB_USER
      - DB_PASSWORD=$DB_PASSWORD
      - DB_NAME=$DB_NAME

  # Сервис базы данных MySQL
  db:
    image: mysql:8.0
    container_name: mysql_db
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: $DB_ROOT_PASSWORD
      MYSQL_DATABASE: $DB_NAME
      MYSQL_USER: $DB_USER
      MYSQL_PASSWORD: $DB_PASSWORD
    ports:
      - "3306:3306"
    volumes:
      - db_data:/var/lib/mysql
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  db_data:
EOF
```
Запуск через Docker Compose
```sh
$ docker compose up --build
```
<details><summary>Вывод</summary>

```sh
[+] Building 1.4s (13/13) FINISHED                                              
 => [internal] load local bake definitions                                 0.0s
 => => reading from stdin 554B                                             0.0s
 => [internal] load build definition from Dockerfile                       0.0s
 => => transferring dockerfile: 463B                                       0.0s
 => [internal] load metadata for docker.io/library/python:3.9-slim         0.9s
 => [internal] load .dockerignore                                          0.0s
 => => transferring context: 2B                                            0.0s
 => [stage-1 1/6] FROM docker.io/library/python:3.9-slim@sha256:2d97f6910  0.0s
 => => resolve docker.io/library/python:3.9-slim@sha256:2d97f6910b16bd338  0.0s
 => [internal] load build context                                          0.0s
 => => transferring context: 4.76kB                                        0.0s
 => CACHED [stage-1 2/6] WORKDIR /app                                      0.0s
 => CACHED [stage-1 3/6] RUN apt-get update && apt-get install -y     bui  0.0s
 => CACHED [stage-1 4/6] COPY requirements.txt .                           0.0s
 => CACHED [stage-1 5/6] RUN pip install --no-cache-dir -r requirements.t  0.0s
 => [stage-1 6/6] COPY . .                                                 0.1s
 => exporting to image                                                     0.2s
 => => exporting layers                                                    0.1s
 => => exporting manifest sha256:3ae03566bc84b7f461747a91e24ff94fbe53b099  0.0s
 => => exporting config sha256:a623f0d7aeffbb461cc389e3d6377482bf7aab38a0  0.0s
 => => exporting attestation manifest sha256:6892f1d4168a3898bac19098a797  0.0s
 => => exporting manifest list sha256:407307833f1e94752323fd3c9047cfb02e2  0.0s
 => => naming to docker.io/library/lab_docker-app:latest                   0.0s
 => => unpacking to docker.io/library/lab_docker-app:latest                0.0s
 => resolving provenance for metadata file                                 0.0s
[+] up 3/3
 ✔ Image lab_docker-app Built                                               1.5s
 ✔ Container mysql_db   Recreated                                           0.1s
 ✔ Container lab_docker Recreated                                           0.1s
Attaching to lab_docker, mysql_db
Container mysql_db Waiting 
mysql_db  | 2026-04-28 08:29:14+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 8.0.46-1.el9 started.
mysql_db  | 2026-04-28 08:29:15+00:00 [Note] [Entrypoint]: Switching to dedicated user 'mysql'
mysql_db  | 2026-04-28 08:29:15+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 8.0.46-1.el9 started.
mysql_db  | 2026-04-28 08:29:15+00:00 [Note] [Entrypoint]: Initializing database files
mysql_db  | 2026-04-28T08:29:15.213397Z 0 [Warning] [MY-011068] [Server] The syntax '--skip-host-cache' is deprecated and will be removed in a future release. Please use SET GLOBAL host_cache_size=0 instead.
mysql_db  | 2026-04-28T08:29:15.215433Z 0 [System] [MY-013169] [Server] /usr/sbin/mysqld (mysqld 8.0.46) initializing of server in progress as process 81
mysql_db  | 2026-04-28T08:29:15.232856Z 1 [System] [MY-013576] [InnoDB] InnoDB initialization has started.
mysql_db  | 2026-04-28T08:29:15.727294Z 1 [System] [MY-013577] [InnoDB] InnoDB initialization has ended.
mysql_db  | 2026-04-28T08:29:16.901174Z 6 [Warning] [MY-010453] [Server] root@localhost is created with an empty password ! Please consider switching off the --initialize-insecure option.
mysql_db  | 2026-04-28 08:29:20+00:00 [Note] [Entrypoint]: Database files initialized
mysql_db  | 2026-04-28 08:29:20+00:00 [Note] [Entrypoint]: Starting temporary server
mysql_db  | 2026-04-28T08:29:20.585277Z 0 [Warning] [MY-011068] [Server] The syntax '--skip-host-cache' is deprecated and will be removed in a future release. Please use SET GLOBAL host_cache_size=0 instead.
mysql_db  | 2026-04-28T08:29:20.586279Z 0 [System] [MY-010116] [Server] /usr/sbin/mysqld (mysqld 8.0.46)


starting as process 125
mysql_db  | 2026-04-28T08:29:20.606588Z 1 [System] [MY-013576] [InnoDB] InnoDB initialization has started.
mysql_db  | 2026-04-28T08:29:20.855952Z 1 [System] [MY-013577] [InnoDB] InnoDB initialization has ended.
mysql_db  | 2026-04-28T08:29:21.082380Z 0 [Warning] [MY-010068] [Server] CA certificate ca.pem is self signed.
mysql_db  | 2026-04-28T08:29:21.082413Z 0 [System] [MY-013602] [Server] Channel mysql_main configured to support TLS. Encrypted connections are now supported for this channel.
mysql_db  | 2026-04-28T08:29:21.084804Z 0 [Warning] [MY-011810] [Server] Insecure configuration for --pid-file: Location '/var/run/mysqld' in the path is accessible to all OS users. Consider choosing a different directory.
mysql_db  | 2026-04-28T08:29:21.111631Z 0 [System] [MY-011323] [Server] X Plugin ready for connections. Socket: /var/run/mysqld/mysqlx.sock
mysql_db  | 2026-04-28T08:29:21.111796Z 0 [System] [MY-010931] [Server] /usr/sbin/mysqld: ready for connections. Version: '8.0.46'  socket: '/var/run/mysqld/mysqld.sock'  port: 0  MySQL Community Server - GPL.
mysql_db  | 2026-04-28 08:29:21+00:00 [Note] [Entrypoint]: Temporary server started.
mysql_db  | '/var/lib/mysql/mysql.sock' -> '/var/run/mysqld/mysqld.sock'
mysql_db  | Warning: Unable to load '/usr/share/zoneinfo/iso3166.tab' as time zone. Skipping it.
mysql_db  | Warning: Unable to load '/usr/share/zoneinfo/leap-seconds.list' as time zone. Skipping it.
mysql_db  | Warning: Unable to load '/usr/share/zoneinfo/leapseconds' as time zone. Skipping it.
mysql_db  | Warning: Unable to load '/usr/share/zoneinfo/tzdata.zi' as time zone. Skipping it.
mysql_db  | Warning: Unable to load '/usr/share/zoneinfo/zone.tab' as time zone. Skipping it.
mysql_db  | Warning: Unable to load '/usr/share/zoneinfo/zone1970.tab' as time zone. Skipping it.
mysql_db  | 2026-04-28 08:29:22+00:00 [Note] [Entrypoint]: Creating database mydb
mysql_db  | 2026-04-28 08:29:22+00:00 [Note] [Entrypoint]: Creating user myuser
mysql_db  | 2026-04-28 08:29:22+00:00 [Note] [Entrypoint]: Giving user myuser access to schema mydb
mysql_db  | 
mysql_db  | 2026-04-28 08:29:22+00:00 [Note] [Entrypoint]: Stopping temporary server
mysql_db  | 2026-04-28T08:29:22.740996Z 13 [System] [MY-013172] [Server] Received SHUTDOWN from user root. Shutting down mysqld (Version: 8.0.46).
mysql_db  | 2026-04-28T08:29:23.818331Z 0 [System] [MY-010910] [Server] /usr/sbin/mysqld: Shutdown complete (mysqld 8.0.46)  MySQL Community Server - GPL.
mysql_db  | 2026-04-28 08:29:24+00:00 [Note] [Entrypoint]: Temporary server stopped
mysql_db  | 
mysql_db  | 2026-04-28 08:29:24+00:00 [Note] [Entrypoint]: MySQL init process done. Ready for start up.
mysql_db  | 
mysql_db  | 2026-04-28T08:29:24.980858Z 0 [Warning] [MY-011068] [Server] The syntax '--skip-host-cache' is deprecated and will be removed in a future release. Please use SET GLOBAL host_cache_size=0 instead.
mysql_db  | 2026-04-28T08:29:24.981932Z 0 [System] [MY-010116] [Server] /usr/sbin/mysqld (mysqld 8.0.46) starting as process 1
mysql_db  | 2026-04-28T08:29:24.988105Z 1 [System] [MY-013576] [InnoDB] InnoDB initialization has started.
mysql_db  | 2026-04-28T08:29:25.177807Z 1 [System] [MY-013577] [InnoDB] InnoDB initialization has ended.
mysql_db  | 2026-04-28T08:29:25.319060Z 0 [Warning] [MY-010068] [Server] CA certificate ca.pem is self signed.
mysql_db  | 2026-04-28T08:29:25.319093Z 0 [System] [MY-013602] [Server] Channel mysql_main configured to support TLS. Encrypted connections are now supported for this channel.
mysql_db  | 2026-04-28T08:29:25.322008Z 0 [Warning] [MY-011810] [Server] Insecure configuration for --pid-file: Location '/var/run/mysqld' in the path is accessible to all OS users. Consider choosing a different directory.
mysql_db  | 2026-04-28T08:29:25.339515Z 0 [System] [MY-011323] [Server] X Plugin ready for connections. Bind-address: '::' port: 33060, socket: /var/run/mysqld/mysqlx.sock
mysql_db  | 2026-04-28T08:29:25.339865Z 0 [System] [MY-010931] [Server] /usr/sbin/mysqld: ready for connections. Version: '8.0.46'  socket:


'/var/run/mysqld/mysqld.sock'  port: 3306  MySQL Community Server - GPL.
Container mysql_db Healthy 
lab_docker  | Hello, Docker!
lab_docker exited with code 0
```
</details>

## Домашнее задание

В репозитории приведен код web-приложения, которое сохраняет в БД введенную информацию о задаче - ее имя.

## Часть I. Docker

1. Добавьте в код Dockerfile, который позволит запустить web-приложение с исходным кодом в каталоге app/ через docker.

Создание `app/app.py`
```sh
mkdir -p app/templates db
cat > app/app.py <<'EOF'
from flask import Flask, render_template, make_response
from models import ItemModel

app = Flask(__name__)
model = ItemModel()

@app.route('/')
def index():
    items = model.get_all_items()
    response = make_response(render_template('index.html', items=items))
    response.headers['Content-Type'] = 'text/html; charset=utf-8'
    return response

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
EOF
```
Создание `app/models.py`
```sh
cat > app/models.py <<'EOF'
import os
import mysql.connector

class ItemModel:
    def __init__(self):
        self.config = {
            'host': os.getenv('DB_HOST', 'db'),
            'user': os.getenv('DB_USER', 'myuser'),
            'password': os.getenv('DB_PASSWORD', 'mypassword'),
            'database': os.getenv('DB_NAME', 'mydb'),
            'charset': 'utf8mb4',
            'use_unicode': True,
            'collation': 'utf8mb4_unicode_ci'
        }

    def get_all_items(self):
        try:
            conn = mysql.connector.connect(**self.config)
            cursor = conn.cursor(dictionary=True)
            cursor.execute('SELECT name FROM items')
            items = cursor.fetchall()
            cursor.close()
            conn.close()
            return items
        except Exception as e:
            print(f"Error: {e}")
            return []
EOF
```
Создание `app/requirements.txt`
```sh
cat > app/requirements.txt <<'EOF'
flask
mysql-connector-python
EOF
```
Создание `app/templates/index.html`
```sh
cat > app/templates/index.html <<'EOF'
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>MVC App</title>
</head>
<body>
    <h1>Список из Базы Данных</h1>
    <ul>
        {% for item in items %}
            <li>{{ item.name }}</li>
        {% endfor %}
    </ul>
</body>
</html>
EOF
```
Создание `db/init.sql`
```sh
cat > db/init.sql <<'EOF'
SET NAMES utf8mb4;
SET CHARACTER SET utf8mb4;

CREATE TABLE IF NOT EXISTS items (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

INSERT INTO items (name) VALUES ('Пример 1'), ('Пример 2');
EOF
```
Создание `Dockerfile`
```sh
cat > Dockerfile <<'EOF'
FROM python:3.9-slim

WORKDIR /app

RUN apt-get update && apt-get install -y \
    build-essential \
    && rm -rf /var/lib/apt/lists/*

COPY app/requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY app/ .

CMD ["python", "app.py"]
EOF
```
2. Выполните запуск контейнера с этим приложением.

Собираем 
```sh
docker build -t flask-app .
```
<details><summary>Вывод</summary>

```sh
[+] Building 92.5s (11/11) FINISHED                              docker:default
 => [internal] load build definition from Dockerfile                       0.0s
 => => transferring dockerfile: 295B                                       0.0s
 => [internal] load metadata for docker.io/library/python:3.9-slim         0.7s
 => [internal] load .dockerignore                                          0.0s
 => => transferring context: 2B                                            0.0s
 => [1/6] FROM docker.io/library/python:3.9-slim@sha256:2d97f6910b16bd338  0.0s
 => => resolve docker.io/library/python:3.9-slim@sha256:2d97f6910b16bd338  0.0s
 => [internal] load build context                                          0.0s
 => => transferring context: 1.55kB                                        0.0s
 => CACHED [2/6] WORKDIR /app                                              0.0s
 => [3/6] RUN apt-get update && apt-get install -y     build-essential    63.4s
 => [4/6] COPY app/requirements.txt .                                      0.3s 
 => [5/6] RUN pip install --no-cache-dir -r requirements.txt              10.1s 
 => [6/6] COPY app/ .                                                      0.1s 
 => exporting to image                                                    17.6s 
 => => exporting layers                                                   14.1s 
 => => exporting manifest sha256:bb0e7b23bf5fbaf70eb74890e8898136c47da982  0.0s 
 => => exporting config sha256:1ea27708bdb1aa0a13cf3012cc192ff6d95b4c2b71  0.0s 
 => => exporting attestation manifest sha256:dbfc3ede35da9c8d0f45643048fe  0.0s 
 => => exporting manifest list sha256:b4de504e361e96d934daf51f14961428751  0.0s
 => => naming to docker.io/library/flask-app:latest                        0.0s
 => => unpacking to docker.io/library/flask-app:latest
```
</details>

И запускаем
```sh
sudo docker run -d --name flask_container -p 5000:5000 flask-app
```
```sh
cf58204349b65b4e8448a80c5de77566e6a3bb04855bbf2d660d24db3775f349
```
3. Скопируйте из консоли в каталог /home/ контейнера файл README.md.
```sh
echo "# Docker Lab" > README.md
docker cp README.md flask_container:/home/
```
4. Подключитесь к терминалу контейнера с приложением в интерактивном режиме. Проверьте, что скопированный файл находится в нужном каталоге.
```sh
sudo docker exec -it flask_container /bin/bash
```
```sh
sudo docker exec -it flask_container /bin/bash
root@cf58204349b6:/app# 
root@cf58204349b6:/app# ls -la /home/
total 12
drwxr-xr-x 1 root root 4096 Apr 28 08:46 .
drwxr-xr-x 1 root root 4096 Apr 28 08:46 ..
-rw-rw-r-- 1 1000 1000   13 Apr 28 08:44 README.md
root@cf58204349b6:/app# cat /home/README.md
# Docker Lab
root@cf58204349b6:/app# ls -la /app/
total 28
drwxr-xr-x 1 root root 4096 Apr 28 08:40 .
drwxr-xr-x 1 root root 4096 Apr 28 08:46 ..
drwxr-xr-x 2 root root 4096 Apr 28 08:40 __pycache__
-rw-rw-r-- 1 root root  298 Apr 28 08:34 app.py
-rw-rw-r-- 1 root root  731 Apr 28 08:35 models.py
-rw-rw-r-- 1 root root   29 Apr 28 08:35 requirements.txt
drwxrwxr-x 2 root root 4096 Apr 28 08:35 templates
root@cf58204349b6:/app#
```
5. Выйдите из интерактивного режима.
6. Остановите контейнер с приложением.
```sh
sudo docker stop flask_container
```
```sh
flask_container
```

## Часть II. Docker compose
1. Создайте файл docker-compose.yml таким образом, чтобы совместно с описанным в части 1 контейнером работала бы база данных mysql. Файл инициализации БД в каталоге db/init.sql. Также пропишите порт подключения к приложению. Например 5000.
```sh
cat > docker-compose.yml <<'EOF'
services:
  web:
    build: .
    container_name: lab_docker
    ports:
      - "5000:5000"
    depends_on:
      db:
        condition: service_healthy
    environment:
      DB_HOST: db
      DB_USER: myuser
      DB_PASSWORD: mypassword
      DB_NAME: mydb

  db:
    image: mysql:8.0
    container_name: mysql_db
    restart: always
    command: --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: mydb
      MYSQL_USER: myuser
      MYSQL_PASSWORD: mypassword
      MYSQL_ROOT_HOST: '%'
    ports:
      - "3306:3306"
    volumes:
      - db_data:/var/lib/mysql
      - ./db:/docker-entrypoint-initdb.d
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost", "-u", "root", "-prootpassword"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  db_data:
EOF
```
2. Запустите связку web-приложение - БД.
```sh
docker compose up --build 
```
<details><summary>Вывод</summary>

```sh
[+] Building 0.4s (13/13) FINISHED                                              
 => [internal] load local bake definitions                                 0.0s
 => => reading from stdin 554B                                             0.0s
 => [internal] load build definition from Dockerfile                       0.0s
 => => transferring dockerfile: 295B                                       0.0s
 => [internal] load metadata for docker.io/library/python:3.9-slim         0.0s
 => [internal] load .dockerignore                                          0.0s
 => => transferring context: 2B                                            0.0s
 => [1/6] FROM docker.io/library/python:3.9-slim@sha256:2d97f6910b16bd338  0.0s
 => => resolve docker.io/library/python:3.9-slim@sha256:2d97f6910b16bd338  0.0s
 => [internal] load build context                                          0.0s
 => => transferring context: 204B                                          0.0s
 => CACHED [2/6] WORKDIR /app                                              0.0s
 => CACHED [3/6] RUN apt-get update && apt-get install -y     build-essen  0.0s
 => CACHED [4/6] COPY app/requirements.txt .                               0.0s
 => CACHED [5/6] RUN pip install --no-cache-dir -r requirements.txt        0.0s
 => CACHED [6/6] COPY app/ .                                               0.0s
 => exporting to image                                                     0.1s
 => => exporting layers                                                    0.0s
 => => exporting manifest sha256:89824d0299b2c53ae040534b0aa3adc322756b7a  0.0s
 => => exporting config sha256:5ae502194daf7c3f36abbd2b1ecf1c189568b78d47  0.0s
 => => exporting attestation manifest sha256:d7d659373a16f05859566803d7ee  0.0s
 => => exporting manifest list sha256:5d1691ed819a939b4590a5d45355d793265  0.0s
 => => naming to docker.io/library/lab_docker-app:latest                   0.0s
 => => unpacking to docker.io/library/lab_docker-app:latest                0.0s
 => resolving provenance for metadata file                                 0.0s
[+] up 2/2
 ✔ Image lab_docker-app Built                                               0.4s
 ✔ Container lab_docker Created                                             0.1s
Attaching to lab_docker, mysql_db
Container mysql_db Waiting 
mysql_db  | 2026-04-28 09:01:29+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 8.0.46-1.el9 started.
mysql_db  | 2026-04-28 09:01:30+00:00 [Note] [Entrypoint]: Switching to dedicated user 'mysql'
mysql_db  | 2026-04-28 09:01:30+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 8.0.46-1.el9 started.
mysql_db  | 2026-04-28 09:01:30+00:00 [Note] [Entrypoint]: Initializing database files
mysql_db  | 2026-04-28T09:01:30.231269Z 0 [Warning] [MY-011068] [Server] The syntax '--skip-host-cache' is deprecated and will be removed in a future release. Please use SET GLOBAL host_cache_size=0 instead.
mysql_db  | 2026-04-28T09:01:30.231382Z 0 [System] [MY-013169] [Server] /usr/sbin/mysqld (mysqld 8.0.46) initializing of server in progress as process 80
mysql_db  | 2026-04-28T09:01:30.238844Z 1 [System] [MY-013576] [InnoDB] InnoDB initialization has started.
mysql_db  | 2026-04-28T09:01:31.093337Z 1 [System] [MY-013577] [InnoDB] InnoDB initialization has ended.
mysql_db  | 2026-04-28T09:01:32.288099Z 6 [Warning] [MY-010453] [Server] root@localhost is created with an empty password ! Please consider switching off the --initialize-insecure option.
mysql_db  | 2026-04-28 09:01:35+00:00 [Note] [Entrypoint]: Database files initialized
mysql_db  | 2026-04-28 09:01:35+00:00 [Note] [Entrypoint]: Starting temporary server
mysql_db  | 2026-04-28T09:01:35.951510Z 0 [Warning] [MY-011068] [Server] The syntax '--skip-host-cache' is deprecated and will be removed in a future release. Please use SET GLOBAL host_cache_size=0 instead.
mysql_db  | 2026-04-28T09:01:35.952711Z 0 [System] [MY-010116] [Server] /usr/sbin/mysqld (mysqld 8.0.46) starting as process 124
mysql_db  | 2026-04-28T09:01:35.976422Z 1 [System] [MY-013576] [InnoDB] InnoDB initialization


has started.
mysql_db  | 2026-04-28T09:01:36.203751Z 1 [System] [MY-013577] [InnoDB] InnoDB initialization has ended.
mysql_db  | 2026-04-28T09:01:36.450359Z 0 [Warning] [MY-010068] [Server] CA certificate ca.pem is self signed.
mysql_db  | 2026-04-28T09:01:36.450403Z 0 [System] [MY-013602] [Server] Channel mysql_main configured to support TLS. Encrypted connections are now supported for this channel.
mysql_db  | 2026-04-28T09:01:36.453714Z 0 [Warning] [MY-011810] [Server] Insecure configuration for --pid-file: Location '/var/run/mysqld' in the path is accessible to all OS users. Consider choosing a different directory.
mysql_db  | 2026-04-28T09:01:36.474055Z 0 [System] [MY-011323] [Server] X Plugin ready for connections. Socket: /var/run/mysqld/mysqlx.sock
mysql_db  | 2026-04-28T09:01:36.474121Z 0 [System] [MY-010931] [Server] /usr/sbin/mysqld: ready for connections. Version: '8.0.46'  socket: '/var/run/mysqld/mysqld.sock'  port: 0  MySQL Community Server - GPL.
mysql_db  | 2026-04-28 09:01:36+00:00 [Note] [Entrypoint]: Temporary server started.
mysql_db  | '/var/lib/mysql/mysql.sock' -> '/var/run/mysqld/mysqld.sock'
mysql_db  | Warning: Unable to load '/usr/share/zoneinfo/iso3166.tab' as time zone. Skipping it.
mysql_db  | Warning: Unable to load '/usr/share/zoneinfo/leap-seconds.list' as time zone. Skipping it.
mysql_db  | Warning: Unable to load '/usr/share/zoneinfo/leapseconds' as time zone. Skipping it.
mysql_db  | Warning: Unable to load '/usr/share/zoneinfo/tzdata.zi' as time zone. Skipping it.
mysql_db  | Warning: Unable to load '/usr/share/zoneinfo/zone.tab' as time zone. Skipping it.
mysql_db  | Warning: Unable to load '/usr/share/zoneinfo/zone1970.tab' as time zone. Skipping it.
mysql_db  | 2026-04-28 09:01:40+00:00 [Note] [Entrypoint]: Creating database mydb
mysql_db  | 2026-04-28 09:01:40+00:00 [Note] [Entrypoint]: Creating user myuser
mysql_db  | 2026-04-28 09:01:40+00:00 [Note] [Entrypoint]: Giving user myuser access to schema mydb
mysql_db  | 
mysql_db  | 2026-04-28 09:01:40+00:00 [Note] [Entrypoint]: Stopping temporary server
mysql_db  | 2026-04-28T09:01:40.267403Z 14 [System] [MY-013172] [Server] Received SHUTDOWN from user root. Shutting down mysqld (Version: 8.0.46).
Container mysql_db Healthy 
lab_docker  |  * Serving Flask app 'app'
lab_docker  |  * Debug mode: off
lab_docker  | WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
lab_docker  |  * Running on all addresses (0.0.0.0)
lab_docker  |  * Running on http://127.0.0.1:5000
lab_docker  |  * Running on http://172.18.0.3:5000
lab_docker  | Press CTRL+C to quit
mysql_db    | 2026-04-28T09:01:41.344960Z 0 [System] [MY-010910] [Server] /usr/sbin/mysqld: Shutdown complete (mysqld 8.0.46)  MySQL Community Server - GPL.
mysql_db    | 2026-04-28 09:01:42+00:00 [Note] [Entrypoint]: Temporary server stopped
mysql_db    | 
mysql_db    | 2026-04-28 09:01:42+00:00 [Note] [Entrypoint]: MySQL init process done. Ready for start up.
mysql_db    | 
mysql_db    | 2026-04-28T09:01:42.493973Z 0 [Warning] [MY-011068] [Server] The syntax '--skip-host-cache' is deprecated and will be removed in a future release. Please use SET GLOBAL host_cache_size=0 instead.
mysql_db    | 2026-04-28T09:01:42.495166Z 0 [System] [MY-010116] [Server] /usr/sbin/mysqld (mysqld 8.0.46) starting as process 1
mysql_db    | 2026-04-28T09:01:42.502882Z 1 [System] [MY-013576] [InnoDB] InnoDB initialization has started.
mysql_db    | 2026-04-28T09:01:42.724173Z 1 [System] [MY-013577] [InnoDB] InnoDB initialization has ended.
mysql_db    | 2026-04-28T09:01:42.884508Z 0 [Warning] [MY-010068] [Server] CA certificate ca.pem is self signed.
mysql_db    | 2026-04-28T09:01:42.884547Z 0 [System] [MY-013602] [Server] Channel mysql_main configured to support TLS. Encrypted connections are now supported for this channel.
mysql_db    | 2026-04-28T09:01:42.887113Z 0 [Warning] [MY-011810] [Server] Insecure configuration for --pid-file: Location '/var/run/mysqld' in the path is accessible to all OS users. Consider choosing a different


directory.
mysql_db    | 2026-04-28T09:01:42.904498Z 0 [System] [MY-011323] [Server] X Plugin ready for connections. Bind-address: '::' port: 33060, socket: /var/run/mysqld/mysqlx.sock
mysql_db    | 2026-04-28T09:01:42.904616Z 0 [System] [MY-010931] [Server] /usr/sbin/mysqld: ready for connections. Version: '8.0.46'  socket: '/var/run/mysqld/mysqld.sock'  port: 3306  MySQL Community Server - GPL.
```
</details>

3. Проверьте подключение к приложению через браузер. Сделайте снимок экрана.
```sh
docker ps
```
```sh
CONTAINER ID   IMAGE            COMMAND                  CREATED             STATUS                       PORTS                                                    NAMES
f3bcfd2b6561   lab_docker-web   "python app.py"          About an hour ago   Up About an hour             0.0.0.0:5000->5000/tcp, [::]:5000->5000/tcp              lab_docker
4e77be3ce8de   mysql:8.0        "docker-entrypoint.s…"   About an hour ago   Up About an hour (healthy)   0.0.0.0:3306->3306/tcp, [::]:3306->3306/tcp, 33060/tcp   mysql_db
```
```sh
curl http://localhost:5000
```
```sh
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>MVC App</title>
</head>
<body>
    <h1>Список из Базы Данных</h1>
    <ul>
        
            <li>Пример 1</li>
        
            <li>Пример 2</li>
        
    </ul>
</body>
```
<img width="640" height="400" alt="CUy1m46-iK9EPAz7QmUP25df5CdI9gt71V6RnoKvfv10Ai0rCey8KqIiBIuYRjf_NRo4wb5O9b3YAmEeTCH6L3HI" src="https://github.com/user-attachments/assets/3424005e-8295-4d13-a10f-aca546f60183" />

4. Проверьте работу приложения через браузер.

Переходим по ссылке: http://localhost:5000

И видим
<img width="640" height="400" alt="808FAsc5nsfUsALtn3tRYOfHblPtlz06m23AFQiOMc4ilzVWMmGteg0HrWMj2DjLpSsCc8Q6N1A_uSG-mNohr65Q" src="https://github.com/user-attachments/assets/4335cf40-0e80-4a9a-a11a-520e4d30216b" />

5. Коммитим изменения и отправляем на GitHub
```sh
git add .
git commit -m "Complete Docker lab: Flask + MySQL application"
git push origin main
```

