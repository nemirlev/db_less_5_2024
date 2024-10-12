# Семинар по Docker, Docker Compose и PostgreSQL

## Введение

Цель семинара — познакомиться с основами Docker и Docker Compose, а также научиться работать с PostgreSQL в контейнерах.
Вы изучите, как создавать Dockerfile, управлять контейнерами через Docker Compose, сохранять данные с помощью volume и
управлять сетями. После каждого шага делайте скриншоты и описывайте результаты.

### Примерные шаги:

1. Основные команды Docker и типы образов
2. Создание Dockerfile для PostgreSQL
3. Multi-stage build
4. Работа с Docker Compose
5. Управление volume и сетями в Docker Compose
6. Подключение к PostgreSQL через DataGrip
7. Тестовые SQL-запросы

---

### 1. Основные команды Docker и типы образов

1.1 **Запуск контейнера:**

```bash
docker run -d -p 8080:80 nginx
```

Команда запускает контейнер с Nginx на порту 8080. Флаг -d запускает контейнер в фоновом режиме, а -p пробрасывает
порт (8080 -> 80).

```bash
docker ps
```
Посмотреть контейнеры 

1.2 **Остановка и удаление контейнера:**

```bash
docker ps
```

Смотрим список контейнеров и копируем ID.

```bash
docker stop <container_id>
#  После остановки опять введите docker ps , а потом docker ps -a
docker rm <container_id>
```

Команда docker stop останавливает контейнер, а docker rm удаляет его.

1.3 **Виды образов:**

* Обычный образ: стандартная версия образа с полным набором инструментов. Пример — `postgres:latest`.
* Alpine образ: минимальный образ с уменьшенным размером. Пример — `postgres:alpine`.

```bash
docker run -d -p 5432:5432 -e "POSTGRES_PASSWORD=password" postgres:alpine
docker run -d -p 5433:5432 -e "POSTGRES_PASSWORD=password" postgres:latest
```

Посмотреть логи 

```bash
docker logs -f <container_id>
```

Сравните размер образов с помощью команды `docker images`.

А после удалите

```bash
docker ps
docker stop <container_id>
docker stop <container_id>
docker rm <container_id>
docker rm <container_id>
```

1.4. **Заходим в контейнер:**

```bash
docker exec -it <container_id> bash
```

Команда позволяет зайти в контейнер и запустить bash.

Давайте зайдем в контейнер с PostgreSQL и выполним команду `psql`.

1.5 **Базовые команды для работы с образами и контейнерами**:

* docker pull [image]: загрузка образа.
* docker build -t [name] . : сборка образа из Dockerfile.
* docker ps: список запущенных контейнеров.
* docker exec -it [container_id] bash: запуск bash внутри контейнера.

### 2. Создание Dockerfile для PostgreSQL

2.1 **Создание простого Dockerfile:** Создайте файл Dockerfile в новой директории:

```Dockerfile
# Указываем базовый образ
FROM postgres:latest

# Устанавливаем переменные окружения для базы данных
ENV POSTGRES_USER=student
ENV POSTGRES_PASSWORD=student
ENV POSTGRES_DB=mydb

# Пробрасываем порт 5432
EXPOSE 5432
```

Основные команды Dockerfile:

* FROM: базовый образ.
* ENV: переменные окружения.
* EXPOSE: указание порта
* RUN: команда для установки пакетов.
* COPY: копирование файлов.
* CMD: команда для запуска контейнера.
* WORKDIR: рабочая директория.

2.2 **Собираем образ:**

```bash
docker build -t my_postgres .
```

Команда собирает образ с именем my_postgres.

2.3 **Запускаем контейнер:**

```bash
docker run -d -p 5432:5432 my_postgres
```

Команда запускает контейнер с созданным образом.

2.3. Тегируем образ и пушим его на Docker Hub:

```bash
docker tag my_postgres <username>/my_postgres:latest
docker login
docker push <username>/my_postgres
```

### 3. Multi-stage build

3.1 **Создание Dockerfile с multi-stage build:**

```Dockerfile
# Этап 1: сборка приложения
FROM node:alpine as builder
WORKDIR /app
COPY . .
RUN npm install && npm run build

# Этап 2: финальный минимальный образ
FROM nginx:alpine
COPY --from=builder /app/build /usr/share/nginx/html
EXPOSE 80
```

Такой подход уменьшает размер финального образа, так как в нем содержатся только необходимые файлы.

### 4. Работа с Docker Compose

4.1 **Создание docker-compose.yml:**

```yml
services:
  postgres:
    image: postgres:latest
    container_name: postgres_db
    environment:
      POSTGRES_USER: student
      POSTGRES_PASSWORD: student
      POSTGRES_DB: mydb
    ports:
      - "5432:5432"
```

4.2. Запуск контейнеров:

    ```bash
    docker compose up -d
    ```

4.3 **Остановка и удаление контейнеров:**

```bash
docker compose down
```

4.4 **Управление volume:**

```yml
services:
  postgres:
    image: postgres:latest
    container_name: postgres_db
    environment:
      POSTGRES_USER: student
      POSTGRES_PASSWORD: student
      POSTGRES_DB: mydb
    ports:
      - "5432:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data
  volumes:
    pgdata:
```

Volume позволяет сохранять данные базы данных при перезапуске контейнеров.

4.5 Volume: Volume — это механизм для сохранения данных за пределами контейнера. Данные, которые сохраняются в volume,
остаются после перезапуска или удаления контейнера.

```bash
docker volume ls
```
Эта команда покажет все созданные volumes.

4.6. **Управление сетями:**

```yml
networks:
  app_network:
    driver: bridge

services:
  postgres:
    image: postgres:latest
    networks:
      - app_network
```

Docker автоматически создает сеть для взаимодействия контейнеров. Вы можете явно указать сеть в docker-compose.yml:

```bash
docker network ls
```

Сеть может быть `external` и `internal`, а также иметь разные драйверы.

### 5. Подключение к PostgreSQL через DataGrip

5.1 **Подключение к PostgreSQL через DataGrip:**

Создаем docker-compose.yml:

```yml
services:
  postgres:
    image: postgres:latest
    container_name: postgres_db
    environment:
      POSTGRES_USER: student
      POSTGRES_PASSWORD: student
      POSTGRES_DB: mydb
    ports:
      - "5432:5432"
```

Запускаем контейнер:

```bash
docker-compose up -d
```

Открываем DataGrip и создаем новое подключение с типом PostgreSQL:

* Host: localhost
* Port: 5432
* Database: mydb
* User: student
* Password: student
* Driver: PostgreSQL

Нажимаем Test Connection, если все ок - сохраняем. У вас автоматически откроется консоль с SQL-запросами.

### 6. Тестовые SQL-запросы

6.1 **Тестовые SQL-запросы:**

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    age INT
);
```

Сгенерируем 10 000 записей:

```sql
INSERT INTO users (name, age)
SELECT
    md5(random()::text),
    floor(random() * 100)
FROM generate_series(1, 10000);
```

Выберем всех пользователей старше 50 лет:

```sql
SELECT * FROM users WHERE age > 50;
```

Получаем данные

```sql
SELECT * FROM users;
```

### 7. Самостоятельное выполнение

1. Создается docker-compose.yml с двумя сервисами: PostgreSQL
2. Загрузите туда данные
3. Подключитесь к базе данных через DataGrip
4. Выполните тестовые SQL-запросы
5. Удалите контейнеры
6. Сделайте скриншоты и опишите результаты
7. Запустите контейнеры снова и проверьте, что данные остались или нет
8. Сделайте скриншоты и опишите результаты