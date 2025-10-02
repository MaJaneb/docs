# lab1»

Проект реализован на FastAPI с использованием SQLModel, JWT-аутентификации и Alembic для миграций. Основные сущности: пользователи, книги, жанры и запросы на обмен.


# Аутентификация
Используется OAuth2 Password Flow (Bearer Token). Токен создаётся при логине и проверяется в защищённых эндпоинтах.

Хеширование пароля: get_password_hash — генерация хеша пароля через bcrypt.

Проверка пароля: verify_password — проверка введённого пароля с хешем.

Генерация JWT: create_access_token — формирование токена с полем sub и сроком действия.

Получение пользователя: get_current_user — декодирование токена и поиск пользователя по email.

```python
from fastapi.security import OAuth2PasswordBearer
from jose import jwt

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="/user/login")

def get_current_user(token: str = Depends(oauth2_scheme)) -> User:
    payload = jwt.decode(token, settings.SECRET_KEY, algorithms=[settings.ALGORITHM])
    email = payload.get("sub")
    # ... получение пользователя из БД ...
```


# Модели данных (кратко)
- User: id, username, email, hashed_password, created_at; связи с книгами и обменами.

- Book: id, user_id, title, author, description, status (Доступна/Обменена), created_at; связи с жанрами.

- Genre: id, name; многие-ко-многим с Book через BookGenre.

- ExchangeRequest: id, sender_id, receiver_id, sender_book_id, requested_book_id, status (Рассматривается/Принято/Отклонено/Завершено), created_at.


# Эндпоинты

## Пользователи (/user)
POST /register — регистрация. Тело: username, email, password. Ответ: UserRead.

POST /login — вход (OAuth2PasswordRequestForm: username=email, password). Ответ: Token с access_token.

GET /users_get — список пользователей (Bearer). Ответ: List[UserRead].

GET /user_get_{user_id} — получить пользователя по id (Bearer). Ответ: UserRead.

PATCH /user_update_{user_id} — обновить username/email (Bearer). Тело: UserUpdate. Ответ: UserRead.

DELETE /delete_{user_id} — удалить пользователя (Bearer). Ответ: { "ok": true }.


## Книги (/book)
POST /book_create — создать книгу. Тело: BookCreate (title, author, description, status, genre_ids?). Ответ: BookResponse (включая genres).

PATCH /book_patch/{book_id} — обновить книгу, включая замену жанров. Тело: BookUpdate. Ответ: BookResponse.

GET /books_get — список книг с жанрами. Ответ: List[BookResponse].

GET /book_get/{book_id} — получить книгу по id. Ответ: BookResponse.

DELETE /book_delete/{book_id} — удалить книгу. Ответ: { "ok": true }.


## Жанры (/genre)
GET /genres_get — список жанров. Ответ: List[GenreResponse].

POST /genre_post — создать жанр. Тело: GenreCreateAndUpdate. Ответ: GenreResponse.

PATCH /genre_patch{genre_id} — обновить жанр. Тело: GenreCreateAndUpdate. Ответ: GenreResponse.

GET /genre_get{genre_id} — получить жанр. Ответ: GenreResponse.

DELETE /genre_delete_{genre_id} — удалить жанр. Ответ: { "ok": true }.


## Запросы на обмен (/exchange_request)
POST /exchange_request_post — создать запрос на обмен. Тело: ExchangeRequestCreate. Ответ: ExchangeRequestResponse.

PATCH /exchange_request_post{exchange_request_id} — обновить статус запроса. Тело: ExchangeRequestUpdateStatus. Ответ: ExchangeRequestResponse.

GET /exchange_requests_get — список запросов. Ответ: List[ExchangeRequestResponse].

GET /exchange_request_get{exchange_request_id} — получить один запрос. Ответ: ExchangeRequestResponse.

DELETE /exchange_request_delete{exchange_request_id} — удалить запрос. Ответ: { "ok": true }.


## Обмены (/exchange)
Заглушки (планируется реализация):

GET /exchange_get — список обменов.

POST /exchange_create — создание обмена.


# Схемы (выдержки)
```python
class BookCreate(BaseModel):
    title: str
    author: str
    description: str
    status: Status = Status.available
    genre_ids: Optional[List[int]] = None

class ExchangeRequestCreate(BaseModel):
    sender_id: int
    receiver_id: int
    sender_book_id: int
    requested_book_id: int
    status: StatusExchange
```


# Запуск и миграции
— Настройте переменные окружения (секрет JWT, БД) в конфигурации.

— Примените миграции Alembic (alembic upgrade head).

— Запустите приложение (uvicorn main:app --reload).


# Alembic
Alembic используется для версионирования схемы БД: создание, изменение, откат миграций.


