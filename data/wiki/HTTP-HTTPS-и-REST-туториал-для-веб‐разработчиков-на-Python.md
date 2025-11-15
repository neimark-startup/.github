# HTTP/HTTPS и REST туториал для веб-разработчиков на Python

## Содержание
- [Полезные советы по HTTP/HTTPS/REST](#полезные-советы-по-httphttpsrest)
- [Посмотреть IP-адрес](#посмотреть-ip-адрес)
- [Посмотреть открытые порты](#посмотреть-открытые-порты)
- [Определить IP-адрес по DNS-имени](#определить-ip-адрес-по-dns-имени)
- [Использование telnet](#использование-telnet)
- [Использование cURL](#использование-curl)
- [Библиотека 'requests'](#библиотека-requests)
  - [Установка](#установка)
  - [HTTP-клиент с помощью библиотеки 'requests'](#http-клиент-с-помощью-библиотеки-requests)
  - [Документация](#документация)
  - [Практика. Библиотека 'requests'](#практика-библиотека-requests)
- [Библиотека 'socket'](#библиотека-socket)
  - [Веб-сервер с использованием библиотеки 'socket'](#веб-сервер-с-использованием-библиотеки-socket)
  - [Практика. Библиотека 'socket'](#практика-библиотека-socket)
- [Фреймворк 'Flask'](#фреймворк-flask)
  - [Установка](#установка-1)
  - [Веб-сервер с использованием фреймворка 'Flask'](#веб-сервер-с-использованием-фреймворка-flask)
  - [Практика. Фреймворк 'Flask'](#практика-фреймворк-flask)
- [Docker-compose](#docker-compose)
  - [Пример `docker-compose.yml`](#пример-docker-composeyml)
  - [Практика. docker-compose](#практика-docker-compose)

## Полезные советы по HTTP/HTTPS/REST

**Безопасность:**
- Всегда используйте HTTPS в продакшене
- Валидируйте и санитизируйте все входящие данные
- Используйте окружения для хранения секретов (не храните в коде)

**REST API лучшие практики:**
- Используйте соответствующие HTTP-методы (GET, POST, PUT, DELETE)
- Возвращайте правильные HTTP-статусы (200, 201, 400, 404, 500)
- Используйте понятные URL пути (`/api/v1/users`)
- Версионируйте API
- Используйте пагинацию для больших наборов данных

**Производительность:**
- Используйте кеширование заголовков
- Сжимайте ответы (gzip)
- Используйте соединения keep-alive

## Посмотреть IP-адрес

```sh
ip a
# либо "ifconfig" - устаревший аналог из пакета "net-tools"

# Просмотр адресов IPv4
ip -4 a
```

## Посмотреть открытые порты

```sh
ss -tunlp
# sudo ss -tunlp

# либо "netstat -tunlp" - устаревший аналог из пакета "net-tools"
```

## Определить IP-адрес по DNS-имени

```sh
nslookup ya.ru
# либо
dig ya.ru

# Получение IP-адреса по доменному имени у указанного DNS-сервера
nslookup ya.ru 8.8.8.8
dig @8.8.8.8 ya.ru
```

## Использование telnet

```sh
telnet ifconfig.co 80

GET /ip HTTP/1.1
Host: ifconfig.co
```

## Использование cURL

```sh
curl -i -X PUT 'https://postman-echo.com/put' --data 'This is expected to be sent back as part of response body.'
```

## Библиотека 'requests'

### Установка

```sh
pip install requests
```

### HTTP-клиент с помощью библиотеки 'requests'

1. Создайте директорию для приложения `httpclient` и перейдите в неё.
2. Создайте файл `requirements.txt` и добавьте в него строку:
    ```
    requests
    ```
3. Создайте файл `httpclient.py` и скопируйте в него код:
    ```python
    import requests
    import json

    response = requests.get('http://10.125.28.96/')
    result = response.json()
    print(result)
    ```

4. Создайте `Dockerfile`:
    ```dockerfile
    FROM python:3.9-alpine

    ENV SRC_PATH /app
    RUN mkdir -p $SRC_PATH
    WORKDIR $SRC_PATH

    COPY requirements.txt $SRC_PATH/
    RUN pip install -r requirements.txt

    COPY . .

    CMD ["python", "httpclient.py"]
    ```

5. Соберите образ:
    ```bash
    docker build -t httpclient .
    ```

6. Запустите приложение:
    ```bash
    docker run --rm httpclient
    ```

### Документация

* https://docs.python-requests.org/en/latest/

### Практика. Библиотека 'requests'

С помощью библиотеки `requests` выполнить запрос:
```sh
curl -i -X PUT 'https://postman-echo.com/put' --data 'This is expected to be sent back as part of response body.'
```

Пример решения:
```python
import requests

response = requests.put(
    'https://postman-echo.com/put', 
    data='This is expected to be sent back as part of response body.'
)
print(f"Status: {response.status_code}")
print(f"Headers: {dict(response.headers)}")
print(f"Body: {response.text}")
```

## Библиотека 'socket'

Документация: https://docs.python.org/3/library/socket.html

### Веб-сервер с использованием библиотеки 'socket'

1. Создайте папку для проекта и перейдите в нее.
2. Создайте файл `web_server.py` и скопируйте в него код:
    ```python
    import socket
    from datetime import datetime

    server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server_socket.bind(('0.0.0.0', 5678))
    server_socket.listen(1)

    while True:
        session, addr = server_socket.accept()
        request = session.recv(1024).decode()
        print(request)

        # Формируем HTTP-ответ
        response_body = "Hello world! The time is " + str(datetime.now())
        response_headers = [
            "HTTP/1.1 200 OK",
            "Content-Type: text/html; charset=UTF-8",
            "Content-Length: " + str(len(response_body)),
            "",
            ""
        ]
        response = "\r\n".join(response_headers) + response_body
        session.send(response.encode())
        session.close()
    ```

3. Запустите docker-контейнер с python и запустите в нем этот скрипт:

    ```sh
    docker run --rm -td -p 8080:5678 -v `pwd`:/app python:3.9-alpine python /app/web_server.py
    ```

    Эта команда вернет идентификатор контейнера.

4. Отправьте запрос в запущенное приложение. Для этого выполните команду `curl 127.0.0.1:8080/test` или откройте страницу в браузере.

5. Посмотрите логи приложения с помощью команды:
    ```sh
    docker logs [identifier]
    ```

6. Остановите контейнер с помощью команды:
    ```sh
    docker stop [identifier]
    ```

### Практика. Библиотека 'socket'

Доработать веб-сервер так, чтобы
- `GET /time` возвращал текущее время
- `GET /hello` возвращал `"hello"`
- `GET /sum?a=1&b=2` возвращал сумму параметров `a` и `b`

Пример решения:
```python
import socket
from datetime import datetime
from urllib.parse import parse_qs, urlparse

def handle_request(request):
    """Обрабатывает HTTP-запрос и возвращает HTTP-ответ."""
    lines = request.split('\r\n')
    if not lines:
        return "HTTP/1.1 400 Bad Request\r\n\r\n"

    method, path, version = lines[0].split(' ', 2)
    parsed_path = urlparse(path)
    path = parsed_path.path
    query_params = parse_qs(parsed_path.query)

    if path == '/time':
        response_body = str(datetime.now())
    elif path == '/hello':
        response_body = "hello"
    elif path == '/sum':
        a = int(query_params.get('a', [0])[0])
        b = int(query_params.get('b', [0])[0])
        response_body = str(a + b)
    else:
        response_body = "Not Found"

    response_headers = [
        "HTTP/1.1 200 OK",
        "Content-Type: text/plain; charset=UTF-8",
        "Content-Length: " + str(len(response_body)),
        "",
        ""
    ]
    response = "\r\n".join(response_headers) + response_body
    return response

server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server_socket.bind(('0.0.0.0', 5678))
server_socket.listen(1)

while True:
    session, addr = server_socket.accept()
    request = session.recv(1024).decode()
    print(request)

    response = handle_request(request)

    session.send(response.encode())
    session.close()
```

## Фреймворк 'Flask'

### Установка

```sh
pip install flask
```

### Веб-сервер с использованием фреймворка 'Flask'

1. Создайте папку для проекта и перейдите в неё.
2. Создайте файл `requirements.txt` и добавьте в него:
    ```
    flask
    ```
3. Создайте файл `flask_server.py` и скопируйте в него код:
    ```python
    from flask import Flask

    app = Flask(__name__)

    @app.route('/hello')
    def hello():
        return 'Hello world!'

    if __name__ == '__main__':
        app.run(host='0.0.0.0', port=5678)
    ```

4. Создайте `Dockerfile`:
    ```dockerfile
    FROM python:3.9-alpine

    ENV SRC_PATH /app
    RUN mkdir -p $SRC_PATH
    WORKDIR $SRC_PATH

    COPY requirements.txt $SRC_PATH/
    RUN pip install -r requirements.txt

    COPY . .

    CMD ["python", "flask_server.py"]
    ```

5. Соберите образ с тэгом `flask_server`.
6. Запустите приложение:
    ```bash
    docker run --rm -t -p 8080:5678 -v `pwd`:/app flask_server
    ```
7. Перейдите по адресу `http://127.0.0.1:8080/hello` для проверки.

### Практика. Фреймворк 'Flask'

Разработать веб-сервер с использованием `Flask`, чтобы
- `GET /time` возвращал текущее время
- `GET /hello` возвращал "hello"
- `GET /sum?a=1&b=2` возвращал сумму параметров a и b

Пример решения:
```python
from flask import Flask, request
from datetime import datetime

app = Flask(__name__)

@app.route('/time')
def time():
    return str(datetime.now())

@app.route('/hello')
def hello():
    return "hello"

@app.route('/sum')
def sum_route():
    a = request.args.get('a', 0, type=int)
    b = request.args.get('b', 0, type=int)
    return str(a + b)

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5678)
```

## Docker-compose

### Пример `docker-compose.yml`

```yml
version: '3'
services:
  app:
    build: .
    command: python /app/flask_server.py
    volumes:
      - .:/app
    ports:
      - 8080:5678
```

### Практика. docker-compose

Напишите `docker-compose.yml`, в котором будут два сервиса: клиент и сервер.

Клиент будет обращаться к серверу, который рассчитывает сумму двух переданных чисел, и выводить на экран результат (сумму) в формате:

`3 + 2 = 5`

Пример:

Сервер (файл `server.py`):
```python
from flask import Flask, request

app = Flask(__name__)

@app.route('/sum')
def sum_route():
    a = request.args.get('a', 0, type=int)
    b = request.args.get('b', 0, type=int)
    return str(a + b)

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5678)
```

Клиент (файл `client.py`):
```python
import requests
import os

server_host = os.getenv('SERVER_HOST', 'server')
server_port = os.getenv('SERVER_PORT', '5678')

a = 3
b = 2
url = f'http://{server_host}:{server_port}/sum?a={a}&b={b}'
response = requests.get(url)
result = response.text
print(f"{a} + {b} = {result}")
```

`docker-compose.yml`:
```yml
version: '3'
services:
  server:
    build: .
    command: python /app/server.py
    volumes:
      - .:/app
    ports:
      - "5678:5678"

  client:
    build: .
    command: python /app/client.py
    volumes:
      - .:/app
    environment:
      - SERVER_HOST=server
      - SERVER_PORT=5678
    depends_on:
      - server
```

Запуск:
```sh
docker-compose up
```