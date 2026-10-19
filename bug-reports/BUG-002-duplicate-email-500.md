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
TODO: вставить фрагмент вокруг `Traceback`. Если там `IntegrityError` или
`UniqueViolation`, причина подтверждена.
