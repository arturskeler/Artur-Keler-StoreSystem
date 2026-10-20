# BUG-002: Сервер возвращает 500 при смене email на уже занятый (PUT /users/edit/{id})

**Severity:** Major | **Priority:** High | **Статус:** Open
**Версия:** Release 1 (Authorization API)
**Воспроизводимость:** стабильная (воспроизводится при каждом запуске)

## 📌 Описание
При редактировании пользователя через `PUT /users/edit/{id}` с email, который уже
принадлежит другому пользователю, сервер отвечает `500 Internal Server Error`
вместо понятной ошибки 4xx.

Для сравнения: при регистрации с занятым email сервер корректно отвечает
`400` с телом `{"detail": "Email already registered"}`. На редактировании
аналогичная проверка отсутствует.

Предположительная причина: нарушение уникальности email в БД не обрабатывается
(необходимо подтвердить логом сервера).

## 🛠 Шаги воспроизведения (Steps to Reproduce)
Предусловие: стенд запущен, получен токен суперадмина. Зарегистрированы два
обычных пользователя:
- пользователь A: `a_user@mail.ru`
- пользователь B: `b_user@mail.ru` (его `id` нужен для запроса)

1. Отправить запрос:
   - `PUT http://localhost:8000/users/edit/{id пользователя B}`
   - Headers: `Authorization: Bearer <токен суперадмина>`, `Content-Type: application/json`
   - Body: `{"name": "Test User", "email": "a_user@mail.ru"}`

## ✅ Ожидаемый результат (Expected Result)
Ответ 4xx с понятным сообщением об ошибке (в ТЗ код не указан; ориентир: 400, 409
или 422). Email пользователя B остаётся прежним.

## ❌ Фактический результат (Actual Result)
`500 Internal Server Error`, тело ответа: `Internal Server Error`.

## 🔍 Окружение
- **Среда:** Docker / FastAPI / PostgreSQL
- **Версия:** Release 1 (Authorization API)

## 📎 Доказательства
- Скриншот запроса и ответа: `evidence/BUG-002-edit-500.png`

## 🧾 Лог сервера (docker logs python_app)
  File "/usr/local/lib/python3.10/site-packages/starlette/middleware/errors.py", line 162, in __call__
    await self.app(scope, receive, _send)
  File "/usr/local/lib/python3.10/site-packages/starlette/middleware/cors.py", line 84, in __call__
    await self.app(scope, receive, send)
  File "/usr/local/lib/python3.10/site-packages/starlette/middleware/exceptions.py", line 79, in __call__
    raise exc
  File "/usr/local/lib/python3.10/site-packages/starlette/middleware/exceptions.py", line 68, in __call__
    await self.app(scope, receive, sender)
  File "/usr/local/lib/python3.10/site-packages/fastapi/middleware/asyncexitstack.py", line 21, in __call__
    raise e
  File "/usr/local/lib/python3.10/site-packages/fastapi/middleware/asyncexitstack.py", line 18, in __call__
    await self.app(scope, receive, send)
  File "/usr/local/lib/python3.10/site-packages/starlette/routing.py", line 718, in __call__
    await route.handle(scope, receive, send)
  File "/usr/local/lib/python3.10/site-packages/starlette/routing.py", line 276, in handle
    await self.app(scope, receive, send)
  File "/usr/local/lib/python3.10/site-packages/starlette/routing.py", line 66, in app
    response = await func(request)
  File "/usr/local/lib/python3.10/site-packages/fastapi/routing.py", line 237, in app
    raw_response = await run_endpoint_function(
  File "/usr/local/lib/python3.10/site-packages/fastapi/routing.py", line 165, in run_endpoint_function
    return await run_in_threadpool(dependant.call, **values)
  File "/usr/local/lib/python3.10/site-packages/starlette/concurrency.py", line 41, in run_in_threadpool
    return await anyio.to_thread.run_sync(func, *args)
  File "/usr/local/lib/python3.10/site-packages/anyio/to_thread.py", line 33, in run_sync
    return await get_asynclib().run_sync_in_worker_thread(
  File "/usr/local/lib/python3.10/site-packages/anyio/_backends/_asyncio.py", line 877, in run_sync_in_worker_thread
    return await future
  File "/usr/local/lib/python3.10/site-packages/anyio/_backends/_asyncio.py", line 807, in run
    result = context.run(func, *args)
  File "/app/app/routes.py", line 134, in edit_user
    user = crud.edit_user(db, user_id, form_data)
  File "/app/app/crud.py", line 75, in edit_user
    db.commit()
  File "/usr/local/lib/python3.10/site-packages/sqlalchemy/orm/session.py", line 1451, in commit
    self._transaction.commit(_to_root=self.future)
  File "/usr/local/lib/python3.10/site-packages/sqlalchemy/orm/session.py", line 829, in commit
    self._prepare_impl()
  File "/usr/local/lib/python3.10/site-packages/sqlalchemy/orm/session.py", line 808, in _prepare_impl
    self.session.flush()
  File "/usr/local/lib/python3.10/site-packages/sqlalchemy/orm/session.py", line 3444, in flush
    self._flush(objects)
  File "/usr/local/lib/python3.10/site-packages/sqlalchemy/orm/session.py", line 3583, in _flush
    with util.safe_reraise():
  File "/usr/local/lib/python3.10/site-packages/sqlalchemy/util/langhelpers.py", line 70, in __exit__
    compat.raise_(
  File "/usr/local/lib/python3.10/site-packages/sqlalchemy/util/compat.py", line 211, in raise_
    raise exception
  File "/usr/local/lib/python3.10/site-packages/sqlalchemy/orm/session.py", line 3544, in _flush
    flush_context.execute()
  File "/usr/local/lib/python3.10/site-packages/sqlalchemy/orm/unitofwork.py", line 456, in execute
    rec.execute(self)
  File "/usr/local/lib/python3.10/site-packages/sqlalchemy/orm/unitofwork.py", line 630, in execute
    util.preloaded.orm_persistence.save_obj(
  File "/usr/local/lib/python3.10/site-packages/sqlalchemy/orm/persistence.py", line 237, in save_obj
    _emit_update_statements(
  File "/usr/local/lib/python3.10/site-packages/sqlalchemy/orm/persistence.py", line 1001, in _emit_update_statements
    c = connection._execute_20(
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
sqlalchemy.exc.IntegrityError: (psycopg2.errors.UniqueViolation) duplicate key value violates unique constraint "ix_users_email"
DETAIL:  Key (email)=(testuser1@example.com) already exists.

[SQL: UPDATE users SET name=%(name)s, email=%(email)s WHERE users.id = %(users_id)s]
[parameters: {'name': 'Test', 'email': 'testuser1@example.com', 'users_id': UUID('96801276-728b-465a-838d-f6fe1a3bc68b')}]
(Background on this error at: https://sqlalche.me/e/14/gkpj)

