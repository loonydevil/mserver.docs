# Карты

Карта - это сохраненный в mserver токен, с помощью которого пользователь кошелька может пополнять счет без ввода данных пластиковой карты.

### Поля

* `id` - уникальный идентификатор карты
* `state` - состояние карты, см. ниже
* `title` - название карты, в качестве названия сейчас используется маскированный номер карты
* `type` - бренд карты: Visa/MasterCard/...

### Состояния карты

| Код состояния карты   | Описание                                                                                     |
| :-------------------  |:---------------------------------------------------------------------------------------------|
| `created`             | Карта только что создана                                                                     |
| `pending`             | Карта сохраняется (ожидается уведомление от IPSP о успехе/неуспехе карточной транзакции      |
| `active`              | Карта сохранена, может быть использована для повторных платежей                              |
| `failed`              | Карта не сохранена (и уже не будет)                                                          |
| `used`                | Карта использована для однократного пополнения

## Создание карты

### Коды ошибок

* `active_cards_limit` - превышено максимально возможное количество активных карт

```shell
$ curl -u+79261111111:password -H 'Content-type:application/json' -X POST https://www.synq.ru/mserver2-dev/v1/cards
```

```json
{
  "meta" : {
    "code" : 200
  },
  "data" : {
    "id" : 62,
    "state" : "pending",
    "payment_page_url" : "https://test1.ipsp.com/frontend/endpoint?product_id=1721&desc=mserver2&payment_type=A&amount=1.00&currency=RUB&biller_client_id=1f95c7b9-74e5-4fd7-983d-c8d03d90347e&perspayee_expiry=0150&recur_freq=1&locale=ru&hash=cace0d7de544a25d2aa685ef12263a10655d9058",
    "payment_token": "0000012aed5ac313f64a708a65fd0e875f60bc1415634865334311dd49178aff439f4b42c6ae628854e71385dc87eb9ae5cb5633ce1ee7221668ec4e90a7a3dcd676ace6b298874e85cb52556b1265d1247dfb0e1c529fd11b0551219f39a6ef296298c1488f3b0ba5cf8d9f8c8af528ffe3aef3a59325e38d07ade2dadef479ca3f4522cc6285ca285e37a63df687ce8ec06a416d78d11fcd7"
  }
}
```

После создания карты следует перенаправить пользователя на платежную страницу по ссылке из поля `payment_page_url`. После ввода пользователем данных карты на платежной странице  и получения mserver уведомления от IPSP об успехе или неуспехе карточной транзакции карта перейдет в состояние `active` или `failed`.

## Загрузка карты

```shell
$ curl -u+79261111111:password -H 'Content-type:application/json' https://www.synq.ru/mserver2-dev/v1/cards/62
```

```json
{
  "meta" : {
    "code" : 200
  },
  "data" : {
    "id" : 62,
    "state" : "active",
    "title" : "541715******2399",
    "type" : "MasterCard"
  }
}
```

Как видно, карта успешно сохранена и может быть использована для пополнения счета кошелька.

## Получение списка карт

```shell
$ curl -u+79261111111:password -H 'Content-type:application/json' https://www.synq.ru/mserver2-dev/v1/cards?state=active,pending
```

```json
{
  "meta" : {
  "page" : {
      "total_elements" : 1
    },
    "code" : 200
  },
  "data" : [ {
    "id" : 62,
    "state" : "active",
    "title" : "541715******2399",
    "type" : "MasterCard"
  } ]
}
```

Опциональный параметр `state` позволяет фильтровать карты по состоянию.

## Удаление карты

```shell
$ сurl -u+79261111111:password -H 'Content-type:application/json' 
 -X DELETE https://www.synq.ru/mserver2-dev/v1/cards/62
```

```json
{
  "meta" : {
    "code" : 200
  }
}
```
## Уведомления об изменении статуса привязки карты

Для уведомления внешней системы об изменениях статуса карты mserver выполняет POST запрос на URL /cards/status. Уведомление передается в теле POST запроса в виде объекта карты сериализованного в JSON. Например:

```json
{
    "id" : 62,
    "state" : "active",
    "title" : "541715******2399",
    "type" : "MasterCard",
    "wallet" : {
     "phone" : "+792611111111"
    }
}
```

## Данные тестовых карт

| Номер карты      | Срок действия  | CVV2/CVC2 | 3D-Secure |
| :--------------  |:-------------- |---------: |:---------:|
| 5417150893587260 | 01/17          | 082       | &#x2713;  |
| 5417150396276825 | 01/17          | 789       | &#x2717;  |  
| 4652060573334999 | 01/17          | 067       | &#x2713;  |
| 4652060724922338 | 01/17          | 989       | &#x2717;  |

Пароль 3D-Secure: 123456.