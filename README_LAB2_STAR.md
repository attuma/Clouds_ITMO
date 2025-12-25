# Лаб 2*: Docker Compose Best Practices

## 1. Плохой docker-compose.yml

```yaml
version: '3'

services:
  web:
    image: python:latest
    ports:
      - "5000:5000"
    volumes:
      - .:/app
    working_dir: /app
    command: python app.py

  db:
    image: postgres:latest
    environment:
      POSTGRES_PASSWORD: password123
    ports:
      - "5432:5432"
    volumes:
      - ./data:/var/lib/postgresql/data
```

---

## 2. Хороший docker-compose.yml

```yaml
version: '3.8'

services:
  web:
    image: python:3.11-slim
    ports:
      - "127.0.0.1:5000:5000"
    volumes:
      - ./app:/app
    working_dir: /app
    command: python app.py
    resources:
      limits:
        cpus: '0.5'
        memory: 512M
    networks:
      - web_network
    depends_on:
      - db

  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    resources:
      limits:
        cpus: '1'
        memory: 1G
    networks:
      - db_network

networks:
  web_network:
    driver: bridge
  db_network:
    driver: bridge

volumes:
  postgres_data:
```

---

## 3. Почему так?

### Ошибка 1: Использование latest тагов вместо конкретных версий

- В плохом Dockerfile мы используем `image: python:latest` и `image: postgres:latest`. Latest всегда берёт последнюю версию, на разных компьютерах может быть разные версии.
- В хорошем Dockerfile мы используем `image: python:3.11-slim` и `image: postgres:15-alpine`. Это конкретные версии, одинаково везде.
- Это изменение делает приложение стабильным и предсказуемым.

---

### Ошибка 2: Отсутствие ограничений ресурсов

- В плохом docker-compose не указаны `resources/limits`. Один контейнер может занять 100% CPU и всю память, и тогда другой контейнер не будет работать.
- В хорошем docker-compose мы указываем `resources/limits` для каждого сервиса. Web получает максимум 0.5 CPU и 512M памяти, DB получает 1 CPU и 1G памяти.
- Это изменение защищает систему от перегрузки и делает её стабильной.

---

### Ошибка 3: Все сервисы видят друг друга по сети

- В плохом docker-compose все сервисы находятся в одной default сети. Это значит, что любой сервис может подключиться к любому другому, это security риск.
- В хорошем docker-compose мы создаём отдельные сети: `web_network` для веб-сервиса и `db_network` для БД. Сервисы не видят друг друга.
- Это изменение повышает безопасность и изолирует сервисы.

---

## 4. Как добились изоляции контейнеров по сети

В хорошем `docker-compose.yml` мы создаём две отдельные сети:

```yaml
networks:
  web_network:
    driver: bridge
  db_network:
    driver: bridge
```

И назначаем сервисы разным сетям:

```yaml
services:
  web:
    networks:
      - web_network    # web только в этой сети

  db:
    networks:
      - db_network     # db только в этой сети
```

**Принцип изоляции:**

- `web_network` - для веб-приложения (может подключиться к Интернету)
- `db_network` - только для БД (закрытая сеть)
- Web и DB находятся в разных сетях и не видят друг друга

Если нужно чтобы они общались, добавляем обе сети для одного сервиса:

```yaml
services:
  db:
    networks:
      - web_network    # теперь db видит web
      - db_network     # и остаётся в своей сети
```

Но в нашем примере они полностью изолированы для максимальной безопасности.

---

## 5. Запуск
<img width="1280" height="175" alt="image" src="https://github.com/user-attachments/assets/2b170075-41b3-43af-9823-af196f08b53d" />
<img width="1280" height="831" alt="image" src="https://github.com/user-attachments/assets/e4062414-0774-40eb-aaba-3f7ab86711af" />


