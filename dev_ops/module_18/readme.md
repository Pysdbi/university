# Модуль 18.6: Практическая работа по SSH и Wireshark

## Задание 1: Настройка SSH с аутентификацией по ключам

### Цель
Попрактиковаться в настройке подключения по SSH к хосту с аутентификацией по ключам.

### Что нужно сделать
1. Поднять Docker контейнер Ubuntu с SSH сервером
2. Настроить SSH подключение с аутентификацией по ключам
3. Определить fingerprint виртуальной машины

### Пошаговое выполнение

#### Шаг 1: Создание Dockerfile для SSH сервера
```dockerfile
FROM ubuntu:22.04

# Обновление системы и установка SSH сервера
RUN apt-get update && apt-get install -y \
    openssh-server \
    sudo \
    curl \
    && rm -rf /var/lib/apt/lists/*

# Создание пользователя для SSH
RUN useradd -m -s /bin/bash sshuser && \
    echo 'sshuser:password123' | chpasswd && \
    usermod -aG sudo sshuser

# Настройка SSH сервера
RUN mkdir /var/run/sshd && \
    sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config && \
    sed -i 's/#PasswordAuthentication yes/PasswordAuthentication yes/' /etc/ssh/sshd_config && \
    sed -i 's/#PubkeyAuthentication yes/PubkeyAuthentication yes/' /etc/ssh/sshd_config

# Создание директории для SSH ключей
RUN mkdir -p /home/sshuser/.ssh && \
    chown sshuser:sshuser /home/sshuser/.ssh && \
    chmod 700 /home/sshuser/.ssh

# Экспорт порта SSH
EXPOSE 22

# Запуск SSH сервера
CMD ["/usr/sbin/sshd", "-D"]
```

#### Шаг 2: Сборка и запуск контейнера
```bash
# Сборка образа
docker build -t ubuntu-ssh .

# Запуск контейнера
docker run -d --name ssh-server -p 2222:22 ubuntu-ssh
```

#### Шаг 3: Генерация SSH ключей на macOS
```bash
# Генерация пары ключей
ssh-keygen -t rsa -b 4096 -f ~/.ssh/ubuntu_ssh_key -N ""

# Копирование публичного ключа в контейнер с помощью ssh-copy-id
# ssh-copy-id автоматически:
# - Создает директорию ~/.ssh если её нет
# - Устанавливает правильные права доступа (700 для директории, 600 для authorized_keys)
# - Добавляет ключ в authorized_keys
ssh-copy-id -i ~/.ssh/ubuntu_ssh_key.pub -p 2222 sshuser@localhost
```

#### Шаг 4: Подключение по SSH
```bash
# Подключение с использованием приватного ключа
ssh -i ~/.ssh/ubuntu_ssh_key -p 2222 sshuser@localhost

# Получение fingerprint сервера
ssh-keyscan -p 2222 localhost
```

### Возможные проблемы и решения

#### Проблема: `ssh-keyscan` ничего не выдает

**Причины:**
1. SSH сервер еще не полностью запустился
2. Порт не проброшен правильно
3. SSH сервер не слушает на нужном порту
4. Проблемы с сетевой конфигурацией

**Решения:**
```bash
# 1. Проверка статуса контейнера
docker ps | grep ssh-server

# 2. Проверка портов
docker port ssh-server

# 3. Проверка логов SSH
docker logs ssh-server --tail 20

# 4. Проверка доступности порта
nc -zv localhost 2222

# 5. Перезапуск контейнера
docker restart ssh-server

# 6. Альтернативные способы получения fingerprint
ssh-keyscan -p 2222 127.0.0.1
ssh -o StrictHostKeyChecking=no -p 2222 sshuser@localhost "echo test" 2>&1 | grep fingerprint
```

#### Проблема: SSH подключение не работает

**Решения:**
```bash
# Проверка SSH процессов в контейнере
docker exec ssh-server ps aux | grep sshd

# Проверка конфигурации SSH
docker exec ssh-server cat /etc/ssh/sshd_config | grep -E "(Port|PasswordAuthentication)"

# Тест подключения по паролю
ssh -p 2222 sshuser@localhost
```

### Ожидаемый результат
- Успешное подключение к виртуальной машине через SSH
- Fingerprint сервера будет выглядеть примерно так:
```
localhost:2222 ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC7... (SHA256:abcd1234...)
```

### Диагностические скрипты

#### Основной скрипт настройки:
```bash
chmod +x setup_ssh.sh
./setup_ssh.sh
```

#### Быстрый тест SSH:
```bash
chmod +x test_ssh.sh
./test_ssh.sh
```

#### Полная диагностика:
```bash
chmod +x diagnose_ssh.sh
./diagnose_ssh.sh
```

#### Ручные команды для диагностики:
```bash
# Проверка статуса контейнера
docker ps | grep ssh-server

# Проверка логов
docker logs ssh-server

# Проверка портов
docker port ssh-server

# Тест подключения
ssh -o StrictHostKeyChecking=no -p 2222 sshuser@localhost
```

---

## Задание 2: Настройка SSH сервера на порт 2222

### Цель
Попрактиковаться в настройке сервера SSH на нестандартный порт.

### Что нужно сделать
Настроить SSH-сервер на работу по порту 2222.

### Выполнение

#### Обновление Dockerfile для порта 2222
```dockerfile
FROM ubuntu:22.04

RUN apt-get update && apt-get install -y \
    openssh-server \
    sudo \
    curl \
    && rm -rf /var/lib/apt/lists/*

RUN useradd -m -s /bin/bash sshuser && \
    echo 'sshuser:password123' | chpasswd && \
    usermod -aG sudo sshuser

RUN mkdir /var/run/sshd && \
    sed -i 's/#Port 22/Port 2222/' /etc/ssh/sshd_config && \
    sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config && \
    sed -i 's/#PasswordAuthentication yes/PasswordAuthentication yes/' /etc/ssh/sshd_config && \
    sed -i 's/#PubkeyAuthentication yes/PubkeyAuthentication yes/' /etc/ssh/sshd_config

RUN mkdir -p /home/sshuser/.ssh && \
    chown sshuser:sshuser /home/sshuser/.ssh && \
    chmod 700 /home/sshuser/.ssh

EXPOSE 2222

CMD ["/usr/sbin/sshd", "-D"]
```

#### Запуск контейнера
```bash
# Остановка предыдущего контейнера
docker stop ssh-server && docker rm ssh-server

# Сборка нового образа
docker build -t ubuntu-ssh-port2222 .

# Запуск с портом 2222
docker run -d --name ssh-server-2222 -p 2222:2222 ubuntu-ssh-port2222
```

#### Подключение по порту 2222
```bash
# Подключение по новому порту
ssh -i ~/.ssh/ubuntu_ssh_key -p 2222 sshuser@localhost
```

### Ожидаемый результат
SSH-клиент успешно подключается к виртуальной машине по порту 2222.

---

## Задание 3: Работа с Wireshark (tshark)

### Цель
Попрактиковаться в работе с консольной версией Wireshark для анализа сетевого трафика.

### Что нужно сделать
1. Установить tshark (консольная версия Wireshark)
2. Перехватить трафик при запросе `curl google.com`
3. Отфильтровать только DNS запросы
4. Ответить на вопросы о DNS протоколе

### Выполнение

#### Установка tshark на macOS
```bash
# Установка через Homebrew
brew install wireshark

# Или установка только tshark
brew install tshark
```

#### Перехват DNS трафика
```bash
# Запуск перехвата трафика с фильтром DNS
sudo tshark -i en0 -f "port 53" -w dns_capture.pcap

# В другом терминале выполнить запрос
curl google.com

# Остановить перехват (Ctrl+C) и проанализировать
tshark -r dns_capture.pcap -Y "dns"
```

#### Альтернативный способ с live анализом
```bash
# Просмотр DNS запросов в реальном времени
sudo tshark -i en0 -Y "dns" -T fields -e dns.qry.name -e dns.a

# Выполнить запрос в другом терминале
curl google.com
```

#### Анализ результатов
```bash
# Детальный анализ DNS пакетов
tshark -r dns_capture.pcap -Y "dns" -V

# Статистика DNS запросов
tshark -r dns_capture.pcap -Y "dns" -T fields -e dns.qry.name | sort | uniq -c
```

### Ответы на вопросы

#### 1. На каком уровне в стеке протоколов TCP/IP располагается DNS?
**Ответ:** DNS располагается на **прикладном уровне (Application Layer)** стека протоколов TCP/IP.

#### 2. Какой протокол транспортного уровня используется для DNS? Почему?
**Ответ:** DNS использует **UDP (User Datagram Protocol)** на транспортном уровне.

**Причины использования UDP:**
- **Простота:** DNS запросы обычно короткие и не требуют сложной логики установления соединения
- **Скорость:** UDP быстрее TCP, так как не требует handshake
- **Эффективность:** DNS запросы обычно помещаются в один UDP пакет
- **Меньше накладных расходов:** Нет необходимости в подтверждениях и повторных передачах для простых запросов
- **Timeout механизм:** Клиент может легко повторить запрос при отсутствии ответа

**Исключения:** DNS может использовать TCP для:
- Больших ответов (>512 байт)
- Передачи зон (zone transfers)
- Когда UDP недоступен

---

## Скрипты для автоматизации

### Скрипт setup_ssh.sh
```bash
#!/bin/bash

echo "Создание Dockerfile для SSH сервера..."
cat > Dockerfile << 'EOF'
FROM ubuntu:22.04

RUN apt-get update && apt-get install -y \
    openssh-server \
    sudo \
    curl \
    && rm -rf /var/lib/apt/lists/*

RUN useradd -m -s /bin/bash sshuser && \
    echo 'sshuser:password123' | chpasswd && \
    usermod -aG sudo sshuser

RUN mkdir /var/run/sshd && \
    sed -i 's/#Port 22/Port 2222/' /etc/ssh/sshd_config && \
    sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config && \
    sed -i 's/#PasswordAuthentication yes/PasswordAuthentication yes/' /etc/ssh/sshd_config && \
    sed -i 's/#PubkeyAuthentication yes/PubkeyAuthentication yes/' /etc/ssh/sshd_config

RUN mkdir -p /home/sshuser/.ssh && \
    chown sshuser:sshuser /home/sshuser/.ssh && \
    chmod 700 /home/sshuser/.ssh

EXPOSE 2222

CMD ["/usr/sbin/sshd", "-D"]
EOF

echo "Сборка Docker образа..."
docker build -t ubuntu-ssh .

echo "Запуск SSH сервера на порту 2222..."
docker run -d --name ssh-server -p 2222:2222 ubuntu-ssh

echo "Генерация SSH ключей..."
ssh-keygen -t rsa -b 4096 -f ~/.ssh/ubuntu_ssh_key -N ""

echo "Копирование публичного ключа в контейнер с помощью ssh-copy-id..."
ssh-copy-id -i ~/.ssh/ubuntu_ssh_key.pub -p 2222 sshuser@localhost

echo "Получение fingerprint сервера..."
ssh-keyscan -p 2222 localhost

echo "Тестирование подключения..."
ssh -i ~/.ssh/ubuntu_ssh_key -p 2222 sshuser@localhost "ls -la /home/sshuser"

echo "Настройка завершена!"
```

### Скрипт capture_dns.sh
```bash
#!/bin/bash

echo "Установка tshark (если не установлен)..."
if ! command -v tshark &> /dev/null; then
    brew install wireshark
fi

echo "Начало перехвата DNS трафика..."
echo "Выполните 'curl google.com' в другом терминале"
echo "Нажмите Ctrl+C для остановки перехвата"

sudo tshark -i en0 -f "port 53" -w dns_capture.pcap

echo "Анализ перехваченного трафика..."
tshark -r dns_capture.pcap -Y "dns" -T fields -e dns.qry.name -e dns.a

echo "Статистика DNS запросов:"
tshark -r dns_capture.pcap -Y "dns" -T fields -e dns.qry.name | sort | uniq -c
```

---

## Команды для выполнения заданий

### Задание 1 и 2:
```bash
# Сделать скрипты исполняемыми
chmod +x setup_ssh.sh

# Запустить настройку SSH
./setup_ssh.sh

# Подключиться к серверу
ssh -i ~/.ssh/ubuntu_ssh_key -p 2222 sshuser@localhost
```

### Задание 3:
```bash
# Сделать скрипт исполняемым
chmod +x capture_dns.sh

# Запустить перехват DNS
./capture_dns.sh
```

---

## Ожидаемые результаты

### Задание 1:
- Скриншот консоли с содержимым домашней директории виртуальной машины
- Fingerprint сервера в формате: `localhost:2222 ssh-rsa AAAAB3NzaC1yc2E...`

### Задание 2:
- Скриншот успешного подключения к виртуальной машине по порту 2222

### Задание 3:
- Скриншоты Wireshark с DNS трафиком
- Ответы на вопросы о DNS протоколе
