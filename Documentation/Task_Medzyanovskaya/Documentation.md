# Лояльность



#### Тип документа
### Функциональная спецификация

### СОДЕРЖАНИЕ
ИСТОРИЯ ИЗМЕНЕНИЙ   
ВВЕДЕНИЕ   
ЦЕЛЬ ДОКУМЕНТА    
ОБЛАСТЬ ПРИМЕНЕНИЯ    
ГЛОССАРИЙ   
ССЫЛКИ  
ТРЕБОВАНИЯ К СЕРВЕРНОЙ ЧАСТИ    
IB.GETLOYALTYINFO   
Структура запроса и ответа   
Основной сценарий (MC)  
Альтернативные сценарии (AC)    
Исключительные сценарии (EC)    
IB.GETLOYALTYINFODETAILS  
Структура запроса и ответа	 
Основной сценарий (MC)	 
Альтернативные сценарии (AC)  	 
Исключительные сценарии (EC)

### ИСТОРИЯ ИЗМЕНЕНИЙ
История изменений и согласований документа.

| 01.10.2019 | 0.2 | Добавила изменения по бонусному счёту клиента | Медзяновская А. |
| ------------- | ------------- | ------------- | ------------- |  

### Введение
#### Цель документа
Целью документа является описание модуля «Лояльность» системы my|GEMINI
#### Область применения
Эта бизнес-область предназначена для операций с бонусами клиента. Клиенты могут просматривать бонусы, детали бонусов.
#### Глоссарий
Таблица содержит важные определения, сокращения и аббревиатуры:

| Термин | Описание |
| ------------- | ------------- |
| **АБС** | Автоматизированная банковская система |
| **myInternet, ИБ** | Краткое обозначение системы myGEMINI Internet (канал интернет-банкинга Системы) |
| **myGEMINI, Систе-ма** | Интегрированная многоканальная фронтальная банковская система myGEMINI |
|**ESB, шина**| IBM WebSphere Enterprise Service Bus |
| **Оператор (Bank Operator)** | Сотрудник Банка, уполномоченный на выполнение тех или иных функций в консоли администрирования Системы |
| **Клиент (Client)** | Клиент банка – зарегистрированное в одной или нескольких соответ-ствующих АБС физическое, юридическое лицо или индивидуальный предприниматель, пользующееся теми или иными финансовыми или дополнительными услугами (продуктами) банка |
| **Канал** | это комплекс организационно-технических средств, процедур и параметров, позволяющих осуществлять прямое обслуживание конечного клиента банка или пользователя банковской автоматизированной системы. Например, канал Интернет банк. |
| **Пользователь (User)** | Физическое лицо, представляющее Клиента банка в том или ином канале Системы. Пользователь может быть либо представителем Кли-ента (уполномоченный сотрудник юридического лица, доверенное лицо физического лица), либо самим Клиентом (физическим лицом, индивидуальным предпринимателем).|
| **Программа лояль-ности** | Бонусная программа, позволяющая держателям карт банка накапли-вать бонусы, которые впоследствии можно использовать, совершая покупки в точках продаж партнёров программы |
| **Бонусы** | Бонусы лояльности, начисленные согласно программе лояльности участнику данной программы |

### Ссылки
Ссылки на другие документы:

| Номер| Описание |
| ------------- | ------------- |
| **[1]** | CommonTypes_89 |
| **[2]** | loyalty-document_1.0.0_draft |

### Требования к серверной части
#### *IB.GetLoyaltyInfo*
**SR-1.** url: /rest/private/loyalty/info; method: GET  
**SR-2.** Назначение сервиса: получение суммы начисленных бонусов
##### Структура запроса и ответа
**SR-3.** Структура запроса: Нет  
**SR-4.** Структура ответа GetLoyaltyInfoResponseIBMo (extends AbstactResponseMo):  

| Название параметра | Тип | Описание/Значение | Обязательность |
| ------------- | ------------- | ------------- | ------------- |
| loyaltyContractId | Long | Идентификатор бонусного контракта | Mandatory |  
| availableLoyaltyAmount | MoneyMo | Общая сумма начисленных баллов | Mandatory |

#### Основной сценарий (MC)
**SR-5.**	[Шаг1] myGEMINI Core получает и разбирает запрос к сервису IB.GetLoyaltyInfo.  
**SR-6.**	[Шаг2] myGEMINI Core проверяет параметры запроса на соответствие прото-колу (Неуспешная проверка: EC-BadRequest[1])  
**SR-7.**	[Шаг3] myGEMINI Core проверяет, что роль текущего пользователя предусматривает права на операцию 3.71.01.00-02 (Неуспешная проверка: EC-Forbidden[1])  
**SR-8.**	[Шаг4] myGEMINI Core формирует и отправляет GET запрос к методу getLoyaltyContract МкС LoyaltyContract для получения информации о бонус-ном счете.

| Элемент запроса | Значение |
| ------------- | ------------- |
| clientId | Идентификатор клиента в АБС CIF |
| _exclude | programs,preliminaryCalculations,agreements |  

**SR-9.**	[Шаг5] myGEMINI Core разбирает полученный ответ от мкс LoyaltyContract.  
**SR-10.**	[Шаг6] myGEMINI Core проверяет, что ответ от метода getLoyaltyContract микросервиса LoyaltyContract содержит http status 200. (EC-Exception[1])  
**SR-10.1.**	HTTP status 404: (EC-1)  
**SR-11.**	[Шаг7] myGEMINI Core проверяет, что бонусный баланс активен: GetLoyaltyContractResponse/bonusBalanceStatus == '1'. (Неуспешная провер-ка: EC-2)  
**SR-12.**	[Шаг8] myGEMINI Core сохраняет описание бонусного счета в БД в соответ-ствии с маппингом [4]  
**SR-13.**	[Шаг9] myGEMINI Core получает сохраненное описание бонусного счета из БД.  
**SR-14.**	[Шаг10] myGEMINI Core формирует ответ приложению, значения атрибутов которого заполняются согласно следующим правилам:

| Элемент запроса | Значение |
| ------------- | ------------- |
| loyaltyContractId | Идентификатор бонусного контракта в БД LoyaltyAccount.id |
| availableLoyaltyAmount | Значение LoyaltyAccount.availableLoyaltyAmount | 
| availableLoyaltyAmount/currency | Значение LoyaltyAccount.availableLoyaltyAmountCurrency |  

**SR-15.**	[Шаг11] myGEMINI Core возвращает ответ приложению.  
**SR-16.**	[Шаг12] Сценарий завершается.
#### Исключительные сценарии (EC)
**EC-1. Продукт с полученным productId не найден или Клиент не найден в мастер-системе лояльности**  
**SR-17.**	[Шаг1] myGEMINI Core формирует описание ошибки:

HTTP/1.1 400

{ "errorResponseMo": {  
"errorMsg": "Функциональность временно недоступна",  
"errorCode": "UNKNOWN_ERROR",  
"errorSubCode": "PRODUCT_NOT_FOUND",  
"errorFields": []  
}  
}  
**SR-18.**	[Шаг2] myGEMINI Core возвращает описание ошибки в ответе.
**SR-19.**	Сценарий завершается.
**EC-2. Статус бонусного баланса неактивный GetLoyaltyContractResponse/bonusBalanceStatus == '0'**
**SR-20.**	[Шаг1] myGEMINI Core формирует описание ошибки:  
HTTP/1.1 400  
{  
"errorResponseMo": {  
"errorMsg": " Бонусный счет заблокирован",  
"errorCode": "BUSINESS_ERROR",  
"errorSubCode": "BONUS_BALANCE_BLOCKED",  
"errorFields": []  
}  
}

**SR-21.**	[Шаг2] myGEMINI Core возвращает описание ошибки в ответе.
**SR-22.**	Сценарий завершается.

#### IB.GetLoyaltyInfoDetails
**SR-23.**	url: /rest/private/loyalty/info/details; method: GET  
**SR-24.**	Назначение сервиса: сумма начисленных бонусов

Структура запроса и ответа  
**SR-25.**	Структура запроса:

| Название параметра | Тип | Описание/Значение | Обязательность |
| ------------- | ------------- | ------------- | ------------- |
| forceRequest | Boolean | Признак необходимости обращения в МкС | Optional |  

**SR-26.**	Структура ответа GetLoyaltyInfoResponseIBMo (extends AbstactResponseMo)

| Название параметра | Тип | Описание/Значение | Обязательность | 
| ------------- | ------------- | ------------- | ------------- |
| loyaltyContractId | Long | Идентификатор бонусного контракта | Mandatory | 
| availableLoyaltyAmount | MoneyMo | Общая сумма начисленных баллов | Mandatory | 
| loyaltyPrograms | List<LoyaltyProgramIBMo> | Информация о бонусной программе | Optional |  

**SR-27.**	Структура LoyaltyProgramIBMo

| Название параметра | Тип | Описание/Значение | Обязательность | 
| ------------- | ------------- | ------------- | ------------- |
| loyaltyProgramId | Integer | Идентификатор бонусной программы | Mandatory | 
| loyaltyProgramName | String | Наименование бонусной программы | Mandatory | 
| totalTransactionAmount | MoneyMo | Общая сумма транзакций | Mandatory |
| percentTariff | String | Проценты для отрисовки на ползунке | Mandatory |
| loyaltyProgramCondition | List <LoyaltyProgramConditionIBMo> | Условия процентной ставки по бонусной программе | Mandatory |  

**SR-28.**	Структура LoyaltyProgramConditionIBMo

| Название параметра |	Тип | Описание/Значение | Обязатель-ность |
| ------------- | ------------- | ------------- | ------------- |
| counterGreaterThan | String | Значение суммы в рублях от которой действует указанный % | Mandatory |
| percent | String | Действующий процент для указанного интервала | Mandatory |  

#### Основной сценарий (MC)
**SR-29.**	[Шаг1] myGEMINI Core получает и разбирает запрос к сервису GetLoyaltyInfo.  
**SR-30.**	[Шаг2] myGEMINI Core проверяет параметры запроса на соответствие про-токолу (Неуспешная проверка: EC-BadRequest[1])  
**SR-31.**	[Шаг3] myGEMINI Core проверяет, что роль текущего пользователя преду-сматривает права на операцию 3.71.01.00-02 (Неуспешная проверка: EC-Forbidden[1])  
**SR-32.**	[Шаг4] myGEMINI Core что в рамках сессии еще не выполнялся запрос на получение информации по бонусному счету или в запросе получен пара-метр request/forceRequest == 'true'. (AC-1)  
**SR-33.**	[Шаг5] myGEMINI Core формирует и отправляет GET запрос к мето-ду getLoyaltyContract МкС LoyaltyContract для получения информации о бо-нусном счете.

|Элемент запроса | Значение |
| ------------- | ------------- |
| clientId | Идентификатор клиента в АБС CIF |
| _exclude | preliminaryCalculations,agreements |  

**SR-34.**	[Шаг6] myGEMINI Core разбирает полученный ответ от мкс LoyaltyContract.   
**SR-35.**	[Шаг7] myGEMINI Core проверяет, что ответ от метода getLoyaltyContract микросервиса LoyaltyContract содержит http status 200. (EC-Exception [1])  
**SR-35.1.**	HTTP status 404: (EC-1)  
**SR-36.**	[Шаг8] myGEMINI Core проверяет, что бонусный баланс активен: GetLoyaltyContractResponse/bonusBalanceStatus == '1'. (Неуспешная провер-ка: EC-2)  
**SR-37.**	[Шаг9] myGEMINI Core сохраняет описание бонусного счета в БД в соответ-ствии с маппингом [4].  
**SR-38.**	[Шаг10] myGEMINI Core проверяет, что в IMDG найден бонусный тариф с идентификатором == GetLoyaltyContractResponse/programId, для которого GetLoyaltyContractResponse/programs/summTransaction != 'null' (Неуспешная проверка: AC-2)  
**SR-39.**	[Шаг11] myGEMINI Core рассчитывает параметры бонусного тарифа:  
**SR-39.1.**	Список GetLoyaltyTariffResponse/conditions сортируется по возрас-танию значения GetLoyaltyTariffResponse/condition/percent  
**SR-39.2.**	Выполняется сравнение значения GetLoyaltyContractResponse/summTransaction со значе-нием GetLoyaltyTariffResponse/condition/counterGreaterThan
*	Для сравнения берется значение counterGreaterThan по убыванию
*	Если summTransaction больше counterGreaterThan, сравнение продолжается.  
     **SR-39.2..1.**	В случае если summTransaction  больше всех экземпляров counterGreaterThan, сравнение прекращается, в качестве percentTariff выбирается максимоатльное из counterGreaterThan
*	Если summTransaction меньше counterGreaterThan, то дальнейшее сравнение прекращается.  
     **SR-39.3.**	Значение GetLoyaltyTariffResponse/condition/percent, ассоцииро-ванное по вычисленно-му значению counterGreaterThan, сохраняется как значе-ние percentTariff

**SR-40.**	[Шаг12] myGEMINI Core обновляет описание бонусного счета в БД в соот-ветствии с маппингом [4]  
**SR-41.**	[Шаг13] myGEMINI Core получает сохраненное описание бонусного счета из БД.  
**SR-42.**	[Шаг14] myGEMINI Core формирует ответ приложению согласно следующим правилам:   

| Атрибут ответа | Значение |
| ------------- | ------------- |
| loyaltyContractId | Идентификатор бонусного контракта в БД LoyaltyAccount.id |
| availableLoyaltyAmount | Значение LoyaltyAccount.availableLoyaltyAmount |
| availableLoyaltyAmount/currency | Значение LoyaltyAccount.availableLoyaltyAmountCurrency |
| loyaltyPrograms | Список с описанием бонусной программы |

| Атрибут объекта | Значение |
| ------------- | ------------- |
| loyaltyProgramId | Значение LoyaltyAccount.loyaltyProgramId |
| loyaltyProgramName | Значение LoyaltyAccount.loyaltyProgramName |
| totalTransactionAmount | Значение LoyaltyProgramTransactions.totalTransactionAmount |
| totalTransactionAmount/currency | Значение LoyaltyProgramTransactions.totalTransactionAmountCurrency |
| percentTariff | LoyaltyAccount.percentTariff + '%' |
| loyaltyProgramCondition | Список значений GetLoyaltyTariffResponse/conditions отсортированный по возрастанию значения percent |
| counterGreaterThan | Значение LoyaltyProgramCondition.counterGreaterThan |
| percent | Значение LoyaltyProgramCondition.percent |  

**SR-43.**	[Шаг15] myGEMINI Core возвращает ответ приложению  
**SR-44.**	Сценарий завершается

#### Альтернативные сценарии (AC)

**AC-1  В рамках сессии ранее был выполнен запрос к методу getLoyaltyContract МкС LoyaltyContract**  
**SR-45.**	[Шаг1] myGEMINI Core получает сохраненное описание бонусного контракта из БД.  
**SR-46.**	Основной сценарий продолжается с шага 14.

**AC-2  В рамках сессии ранее был выполнен запрос к методу getLoyaltyContract МкС LoyaltyContract**  
**SR-47.**	[Шаг1] myGEMINI Core формирует и отправляет параллельные GET запросы к методу getLoyaltyTariff МкС LoyaltyTariff для получения информации о бо-нусном тарифе, по каждой программе.  

| Элемент запроса | Значение |
| ------------- | ------------- |
| programId | Идентификатор программы лояльности Значение GetLoyaltyContractResponse/programId |   |

**SR-48.**	[Шаг2] myGEMINI Core разбирает полученный ответ от мкс LoyaltyTariff.  
**SR-49.**	[Шаг3] myGEMINI Core проверяет, что ответ от метода getLoyaltyTariff микросервиса LoyaltyTariff содержит http status 200.   
**SR-49.1.**	HTTP status 401 или 400 или 500 или вызов завершен по тай-мауту: (EC-Exception[1])  
**SR-50.**	[Шаг4] myGEMINI сохраняет данные по тарифу в IMDG, в соответ-ствии с маппингом [4]  
**SR-51.**	Основной сценарий продолжается с шага 12 - расчет параметров бонусного тарифа.

#### Исключительные сценарии (EC)

**EC-1. Продукт с полученным productId не найден или Клиент не найден в мастер-системе лояльности**
**SR-52.**	[Шаг1] myGEMINI Core формирует описание ошибки:

HTTP/1.1 400  
{  
"errorResponseMo": {  
"errorMsg": "Функциональность временно недоступна",   
"errorCode": "UNKNOWN_ERROR",  
"errorSubCode": "PRODUCT_NOT_FOUND",  
"errorFields": []  
}  
}

**SR-53.**	 [Шаг2] myGEMINI Core возвращает описание ошибки в ответе.  
**SR-54.**	Сценарий завершается.

**EC-2. Продукт с полученным productId не найден или Клиент не найден в мастер-системе лояльности**
**SR-55.**	[Шаг1] myGEMINI Core формирует описание ошибки:

HTTP/1.1 400  
{  
"errorResponseMo": {  
"errorMsg": "Бонусный счет заблокирован",  
"errorCode": "BUSINESS_ERROR",  
"errorSubCode": "BONUS_BALANCE_BLOCKED",  
"errorFields": []  
}  
}

**SR-56.**	 [Шаг2] myGEMINI Core возвращает описание ошибки в ответе.  
**SR-57.**	Сценарий завершается.

### IB.GetLoyaltyInfoHistory
**SR-58.**	url: /rest/private/loyalty/info/history; method: GET  
**SR-59.**	Назначение сервиса: вывод истории операций с бонусами
#### Структура запроса и ответа
**SR-60.**	Структура запроса: Нет  
**SR-61.**	Структура ответа IB. GetLoyaltyInfoHistory (extends AbstactResponseMo):


| Название параметра | Тип | Описание/Значение | Обязательность |
| ------------- | ------------- | ------------- | ------------- |
| loyaltyContractId | Long | Идентификатор бонусного контракта | Mandatory |
| operationDate | Date | Общая сумма начисленных баллов | Mandatory |  

#### Основной сценарий (MC)

**SR-62.**	[Шаг1] myGEMINI Core получает и разбирает запрос к сервису IB. GetLoyal-tyInfoHistory.  
**SR-63.**	[Шаг2] myGEMINI Core проверяет параметры запроса на соответствие прото-колу (Неуспешная проверка: EC-BadRequest[1])  
**SR-64.**	[Шаг3] myGEMINI Core проверяет, что роль текущего пользователя предусматривает права на операцию 3.71.01.00-02 (Неуспешная проверка: EC-Forbidden[1])  
**SR-65.**	[Шаг4] myGEMINI Core формирует и отправляет GET запрос к методу GetLoy-altyInfoHistory МкС LoyaltyContract для получения информации об истории операций на бонусном счёте.  
**SR-66.**	 [Шаг5] myGEMINI Core разбирает полученный ответ от мкс LoyaltyContract.  
**SR-67.**	[Шаг6] myGEMINI Core проверяет, что ответ от метода getLoyaltyContract микросервиса LoyaltyContract содержит http status 200. (EC-Exception[1])  
**SR-67.1.**	HTTP status 404: (EC-1)  
**SR-68.**	[Шаг7] myGEMINI Core проверяет, что бонусный баланс активен: GetLoyaltyContractResponse/bonusBalanceStatus == '1'. (Неуспешная провер-ка: EC-2)  
**SR-69.**	[Шаг8] myGEMINI Core сохраняет описание операций бонусного счета в БД в соответствии с маппингом [4]  
**SR-70.**	[Шаг9] myGEMINI Core получает сохраненный список операций с бонусного счета из БД.  
**SR-71.**	[Шаг10] myGEMINI Core формирует ответ приложению, значения атрибутов которого заполняются согласно следующим правилам:

| Элемент запроса | Значение |
| ------------- | ------------- |
| loyaltyContractId | Идентификатор бонусного контракта в БД LoyaltyAccount.id |
| description | Вид операции (списание/начисление) |
| operationAmount | Сумма операции |
| operationCurrency | Валюта операции |
| bonusAmount | Сумма бонусов |
| bonusCurrency | Валюта бонусов |  

**SR-72.**	[Шаг11] myGEMINI Core возвращает ответ приложению.  
**SR-73.**	[Шаг12] Сценарий завершается.

#### Исключительные сценарии (EC)
**EC-1. Продукт с полученным productId не найден или Клиент не найден в мастер-системе лояльности**

**SR-74.**	[Шаг1] myGEMINI Core формирует описание ошибки:  
HTTP/1.1 400  
{  
"errorResponseMo": {  
"errorMsg": "Функциональность временно недоступна",  
"errorCode": "UNKNOWN_ERROR",  
"errorSubCode": "PRODUCT_NOT_FOUND",  
"errorFields": []  
}  
}  
**SR-75.**	[Шаг2] myGEMINI Core возвращает описание ошибки в ответе.  
**SR-76.**	Сценарий завершается.

**EC-2. Статус бонусного баланса неактивный GetLoyaltyContractResponse/bonusBalanceStatus == '0'**

**SR-77.**	[Шаг1] myGEMINI Core формирует описание ошибки:  
HTTP/1.1 400  
{  
"errorResponseMo": {  
"errorMsg": " Бонусный счет заблокирован",  
"errorCode": "BUSINESS_ERROR",  
"errorSubCode": "BONUS_BALANCE_BLOCKED",  
"errorFields": []  
}  
}  
**SR-78.**	[Шаг2] myGEMINI Core возвращает описание ошибки в ответе.  
**SR-79.**	Сценарий завершается.

**EC-3. Не было совершено ни одной операции с начислением/списанием бонусов**  
**GetLoyaltyInfoBonus/operationDate == '0'**

**SR-80.**	[Шаг4] myGEMINI Core отображает значение параметра GetLoyaltyInfoBo-nus/bonusEmpty.  
**SR-81.**	Сценарий завершается.

### Таблица «История операций по бонусному счету»
**SR-82.**	Наименование таблицы - LoyaltyHistory  
**SR-83.**	Структура таблицы представлена ниже:

| Название параметра | Тип | Описание/Значение |
| ------------- | ------------- | ------------- |
| loyaltyContractId | Long | Идентификатор бонусного контракта |
| operationDate | Date | Дата совершения операции |
| description | String | Описание операции |
| operationAmount | String | Сумма операции |
| operationCurrency | String | Валюта операции |
| bonusAmount | String | Сумма бонусов |
| bonusCurrency | String | Валюта бонусов |