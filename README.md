# Achievement Service

Сервис достижений для платформы CorporationX. Он хранит каталог достижений,
выданные пользователям достижения и текущий прогресс пользователя по каждому
достижению.

## Реализованный функционал

- хранение достижений с названием, описанием, редкостью и количеством баллов;
- поддержка уровней редкости: `COMMON`, `UNCOMMON`, `RARE`, `EPIC`,
  `LEGENDARY`;
- хранение связи пользователя с полученным достижением;
- хранение и увеличение текущего прогресса пользователя;
- поиск прогресса и полученных достижений по `userId`;
- проверка, получал ли пользователь конкретное достижение;
- автоматическое создание записи прогресса при необходимости;
- передача идентификатора пользователя через обязательный HTTP-заголовок
  `x-user-id`;
- передача `x-user-id` во внутренние Feign-запросы;
- применение Liquibase-миграций при запуске приложения.

REST-контроллеры в текущей версии не добавлены: сервис предоставляет
подготовленный слой хранения и интеграционную инфраструктуру для дальнейшего
подключения обработчиков событий и API.

## Технологии

- Java 17;
- Spring Boot 3;
- Spring Data JPA/Hibernate;
- PostgreSQL;
- Redis;
- Spring Cloud OpenFeign;
- Liquibase;
- Gradle;
- Testcontainers, JUnit 5 и AssertJ.

## Требования

- JDK 17 или новее;
- PostgreSQL 12+;
- Redis 6+;
- Docker (для контейнерного запуска).

Приложение использует базу PostgreSQL и ожидает, что таблица `users` уже
существует в этой базе: миграции сервиса создают внешние ключи на неё.

## Конфигурация

Основные параметры задаются переменными окружения:

| Переменная | По умолчанию | Назначение |
|---|---:|---|
| `DB_HOST` | `localhost` | хост PostgreSQL |
| `DB_PORT` | `5432` | порт PostgreSQL |
| `DB_NAME` | `postgres` | имя базы данных |
| `DB_USERNAME` | `user` | пользователь PostgreSQL |
| `DB_PASSWORD` | `password` | пароль PostgreSQL |
| `REDIS_HOST` | `localhost` | хост Redis |
| `REDIS_PORT` | `6379` | порт Redis |
| `PROJECT_SERVICE_HOST` | `localhost` | хост project-service |
| `PROJECT_SERVICE_PORT` | `8082` | порт project-service |

Порт приложения: `8085`.

## Локальный запуск

Соберите проект из корневой директории:

```bash
./gradlew clean build
```

В Windows используйте:

```powershell
.\gradlew.bat clean build
```

Запустите приложение:

```bash
java -jar build/libs/service.jar
```

Перед запуском убедитесь, что PostgreSQL и Redis доступны по адресам из
конфигурации. При старте Liquibase автоматически применит миграции из
`src/main/resources/db/changelog`.

Для запуска тестов:

```bash
./gradlew test
```

## Запуск в Docker

Сначала соберите jar-файл:

```bash
./gradlew bootJar
```

Создайте Docker-образ:

```bash
docker build -t achievement-service .
```

Запустите контейнер в Docker-сети, где доступны PostgreSQL и Redis:

```bash
docker run --name achievement-service \
  --network <network-name> \
  -p 8085:8085 \
  -e DB_HOST=<postgres-container> \
  -e DB_PORT=5432 \
  -e DB_NAME=<database-name> \
  -e DB_USERNAME=<database-user> \
  -e DB_PASSWORD=<database-password> \
  -e REDIS_HOST=<redis-container> \
  -e REDIS_PORT=6379 \
  achievement-service
```

Если зависимости опубликованы на хост-машине, контейнер можно запустить с
параметрами `-e DB_HOST=host.docker.internal` и
`-e REDIS_HOST=host.docker.internal`.

Dockerfile открывает порт `8085` и запускает собранный файл
`build/libs/service.jar`.

## Структура проекта

- `model` — JPA-сущности достижений, прогресса и полученных достижений;
- `repository` — репозитории для работы с PostgreSQL;
- `config/context` — получение и хранение текущего `userId`;
- `client` — Feign-конфигурация для внутренних вызовов;
- `src/main/resources/db/changelog` — Liquibase-миграции и начальные данные.
