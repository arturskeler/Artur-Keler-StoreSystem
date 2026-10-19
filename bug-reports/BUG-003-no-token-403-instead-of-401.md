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
Проверено на двух эндпоинтах (`GET /users/`, `PUT /users/edit/{id}`). Вероятно,
затрагивает все защищённые эндпоинты (promote, delete): проверить отдельно.
