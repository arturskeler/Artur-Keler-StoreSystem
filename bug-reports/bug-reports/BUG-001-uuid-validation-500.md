# BUG-001: Сервер возвращает 500 вместо 422 при невалидном UUID в пути запроса

**Severity:** Major | **Priority:** High | **Статус:** Open
**Версия:** Release 1 (Authorization API)
**Воспроизводимость:** стабильная (воспроизводится при каждом запуске)

## 📌 Описание
При передаче в путь значения, которое не является UUID, в запросах
`PUT /users/edit/{id}` и `DELETE /users/delete/{id}` сервер отвечает
`500 Internal Server Error` вместо `422 Unprocessable Entity`.
Ответ 5xx на некорректный ввод клиента является дефектом при любой спецификации.

Для сравнения: валидный, но несуществующий UUID обрабатывается корректно
(`DELETE` и `PUT /users/promote/{id}` возвращают 404).

## 🛠 Шаги воспроизведения (Steps to Reproduce)
Предусловие: стенд запущен (`docker compose up -d`), получен токен суперадмина
через `POST /login/`.

**Вариант 1 (PUT):**
1. Отправить запрос:
   - `PUT http://localhost:8000/users/edit/not-a-valid-uuid`
   - Headers: `Authorization: Bearer <токен суперадмина>`, `Content-Type: application/json`
   - Body: `{"name": "Test User", "email": "test@mail.ru"}`

**Вариант 2 (DELETE):**
1. Отправить запрос:
   - `DELETE http://localhost:8000/users/delete/invalid-uuid-123`
   - Headers: `Authorization: Bearer <токен суперадмина>`

## ✅ Ожидаемый результат (Expected Result)
`422 Unprocessable Entity` с описанием ошибки валидации формата данных.

## ❌ Фактический результат (Actual Result)
`500 Internal Server Error`, тело ответа: `Internal Server Error`.

## 🔍 Окружение
- **Среда:** Docker / FastAPI / PostgreSQL
- **Версия:** Release 1 (Authorization API)

## 📎 Доказательства
- Скриншот PUT: `evidence/BUG-001-edit-500.png`
- Скриншот DELETE: `evidence/BUG-001-delete-500.png`

## 🧾 Лог сервера (docker logs python_app)
  File "/usr/local/lib/python3.10/site-packages/sqlalchemy/orm/query.py", line 2916, in _iter

    result = self.session.execute(

  File "/usr/local/lib/python3.10/site-packages/sqlalchemy/orm/session.py", line 1714, in execute

    result = conn._execute_20(statement, params or {}, execution_options)

  File "/usr/local/lib/python3.10/site-packages/sqlalchemy/engine/base.py", line 1705, in _execute_20

    return meth(self, args_10style, kwargs_10style, execution_options)

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

sqlalchemy.exc.DataError: (psycopg2.errors.InvalidTextRepresentation) invalid input syntax for type uuid: "invalid-uuid-123"

LINE 3: WHERE users.id = 'invalid-uuid-123' 
