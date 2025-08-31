# BankAPI

**BankAPI** — REST API для взаимодействия с банковской системой.  
С помощью API можно управлять счетами и картами, просматривать историю операций, выполнять переводы и платежи, а также работать с кредитами и автоплатежами.  

Документация подготовлена в формате **OpenAPI 3.0**.  
Файл спецификации: [`openapi.yaml`](./openapi.yaml)

---

## Быстрый старт

### Получение токена доступа
BankAPI использует **JWT Bearer Token** для авторизации.  

**Запрос:**
```bash
POST /auth/token
Content-Type: application/x-www-form-urlencoded
```

**Пример запроса (cURL):**
```bash
curl -X POST https://api.superbank.ru/v1/auth/token \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=password&username=user@example.com&password=MyStrongPass123!"
```

**Пример ответа:**
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI...",
  "token_type": "bearer",
  "expires_in": 3600,
  "refresh_token": "d3f4ultRefr3shT0ken"
}
```

Все защищённые запросы выполняются с заголовком:
```makefile
Authorization: Bearer <access_token>
```

### Служебные эндпоинты

`GET /health` — проверка состояния сервиса

Ответ:
```json
{ 
	"status": "OK" 
}
```

`GET /info` — информация о банке и версии API

Ответ:
```json
{
  "name": "SuperBank",
  "version": "1.0.0",
  "timestamp": "2025-08-30T12:34:56Z"
}
```

### Работа с клиентом

#### Профиль

`GET /my/profile`

Возвращает ФИО, контакты и паспортные данные.

#### Счета

`GET /my/accounts`

Пример ответа:
```json
{
  "accounts": [
    {
      "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
      "number": "40817810099910004312",
      "balance": { "value": 15430.45, "currency": "RUB" },
      "currency": "RUB",
      "status": "ACTIVE"
    }
  ]
}

```

### История операций

`GET /my/accounts/{accountId}/transactions`

Параметры фильтрации:

* `fromDate` — начальная дата (YYYY-MM-DD)

* `toDate` — конечная дата (YYYY-MM-DD)

* `limit / offset` — пагинация

Пример ответа:
```json
{
  "transactions": [
    {
      "id": "tx123456",
      "date": "2025-08-01T10:15:00Z",
      "amount": { "value": -1500.00, "currency": "RUB" },
      "description": "Оплата мобильной связи"
    },
    {
      "id": "tx123457",
      "date": "2025-08-02T09:00:00Z",
      "amount": { "value": 20000.00, "currency": "RUB" },
      "description": "Зачисление зарплаты"
    }
  ]
}

```

### Переводы

Перевод внутри банка

`POST /transfer/within-bank`

Запрос:
```json
{
  "fromAccount": "40817810099910004312",
  "toAccount": "40817810011120005432",
  "amount": { "value": 5000.00, "currency": "RUB" },
  "description": "Перевод другу"
}

```

Ответ:
```json
{
  "transactionId": "tx999888",
  "status": "PENDING",
  "confirmationRequired": true
}

```

### Валюты и курсы

`GET /public/currencies`

Ответ:
```json
{
  "currencies": [
    { "code": "USD", "buy": 94.50, "sell": 96.00 },
    { "code": "EUR", "buy": 102.30, "sell": 104.10 }
  ]
}

```

### Инфраструктура

* `GET /public/branches` — список отделений

* `GET /public/atms` — список банкоматов

Пример ответа (банкомат):
```json
{
  "atms": [
    {
      "id": "atm123",
      "address": "г. Москва, ул. Ленина, 10",
      "supportsCashIn": true,
      "status": "ONLINE"
    }
  ]
}

```

### Структура данных

#### Account

```yaml
id: string (uuid)
number: string (номер счёта)
balance: { value: number, currency: string }
currency: string
status: enum [ACTIVE, BLOCKED, CLOSED]
```

#### Amount

```yaml
value: number
currency: string
```

#### Transaction

```yaml
id: string
date: string (ISO 8601)
amount: Amount
description: string
```

### Инструменты

* `Swagger Editor` — для просмотра спецификации

* `Redoc` — для красивого рендеринга документации

* `Postman` — для тестирования запросов