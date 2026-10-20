# BUG-003: Запрос без токена возвращает 403 вместо 401

**Severity:** Minor | **Priority:** Medium | **Статус:** Open
**Версия:** Release 1 (Authorization API)
**Воспроизводимость:** стабильная

## 📌 Описание
При обращении к защищённым эндпоинтам без заголовка `Authorization` сервер
отвечает `403 Forbidden` с телом `{"detail": "Not authenticated"}`.
Для отсутствующих учётных данных корректный ответ: `401 Unauthorized`.

Поведение непоследовательное: запрос с неверным токеном возвращает
`401` с телом `{"detail": "Invalid token"}`, а запрос без токена возвращает `403`.

## 🛠 Шаги воспроизведения (Steps to Reproduce)
Предусловие: стенд запущен.

**Вариант 1:**
1. `GET http://localhost:8000/users/` без заголовка `Authorization`.

**Вариант 2:**
1. `PUT http://localhost:8000/users/edit/{любой валидный UUID}`
   без заголовка `Authorization`, Body: `{"name": "Test User", "email": "test@mail.ru"}`.

## ✅ Ожидаемый результат (Expected Result)
`401 Unauthorized`. Источник: курс (страница «Как писать тесты для API»):
отсутствие токена должно приводить к 401. В ТЗ раздела 4-7 указано, что
токен передаётся через `Authorization: Bearer`.

## ❌ Фактический результат (Actual Result)
`403 Forbidden`, тело: `{"detail": "Not authenticated"}`.

## 🔍 Окружение
- **Среда:** Docker / FastAPI / PostgreSQL
- **Версия:** Release 1 (Authorization API)

## 📎 Доказательства
- Скриншоты: `evidence/BUG-003-get-users-403.png`, `evidence/BUG-003-edit-403.png`

## Примечание
  File "/usr/local/lib/python3.10/site-packages/sqlalchemy/sql/elements.py", line 334, in _execute_on_connection

    return connection._execute_clauseelement(

  File "/usr/local/lib/python3.10/site-packages/sqlalchemy/engine/base.py", line 1572, in _execute_clauseelement

    ret = self._execute_context(

  File "/usr/local/lib/python3.10/site-packages/sqlalchemy/engine/base.py", line 1943, in _execute_context

    self._handle_dbapi_exception(

  File "/usr/local/lib/python3.10/site-packages/sqlalchemy/engine/base.py", line 2124, in _handle_dbapi_exception

    util.raise_(

  File "/usr/local/lib/python3.10/site-packages/sqlalchemy/util/compat.py", line 211, in raise_

    raise exception

  File "/usr/local/lib/python3.10/site-packages/sqlalchemy/engine/base.py", line 1900, in _execute_context

    self.dialect.do_execute(

  File "/usr/local/lib/python3.10/site-packages/sqlalchemy/engine/default.py", line 736, in do_execute

    cursor.execute(statement, parameters)

sqlalchemy.exc.IntegrityError: (psycopg2.errors.UniqueViolation) duplicate key value violates unique constraint "ix_users_email"

DETAIL:  Key (email)=(testuser1@example.com) already exists.


[SQL: UPDATE users SET name=%(name)s, email=%(email)s WHERE users.id = %(users_id)s]

[parameters: {'name': 'Test', 'email': 'testuser1@example.com', 'users_id': UUID('96801276-728b-465a-838d-f6fe1a3bc68b')}]

(Background on this error at: https://sqlalche.me/e/14/gkpj)

INFO:     172.18.0.1:48526 - "GET /users/ HTTP/1.1" 401 Unauthorized

INFO:     172.18.0.1:45438 - "GET /users/ HTTP/1.1" 401 Unauthorized

INFO:     172.18.0.1:35046 - "GET /users/ HTTP/1.1" 401 Unauthorized

INFO:     172.18.0.1:41628 - "GET /users/ HTTP/1.1" 403 Forbidden

INFO:     172.18.0.1:43294 - "GET /users/ HTTP/1.1" 401 Unauthorized

INFO:     172.18.0.1:38206 - "GET /users/ HTTP/1.1" 403 Forbidden

INFO:     172.18.0.1:34350 - "GET /users/ HTTP/1.1" 401 Unauthorized

INFO:     172.18.0.1:57148 - "PUT /users/edit/96801276-728b-465a-838d-f6fe1a3bc68b HTTP/1.1" 401 Unauthorized

INFO:     172.18.0.1:41098 - "PUT /users/edit/96801276-728b-465a-838d-f6fe1a3bc68b HTTP/1.1" 403 Forbidden
