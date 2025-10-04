# Практическая работа по работе с curl, shell prompt и nginx

## Обзор заданий

В данной практической работе необходимо выполнить три задания:
1. Отправить запрос к веб-сайту при помощи утилиты curl
2. Добавить в shell prompt дополнительную информацию о погоде
3. Применить знания о простом случае настройки nginx

---

## Задание 1: Работа с curl

### Цель задания
Потренироваться отправлять запросы к веб-сайтам при помощи утилиты curl.

### Что нужно сделать
Обратитесь к сайту http://ya.ru при помощи утилиты curl. Какие cookies придут в ответе? Сделайте скриншот вывода утилиты и отметьте необходимые строки.

### Выполнение

```bash
# Отправляем запрос к ya.ru
curl -v http://ya.ru

# Альтернативный вариант с более подробным выводом
curl -I http://ya.ru

# Для просмотра только заголовков ответа
curl -D - http://ya.ru
```

### Что искать в ответе
В выводе команды `curl -v` найдите секцию с заголовками ответа, особенно:
- `Set-Cookie:` - строки, которые устанавливают cookies
- `Location:` - если есть редирект

### Команды для скриншота
```bash
# Основная команда для задания
curl -v http://ya.ru

# Дополнительная информация о cookies
curl -c cookies.txt -b cookies.txt http://ya.ru
cat cookies.txt
```

---

## Задание 2: Настройка shell prompt с погодой

### Цель задания
Научиться добавлять в shell prompt дополнительную информацию.

### Что нужно сделать
Сайт wttr.in — это ориентированный на консольный доступ сервис погоды. Сделайте так, чтобы в вашем prompt выводилась температура в том городе, в котором вы находитесь (название города должно быть написано транслитерацией).

### Выполнение

#### Шаг 1: Тестирование API wttr.in
```bash
# Проверяем работу API
curl wttr.in/Moscow

# Получаем только температуру
curl "wttr.in/Moscow?format=%t"

# Получаем скорость и направление ветра
curl "wttr.in/Moscow?format=%w"

# Получаем краткую информацию
curl "wttr.in/Moscow?format=%C+%t"
```

#### Шаг 2: Настройка переменной окружения
```bash
# Добавляем в ~/.bashrc функцию для получения температуры
echo 'get_weather() {
    local city="Moscow"  # Замените на ваш город
    local temp=$(curl -s "wttr.in/$city?format=%t" 2>/dev/null)
    if [ $? -eq 0 ] && [ -n "$temp" ]; then
        echo "$temp"
    else
        echo "N/A"
    fi
}' >> ~/.bashrc
```

#### Шаг 3: Настройка PS1
```bash
# Добавляем в ~/.bashrc настройку prompt
echo 'export PS1="\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\] \[\033[01;33m\]\$(get_weather)\[\033[00m\]$ "' >> ~/.bashrc

# Применяем изменения
source ~/.bashrc
```

#### Альтернативный вариант (более производительный)
```bash
# Создаем функцию с кэшированием
echo 'get_weather_cached() {
    local city="Moscow"
    local cache_file="/tmp/weather_cache"
    local cache_time=300  # 5 минут
    
    if [ -f "$cache_file" ] && [ $(($(date +%s) - $(stat -c %Y "$cache_file" 2>/dev/null || echo 0))) -lt $cache_time ]; then
        cat "$cache_file"
    else
        local temp=$(curl -s "wttr.in/$city?format=%t" 2>/dev/null)
        if [ $? -eq 0 ] && [ -n "$temp" ]; then
            echo "$temp" > "$cache_file"
            echo "$temp"
        else
            echo "N/A"
        fi
    fi
}' >> ~/.bashrc

# Настраиваем PS1 с кэшированной функцией
echo 'export PS1="\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\] \[\033[01;33m\]\$(get_weather_cached)\[\033[00m\]$ "' >> ~/.bashrc
```

### Проверка работы
```bash
# Перезагружаем конфигурацию
source ~/.bashrc

# Проверяем, что prompt изменился
echo "Текущий prompt: $PS1"

# Тестируем функцию отдельно
get_weather
```

### Что оценивается
Работоспособность решения.

### Как отправить задание на проверку
Пришлите скриншот с получившимся промптом, а также фрагмент .bashrc, описывающий PS1.

---

## Задание 3: Установка и настройка nginx

### Цель задания
Применить знания о простом случае настройки nginx, полученные в модуле.

### Что нужно сделать
Установите и запустите nginx. Воспользовавшись примером из четвёртого урока, создайте веб-страницу со словами Hello World или любым понравившимся вам текстом и опубликуйте её при помощи nginx.

### Выполнение в Docker контейнере

#### Шаг 0: Подготовка Docker контейнера
```bash
# Запускаем Ubuntu контейнер с проброшенными портами
docker run --rm -it -p 8080:80 ubuntu /bin/bash

# Обновляем пакеты
apt update

# Устанавливаем необходимые утилиты
apt install -y curl wget nano links
```

#### Шаг 1: Установка nginx
```bash
# Устанавливаем nginx (в Docker контейнере sudo не нужен)
apt install nginx -y

# Запускаем nginx вручную (в Docker нет systemd)
nginx

# Проверяем, что nginx запущен
ps aux | grep nginx

# Проверяем, что nginx слушает порт 80
netstat -tlnp | grep :80
```

#### Шаг 2: Проверка работы nginx
```bash
# Проверяем, что nginx запущен
ps aux | grep nginx

# Проверяем, что nginx слушает порт 80
netstat -tlnp | grep :80

# Тестируем доступность
curl localhost
```

#### Шаг 3: Создание веб-страницы
```bash
# Создаем директорию для нашего сайта
mkdir -p /var/www/hello

# Создаем HTML-файл
tee /var/www/hello/index.html > /dev/null <<EOF
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Hello World</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            text-align: center;
            margin-top: 100px;
            background-color: #f0f0f0;
        }
        h1 {
            color: #333;
            font-size: 3em;
        }
        p {
            color: #666;
            font-size: 1.2em;
        }
    </style>
</head>
<body>
    <h1>Hello World!</h1>
    <p>Добро пожаловать на мой первый сайт с nginx!</p>
    <p>Время загрузки: <span id="time"></span></p>
    <script>
        document.getElementById('time').textContent = new Date().toLocaleString('ru-RU');
    </script>
</body>
</html>
EOF

# Устанавливаем права доступа
chown -R www-data:www-data /var/www/hello
chmod -R 755 /var/www/hello
```

#### Шаг 4: Настройка nginx
```bash
# Создаем конфигурацию сайта
tee /etc/nginx/sites-available/hello > /dev/null <<EOF
server {
    listen 80;
    server_name localhost;
    
    root /var/www/hello;
    index index.html;

    location / {
        try_files \$uri \$uri/ =404;
    }

    access_log /var/log/nginx/hello_access.log;
    error_log /var/log/nginx/hello_error.log;
}
EOF

# Включаем сайт
ln -s /etc/nginx/sites-available/hello /etc/nginx/sites-enabled/

# Удаляем дефолтный сайт (опционально)
rm -f /etc/nginx/sites-enabled/default

# Проверяем конфигурацию
nginx -t

# Перезагружаем nginx
nginx -s reload
```

#### Шаг 5: Проверка работы и создание скриншотов
```bash
# Проверяем доступность сайта
curl localhost

# Проверяем HTML-код страницы
cat /var/www/hello/index.html

# Просматриваем конфигурацию nginx
cat /etc/nginx/sites-available/hello

# Открываем сайт в текстовом браузере Links
links http://localhost

# Проверяем логи
tail /var/log/nginx/access.log
tail /var/log/nginx/error.log
```

#### Шаг 6: Доступ к сайту с хоста
После запуска контейнера с проброшенным портом, сайт будет доступен по адресу:
- **http://localhost:8080** - с вашего основного компьютера
- **http://localhost** - внутри контейнера

### Альтернативный вариант с простой страницей
```bash
# Создаем простую страницу
tee /var/www/hello/index.html > /dev/null <<EOF
<html>
<head>
    <title>Hello World</title>
</head>
<body>
    <h1>Hello World!</h1>
    <p>Это мой первый сайт на nginx!</p>
</body>
</html>
EOF
```

### Команды для скриншотов в Docker
```bash
# 1. Скриншот запуска контейнера
docker run --rm -it -p 8080:80 ubuntu /bin/bash

# 2. Скриншот установки nginx
apt install nginx -y

# 3. Скриншот запуска nginx
nginx
ps aux | grep nginx

# 4. Скриншот создания HTML-файла
cat /var/www/hello/index.html

# 5. Скриншот конфигурации nginx
cat /etc/nginx/sites-available/hello

# 6. Скриншот работы сайта в Links
links http://localhost

# 7. Скриншот доступа с хоста (в браузере)
# Откройте http://localhost:8080 в браузере на основном компьютере
```

### Что оценивается
Веб-сервер работает, корректно отображает созданную веб-страницу.

### Как отправить задание на проверку
Пришлите скриншот с открытой в браузере Links страницей, конфигурационный файл nginx, скриншот HTML-документа.

---

## Полезные команды для всех заданий

### Для задания 1 (curl)
```bash
# Основные опции curl
curl -v http://ya.ru          # Подробный вывод
curl -I http://ya.ru          # Только заголовки
curl -D cookies.txt http://ya.ru  # Сохранить cookies
curl -L http://ya.ru          # Следовать редиректам
```

### Для задания 2 (prompt)
```bash
# Просмотр текущего prompt
echo $PS1

# Сброс prompt к дефолтному
export PS1="\u@\h:\w$ "

# Просмотр файла конфигурации
cat ~/.bashrc | grep PS1
```

### Для задания 3 (nginx в Docker)
```bash
# Управление nginx в Docker (без systemd)
nginx                         # Запуск
nginx -s stop                 # Остановка
nginx -s reload               # Перезагрузка конфигурации
nginx -s quit                 # Корректное завершение
ps aux | grep nginx           # Проверка статуса

# Просмотр конфигурации
nginx -t                      # Проверка синтаксиса
nginx -T                      # Показать полную конфигурацию

# Логи
tail -f /var/log/nginx/access.log
tail -f /var/log/nginx/error.log

# Работа с Docker
docker run --rm -it -p 8080:80 ubuntu /bin/bash  # Запуск контейнера
exit                          # Выход из контейнера
```

## Порядок выполнения

1. **Задание 1**: Выполните команду `curl -v http://ya.ru` и найдите cookies в ответе
2. **Задание 2**: Настройте prompt с погодой, добавив функцию в `.bashrc`
3. **Задание 3**: 
   - Запустите Docker контейнер с проброшенным портом
   - Установите nginx в контейнере
   - Создайте HTML-страницу
   - Настройте виртуальный хост
   - Проверьте работу через Links и браузер хоста

## Требования к скриншотам

- **Задание 1**: Скриншот вывода `curl -v http://ya.ru` с отмеченными строками cookies
- **Задание 2**: Скриншот терминала с новым prompt, показывающим температуру
- **Задание 3**: 
  - Скриншот терминала с установкой nginx в Docker
  - Скриншот HTML-кода страницы (`cat /var/www/hello/index.html`)
  - Скриншот конфигурации nginx (`cat /etc/nginx/sites-available/hello`)
  - Скриншот работы сайта в Links (`links http://localhost`)
  - Скриншот браузера с открытой страницей по адресу http://localhost:8080
