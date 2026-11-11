# Django Mailing Service

Учебный сервис email-рассылок на Django. Можно создать сообщение, выбрать получателей, отправить письма и посмотреть результат каждой попытки.

**Стек:** Python 3.14, Django, PostgreSQL, Redis, SMTP, Poetry.

## Что умеет

- Регистрация с подтверждением email и восстановление пароля.
- Управление получателями, сообщениями и рассылками. Пользователи работают со своими записями.
- Отправка писем через SMTP и статистика успешных и неудачных попыток.
- Отключение рассылок и блокировка пользователей для менеджера.
- Кэширование счётчиков главной страницы в Redis.

Рассылка запускается кнопкой. Даты начала и окончания задают период, в который её можно отправить.

## Запуск

Нужны Python 3.14, Poetry, PostgreSQL, Redis и SMTP-аккаунт. Команды ниже — для PowerShell.

```powershell
git clone https://github.com/VTokyoGhoulV/django-mailing-service.git
cd django-mailing-service
poetry install --no-root
Copy-Item .env_example .env
```

Создайте отдельную базу PostgreSQL. Например, через `psql` под административной учётной записью:

```sql
CREATE USER mailing_user WITH PASSWORD 'replace_with_database_password';
CREATE DATABASE mailing_service OWNER mailing_user;
```

В `.env` укажите параметры базы: `NAME`, `USER`, `PASSWORD`, `HOST`, `PORT`.
Для почты заполните `EMAIL_HOST`, `EMAIL_PORT`, `EMAIL_HOST_USER` и `EMAIL_HOST_PASSWORD`. В настройках включён TLS.

Запустите Redis по адресу `127.0.0.1:6379`. Приложение использует базу Redis № 1; адрес задан в `config/settings.py`.

```powershell
poetry run python manage.py migrate
poetry run python manage.py createsuperuser
poetry run python manage.py runserver
```

Откройте http://127.0.0.1:8000/. Административная панель — http://127.0.0.1:8000/admin/.

## Как отправить рассылку

1. Зарегистрируйтесь через `/users/register/` и подтвердите email.
2. Добавьте получателя и сообщение. Для проверки используйте свой почтовый адрес.
3. Создайте рассылку с датой начала не раньше текущего времени и датой окончания позже начала.
4. В период действия рассылки откройте её карточку и нажмите кнопку отправки.
5. Посмотрите результат на странице `/mailer/mailer_statistic/`.

Права менеджера назначаются через административную панель: `mailer.view_recipient`, `mailer.view_mailer`, `mailer.disable_mailer`, `auth.view_user` и `users.block_user`.

Проверить настройки Django можно командой `poetry run python manage.py check`. Автоматических тестов пока нет.

## Ограничения

- `Mailer.__str__` возвращает объект сообщения вместо строки. Это вызывает `TypeError` при строковом отображении рассылки.
- Отправка выполняется в текущем запросе. Планировщика и автоматических повторных попыток нет.
- Повторное нажатие кнопки может отправить письма тем же получателям ещё раз.
- Успешная попытка означает, что SMTP-сервер принял письмо; доставка и прочтение не отслеживаются.
- Email получателя должен быть уникальным для всего приложения.
- Настройки рассчитаны на локальный запуск: `DEBUG` включён, ключ Django и адрес Redis заданы в исходниках.
