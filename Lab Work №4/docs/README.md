# **Лабораторная работа №4**

**Тема:** Проектирование REST API

**Цель:** Получить опыт проектирования программного интерфейса.

---

## Принятые проектные решения

### 1. Архитектурное решение по разделению по ролям пользователей

API логически разделено по ролям:
- API студентов
- API преподавателей
- API администратора
- API авторизации

Каждый набор эндпоинтов вынесен в отдельный router (группа эндпоинтов), что позволяет разделять ответственности, упрощает контроль доступа.

### 2. Использование стандартных HTTP-методов

Для работы с ресурсами используются стандартные HTTP-методы:
- `GET` - получение данных (задания, результаты проверки)
- `POST` - создание ресурсов (отправка работы)
- `PUT` - обновление ресурсов (пересдача, обновление статуса)
- `DELETE` - удаление ресурсов (отзыв отправки до дедлайна)


### 3. Единая модель URL для ресурсов

API проектируется на основе сущностей:
- `/student/tasks`
- `/student/submissions`
- `/student/submissions/{submission_id}`

URL не содержит глаголов и описывает ресурсы - позволяет удобно масштабировать API.


### 4. Использование JWT для идентификации пользователя

Аутентификация реализована с помощью JWT access token, токен содержит:
- `user_id`
- `role`
- срок действия (`exp`)

Позволяет достичь единого механизма авторизации для всех API.

### 5. Асинхронная обработка ресурсоёмких операций

Загрузка и регистрация работы выполняются синхронно, а анализ содержимого и генерация отчёта предполагаются как асинхронные операции (внешний сервис).

Это позволяет избежать "зависания" API на время их выполенения.


### 6. Разделение API-слоя, бизнес-логики и ORM

Контроллеры (routers) выполняют:
- валидацию входных данных;
- проверку прав доступа;
- передачу управления сервисам.

Бизнес-логика, управление данными вынесены в отдельные сервисы.

### 7. Использование схем (Pydantic) для входных и выходных данных

Для каждого эндпоинта определены Pydantic-схемы:
- входные (Create / Update)
- выходные (Response)

Автоматическая валидация данных.

### 8. Явная обработка ошибок и кодов ответа

API использует стандартные HTTP-коды:
- `200` — успешные операции
- `400` — ошибка запроса
- `401` — ошибка аутентификации
- `403` — недостаточно прав
- `404` — ресурс не найден
- `500` — ошибка на сервере

---

## Описание реализуемого API и тестирование 

### 1. Регистрация пользователя

**API:** `/auth/register`
**Метод:** `POST`

### Пример запроса (CURL)

```
postman request POST 'http://127.0.0.1:8000/api/v1/auth/register' \
  --header 'Content-Type: application/json' \
  --body '{
  "email": "student@mail.com",
  "password": "123456",
  "admin_key": null,
  "full_name": "Student 1"
}'
```

### Ответ (200) - успешная регистрация

```json
{
    "message": "User registered successfully",
    "user_id": 4
}
```

### Ответ (403) - пользователь с таким email уже существует

```json
{
    "detail": "User already exists"
}
```


---

### 2. Аутентификация пользователя

**API:** `/auth/login`
**Метод:** `POST`

### Пример запроса (CURL)

```
postman request POST 'http://127.0.0.1:8000/api/v1/auth/login' \
  --header 'Content-Type: application/json' \
  --body '{
  "email": "student@mail.com",
  "password": "123456"
}'
```

### Ответ (200) - успешный вход

```json
{
    "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX2lkIjo0LCJyb2xlIjoiVVNFUiIsImV4cCI6MTc3MDIzNDA2Nn0.-d4d1qVRUqyTzA8M8I55wi_uKjRtRjztH_GGzBaTLX8",
    "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX2lkIjo0LCJleHAiOjE3NzAzMTk1NjZ9.NnOWAPlTU_NyPvJi_VzrVrYQNwUZI01lE_NQRvlCai4"
}
```

### Ответ (403) - неверный пароль

```json
{
    "detail": "Wrong password"
}
```

---

### 3. Обновление access-токена

**API:** `/auth/refresh`
**Метод:** `POST`

### Пример запроса (CURL)

```
postman request POST 'http://127.0.0.1:8000/api/v1/auth/refresh' \
  --header 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX2lkIjo0LCJyb2xlIjoiVVNFUiIsImV4cCI6MTc3MDIzNDA2Nn0.-d4d1qVRUqyTzA8M8I55wi_uKjRtRjztH_GGzBaTLX8' \
  --body '' \
  --auth-bearer-token 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX2lkIjo0LCJyb2xlIjoiVVNFUiIsImV4cCI6MTc3MDIzNDA2Nn0.-d4d1qVRUqyTzA8M8I55wi_uKjRtRjztH_GGzBaTLX8'
```

### Ответ (200) - успешное обновление

```json
{
    "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX2lkIjo0LCJyb2xlIjoiVVNFUiIsImV4cCI6MTc3MDIzNDE1NX0.w2l-4L8inLtkQ_hCY3jS8S1RzPF98RBvGvIC1YDLCeI"
}
```

### Ответ (403) - не авторизован

```json
{
    "detail": "Not authenticated"
}
```


---

### 4. Загрузка работы студентом

**API:** `/student/submissions`
**Метод:** `POST`

### Параметры

* `task_id` — ID задания
* `file` — файл работы

### Пример запроса (CURL)

```
postman request POST 'http://127.0.0.1:8000/api/v1/student/submissions?task_id=1&file=%22test.py%22' \
  --header 'Content-Type: application/json' \
  --header 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX2lkIjo0LCJyb2xlIjoiVVNFUiIsImV4cCI6MTc3MDIzNDA2Nn0.-d4d1qVRUqyTzA8M8I55wi_uKjRtRjztH_GGzBaTLX8' \
  --body '{
    "file": "test.py"
}' \
  --auth-bearer-token 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX2lkIjo0LCJyb2xlIjoiVVNFUiIsImV4cCI6MTc3MDIzNDA2Nn0.-d4d1qVRUqyTzA8M8I55wi_uKjRtRjztH_GGzBaTLX8'
```

### Ответ (200)

```
{
    "id": 4,
    "role": "USER",
    "task_id": 1,
    "file_name": "\"test.py\"",
    "status": "Submitted successfully"
}
```

### Ответ (404) - задание не найдено

```
{
    "detail": "Task not found"
}
```

---

### 5. Получение результата проверки

**API:** `/student/submissions/{submission_id}`
**Метод:** `GET`

### Пример запроса (CURL)

```
postman request 'http://127.0.0.1:8000/api/v1/student/submissions/4' \
  --header 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX2lkIjo0LCJyb2xlIjoiVVNFUiIsImV4cCI6MTc3MDIzNDA2Nn0.-d4d1qVRUqyTzA8M8I55wi_uKjRtRjztH_GGzBaTLX8' \
  --body '' \
  --auth-bearer-token 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX2lkIjo0LCJyb2xlIjoiVVNFUiIsImV4cCI6MTc3MDIzNDA2Nn0.-d4d1qVRUqyTzA8M8I55wi_uKjRtRjztH_GGzBaTLX8'
```

### Ответ (200)

```json
{
    "id": 4,
    "role": "USER",
    "email": "student@mail.com",
    "submission_id": 4,
    "result": "Some result..."
}
```

### Ответ (404) - результат проверки не найден

```
{
    "detail": "Submission not found or not ready yet"
}
```

---

### 6. Обновление роли пользователя

**API:** `/admin/update_user_role/{user_id}`
**Метод:** `PUT`

### Пример запроса

```
postman request PUT 'http://127.0.0.1:8000/api/v1/admin/update_user_role/2?new_role=%22ADMIN%22' \
  --header 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX2lkIjoyLCJyb2xlIjoiQURNSU4iLCJleHAiOjE3NzAyMzUxMTF9.6BV6pWZtphlAW0WXTp368zDe1eYREHrrZy-D_Sdqndc' \
  --body '' \
  --auth-bearer-token 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX2lkIjoyLCJyb2xlIjoiQURNSU4iLCJleHAiOjE3NzAyMzUxMTF9.6BV6pWZtphlAW0WXTp368zDe1eYREHrrZy-D_Sdqndc'
```

### Ответ (200)

```json
{
    "id": 2,
    "password": "$2b$12$mcdJ5UMKhRtFH/Gp/qKMEuFvnFdEksPS2gt0Jc7dEI4aoCw.dxN6i",
    "full_name": "Ярослав Тестович",
    "role_id": null,
    "last_entry_date": null,
    "email": "test",
    "role": null
}
```

### Ответ (403) - недостаточно прав

```
{
    "detail": "Forbidden - admin role required"
}
```
---

### 7. Получение списка всех пользователей

**API:** `/admin/users/`
**Метод:** `GET`


### Пример запроса

```
postman request 'http://127.0.0.1:8000/api/v1/admin/users/' \
  --header 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX2lkIjoyLCJyb2xlIjoiQURNSU4iLCJleHAiOjE3NzAyMzUxMTF9.6BV6pWZtphlAW0WXTp368zDe1eYREHrrZy-D_Sdqndc' \
  --auth-bearer-token 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX2lkIjoyLCJyb2xlIjoiQURNSU4iLCJleHAiOjE3NzAyMzUxMTF9.6BV6pWZtphlAW0WXTp368zDe1eYREHrrZy-D_Sdqndc'
```


### Ответ (200)

```json
[
    {
        "id": 3,
        "password": "$2b$12$2I6jyK/6eeTfiPzWH4n0yeuj5UWkdfGKsoPEXha7ucNmhFll8V9LW",
        "full_name": "Ярослав Студент",
        "role_id": 2,
        "last_entry_date": null,
        "email": "test1"
    },
    {
        "id": 4,
        "password": "$2b$12$EPvuGyJticd9D3DPRUT5Fe6zPujsza0V7idMzBa7YONYFQmhp.R5m",
        "full_name": "Student 1",
        "role_id": 2,
        "last_entry_date": null,
        "email": "student@mail.com"
    }
]
```

### Ответ (403) - недостаточно прав

```
{
    "detail": "Forbidden - admin role required"
}
```
