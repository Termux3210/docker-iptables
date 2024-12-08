## Описание проекта

Проект "docker-iptables" представляет собой конфигурацию сетевого шлюза с использованием WireGuard и Docker для управления VPN-трафиком. Он включает в себя несколько сервисов, таких как `wg-easy`, `peer`, `gateway` и `nginx`, которые взаимодействуют друг с другом через изолированные сети Docker.

## Структура проекта

Проект состоит из следующих компонентов:

- **wg-easy**: Упрощенный интерфейс для управления WireGuard.
- **peer**: Клиентская конфигурация WireGuard.
- **gateway**: Шлюз для маршрутизации трафика между сетями.
- **nginx**: Веб-сервер для обработки HTTP-запросов.

## Сетевые правила

### Правила Gateway

```plaintext
:INPUT ACCEPT [0:0]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
-A FORWARD -i eth0 -o eth1 -j ACCEPT
-A FORWARD -i eth1 -o eth0 -j ACCEPT
```

### Правила Peer

```plaintext
:PREROUTING ACCEPT [0:0]
:INPUT ACCEPT [0:0]
:OUTPUT ACCEPT [2:144]
:POSTROUTING ACCEPT [0:0]
-A OUTPUT -d 127.0.0.11/32 -j DOCKER_OUTPUT
-A POSTROUTING -d 127.0.0.11/32 -j DOCKER_POSTROUTING
-A POSTROUTING -s 10.1.0.0/24 -o eth0 -j MASQUERADE
```

### Правила wg-easy

```plaintext
:PREROUTING ACCEPT [3:228]
:INPUT ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
:POSTROUTING ACCEPT [0:0]
-A OUTPUT -d 127.0.0.11/32 -j DOCKER_OUTPUT
-A POSTROUTING -d 127.0.0.11/32 -j DOCKER_POSTROUTING
```

### Правила Nginx

```plaintext
:INPUT DROP [21:1764]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
-A INPUT -p tcp -m tcp --dport 80 -j ACCEPT
```

## Docker Compose

Для развертывания всех сервисов используется файл `docker-compose.yaml`. Пример конфигурации:

```yaml
services:
  wg-easy:
    environment:
      - WG_HOST=10.2.0.3
      ...
    image: ghcr.io/wg-easy/wg-easy
    ...

  peer:
    build: .
    ...

  gateway:
    image: nicolaka/netshoot
    ...

  nginx:
    image: nginx
    ...
```

### Сети

Проект использует три сети Docker:

- **net_A** (10.1.0.0/24)
- **net_B** (10.2.0.0/24)
- **net_C** (10.3.0.0/24)

Каждая сеть имеет свой шлюз и предназначена для отдельного сегмента трафика.

## Установка и запуск

1. Убедитесь, что Docker и Docker Compose установлены на вашей системе.
2. Склонируйте репозиторий с проектом.
3. Перейдите в директорию проекта.
4. Запустите команду:

   ```bash
   docker-compose up -d
   ```

Это создаст и запустит все необходимые контейнеры.

## Заключение

Проект "docker-iptables" обеспечивает надежное и безопасное управление VPN-трафиком с помощью WireGuard и Docker, позволяя легко настраивать и управлять сетевыми правилами и конфигурациями.

Если у вас есть вопросы или предложения, не стесняйтесь обращаться!
