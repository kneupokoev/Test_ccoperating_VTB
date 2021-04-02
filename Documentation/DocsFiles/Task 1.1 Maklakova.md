# IB.GetLoyaltyInfoOperations #
58.  url: /rest/private/loyalty/info/operations; method: GET.
59.  Назначение сервиса: получение истории операций по бонусному счету.

## Структура запроса и ответа ##
60. Структура запроса:

| Название параметра | Тип  | Описание/Значение | Обязательность |
|:------------- |:-----------:| :-------------:| :--------------:|
| operationCount     | Integer |    Количество операций по бонусному сче-ту/заданное по умолчанию | Optional |
61. Структура ответа GetLoyaltyHistoryResponseIBMo (extends AbstactResponseMo):

| Название параметра | Тип  | Описание/Значение | Обязательность |
|:------------- |:------------:| :---------------:| :--------------:|
| loyaltyContractId     | Long |    Идентификатор бонусного контракта | Mandatory |
| loyaltyOperation     | List [loyaltyOperationIBMo] |    Информация об операциях по бонусному счету | Mandatory |

62. Структура loyaltyOperationIBMo:

| Название параметра | Тип  | Описание/Значение | Обязательность |
|:------------- |:------------:| :---------------:| :--------------:|
| loyaltyProgramId   | Integer |  Идентификатор бонусной программы | Mandatory |
| operationStatus  | String |  Статус операции: начисление, списание | Mandatory |
| describtion | String |  Описание операции | Mandatory |
| operationAmountM   | MoneyMo |  Сумма операции| Mandatory |
| bonusAmountM | MoneyMo |  Сумма бонусов | Mandatory |
| operationDate  | Date |  Дата совершения операции | Mandatory |

## Основной сценарий (MC) ##
63.	[Шаг1] myGEMINI Core получает и разбирает запрос к сервису GetLoyaltyInfo.
64.	[Шаг2] myGEMINI Core проверяет параметры запроса на соответствие прото-колу (Неуспешная проверка: EC-BadRequest[1])
65.	[Шаг3] myGEMINI Core проверяет, что роль текущего пользователя преду-сматривает права на операцию 3.71.01.00-02 (Неуспешная проверка: EC-Forbidden[1])
66.	[Шаг4] В зависимости от значения полученного параметра operationCount myGEMINI Core возвращает:
       1. При заданном параметре operationCount заданное количество операций по бонусному счету.
       2. При незаданном параметре operationCount  количество операций по бонусному счету, установленное по умолчанию = 10.
67.	[Шаг5] myGEMINI Core формирует и отправляет GET запрос к мето-ду getLoyaltyHistory МкС LoyaltyHistory для получения истории операций по бонусному счету.
68.	Шаг6] myGEMINI Core проверяет, что ответ от метода getLoyaltyHistory микро-сервиса LoyaltyHistory содержит http status 200. (EC-Exception [1])
       1.	HTTP status 404: (EC-1)
69.	[Шаг7] myGEMINI Core разбирает полученный ответ от мкс LoyaltyHistory.
70.	[Шаг8] myGEMINI Core формирует ответ  согласно следующим правилам:

| Атрибут ответа | Значение  | 
|:------------- |:------------:| 
| loyaltyContractId – Идентификатор бонусного контракта в БД LoyaltyAccount.id |
| loyaltyOperation - Список с историей операций |


| Атрибут объекта | Значение  | 
|:------------- |:------------| 
| loyaltyProgramId | Значение LoyaltyAccount.loyaltyProgramId |
| operationStatus  | Значение LoyaltyHistory.operationStatus |
| describtion | Значение LoyaltyHistory.descriprion |
| operationAmountM:    |
| operationAmount | Значение LoyaltyHistory.bonusAmount |
| bonusCurrency  | Значение LoyaltyHistory.bonusCurrency| 
| operationDate  | Значение LoyaltyHistory.operationDate|

71.	 [Шаг9] myGEMINI Core возвращает ответ приложению.
72.	 Сценарий завершается.

### _Таблица «История операций по бонусному счету»_ ###

73.	Наименование таблицы - LoyaltyHistory
74.	Структура таблицы представлена ниже:

| Название параметра | Тип  | Описание/Значение | 
|:------------- |:------------| :---------------| 
| loyaltyContractId  | Integer |  Идентификатор бонусного контракта | 
| operationStatus  | String |  Статус операции: начисление, списание | Mandatory |
| describtion | String |  Описание операции | Mandatory |
| operationAmountM   | MoneyMo |  Сумма операции | Mandatory |
| bonusAmountM | MoneyMo |  Сумма бонусов | Mandatory |
| operationDate  | Date |  Дата совершения операции | 
| describtion | String |  Описание операции | 
| operationAmount  | String |  Сумма операции | 
| operationCurrency  | String | Валюта операции | 
| bonusAmount  | String |  Сумма бонусов | 
| bonusCurrency  | String | Валюта бонусов |