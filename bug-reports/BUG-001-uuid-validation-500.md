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
TODO: вставить фрагмент вокруг `Traceback` после воспроизведения.
