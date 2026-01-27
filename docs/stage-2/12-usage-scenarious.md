# 12. Сценарии использования приложения

Ниже приведены ключевые пользовательские сценарии, отражающие основные ценности платформы: тренировки, социальное взаимодействие, челленджи, геймификация и рекомендации.  
Каждый сценарий описывает последовательность действий пользователя и реакцию системы на уровне концептуальной архитектуры.


## 1. Регистрация и создание профиля

### Описание
Пользователь устанавливает приложение, создаёт аккаунт и настраивает профиль.

### Основной поток
1. Пользователь открывает приложение и выбирает «Создать аккаунт».
2. Вводит email/телефон или использует социальный логин.
3. Приложение отправляет запрос в **External API Gateway**.
4. **User Service** создаёт профиль и сохраняет данные в **User Store**.
5. Пользователь настраивает цели, параметры и предпочтения.
6. **User Service** публикует событие *UserCreated* в **Event Bus**.
7. **Recommendation Service** и **Analytics Service** инициализируют модели для нового пользователя.

### Ценность
Быстрый вход в платформу и персонализация с первого шага.

```mermaid
sequenceDiagram
    participant U as User
    participant App as Mobile/Web App
    participant GW as External API Gateway
    participant US as User Service
    participant AS as Auth Service
    participant ST as User Store
    participant EB as Event Bus
    participant RS as Recommendation Service
    participant AN as Analytics Service

    U ->> App: Открыть приложение, выбрать "Создать аккаунт"
    App ->> GW: POST /auth/signup
    GW ->> AS: Validate credentials / social login
    AS -->> GW: Auth result
    GW ->> US: Create user profile
    US ->> ST: Save user profile
    ST -->> US: OK
    US ->> EB: Publish UserCreated
    EB ->> RS: UserCreated
    EB ->> AN: UserCreated
    RS -->> EB: Init user model
    AN -->> EB: Init analytics profile
    GW -->> App: Signup success + user profile
    App -->> U: Профиль создан, переход к настройкам
```

## 2. Подключение устройства и сбор телеметрии

### Описание
Пользователь подключает фитнес-браслет или другое устройство, которое отправляет данные о тренировке.

### Основной поток
1. Пользователь открывает экран «Устройства».
2. Приложение инициирует подключение через BLE/MQTT/HTTPS.
3. Устройство отправляет телеметрию в **IoT Gateway**.
4. **Telemetry Ingest** валидирует данные и сохраняет их в **Training Store**.
5. События *TelemetryReceived* публикуются в **Event Bus**.
6. **Training Service**, **Analytics Service** и **Gamification Service** реагируют на события.

### Ценность
Надёжный сбор данных в реальном времени, основа для всех тренировочных функций.

```mermaid
sequenceDiagram
    participant U as User
    participant App as Mobile App
    participant Dev as Device
    participant IoT as IoT Gateway
    participant TI as Telemetry Ingest
    participant TS as Training Store
    participant EB as Event Bus
    participant TR as Training Service
    participant AN as Analytics Service
    participant GM as Gamification Service

    U ->> App: Открыть экран "Устройства"
    App ->> Dev: Pair / Connect
    Dev ->> IoT: Send telemetry stream
    IoT ->> TI: Forward telemetry
    TI ->> TS: Store telemetry
    TS -->> TI: OK
    TI ->> EB: Publish TelemetryReceived
    EB ->> TR: TelemetryReceived
    EB ->> AN: TelemetryReceived
    EB ->> GM: TelemetryReceived
    TR -->> EB: Update training state
    AN -->> EB: Update analytics
    GM -->> EB: Update gamification state
```

## 3. Проведение тренировки

### Описание
Пользователь начинает тренировку, получает обратную связь и завершает сессию.

### Основной поток
1. Пользователь нажимает «Начать тренировку».
2. Приложение отправляет команду в **Training Service** через **External API Gateway**.
3. Устройство передаёт телеметрию в реальном времени.
4. **Training Service** обновляет состояние тренировки.
5. **Gamification Service** начисляет очки и прогресс.
6. По завершении тренировки создаётся событие *TrainingCompleted*.
7. **Analytics Service** обновляет статистику и рекомендации.

### Ценность
Бесшовный тренировочный опыт с мгновенной обратной связью.

```mermaid
sequenceDiagram
    participant U as User
    participant App as Mobile App
    participant GW as External API Gateway
    participant TR as Training Service
    participant Dev as Device
    participant IoT as IoT Gateway
    participant TI as Telemetry Ingest
    participant TS as Training Store
    participant EB as Event Bus
    participant GM as Gamification Service
    participant AN as Analytics Service

    U ->> App: Нажать "Начать тренировку"
    App ->> GW: POST /training/start
    GW ->> TR: StartTraining
    TR -->> GW: Training session id
    GW -->> App: Session started

    loop During training
        Dev ->> IoT: Telemetry
        IoT ->> TI: Forward telemetry
        TI ->> TS: Store telemetry
        TI ->> EB: TelemetryReceived
        EB ->> TR: TelemetryReceived
        EB ->> GM: TelemetryReceived
        EB ->> AN: TelemetryReceived
        TR -->> App: Optional live feedback
    end

    U ->> App: Нажать "Завершить тренировку"
    App ->> GW: POST /training/complete
    GW ->> TR: CompleteTraining
    TR ->> EB: Publish TrainingCompleted
    EB ->> GM: TrainingCompleted
    EB ->> AN: TrainingCompleted
    GM -->> EB: Update points / badges
    AN -->> EB: Update stats
    GW -->> App: Training summary
    App -->> U: Показать результаты тренировки
```

## 4. Просмотр ленты активности

### Описание
Пользователь открывает ленту, чтобы увидеть активности друзей, рекомендации и челленджи.

### Основной поток
1. Приложение запрашивает ленту через **External API Gateway**.
2. **Social Service** формирует ленту на основе:
   - событий из **Event Bus**,
   - данных из **Social Graph Store**,
   - рекомендаций от **Recommendation Service**.
3. Лента возвращается пользователю с минимальной задержкой.

### Ценность
Социальная вовлечённость и мотивация через активности друзей.

```mermaid
sequenceDiagram
    participant U as User
    participant App as Mobile/Web App
    participant GW as External API Gateway
    participant SS as Social Service
    participant SG as Social Graph Store
    participant RS as Recommendation Service
    participant EB as Event Bus

    U ->> App: Открыть ленту
    App ->> GW: GET /feed
    GW ->> SS: GetFeed(userId)
    SS ->> SG: Get friends / follows
    SG -->> SS: Social graph
    SS ->> RS: GetRecommendations(userId, context)
    RS -->> SS: Recommended items
    SS ->> EB: (optional) Query recent events
    EB -->> SS: Recent activities
    SS -->> GW: Feed items
    GW -->> App: Feed response
    App -->> U: Показать ленту активности
```

## 5. Участие в челленджах

### Описание
Пользователь присоединяется к челленджу и соревнуется с другими.

### Основной поток
1. Пользователь выбирает челлендж в приложении.
2. **Gamification Service** регистрирует участие.
3. Во время тренировок события *TelemetryReceived* и *TrainingCompleted* обновляют прогресс.
4. **Leaderboard Service** пересчитывает позиции участников.
5. Пользователь видит обновлённый рейтинг в реальном времени.

### Ценность
Геймификация и социальная мотивация.

```mermaid
sequenceDiagram
    participant U as User
    participant App as Mobile/Web App
    participant GW as External API Gateway
    participant GM as Gamification Service
    participant LB as Leaderboard Service
    participant EB as Event Bus
    participant TR as Training Service

    U ->> App: Открыть список челленджей
    App ->> GW: GET /challenges
    GW ->> GM: GetChallenges
    GM -->> GW: Challenges list
    GW -->> App: Challenges

    U ->> App: Присоединиться к челленджу
    App ->> GW: POST /challenges/{id}/join
    GW ->> GM: JoinChallenge(userId, challengeId)
    GM ->> EB: Publish ChallengeJoined
    EB ->> LB: ChallengeJoined
    GM -->> GW: Join success
    GW -->> App: Joined

    loop During challenge
        EB ->> LB: TrainingCompleted / TelemetryReceived
        LB -->> EB: Updated leaderboard
    end

    U ->> App: Открыть рейтинг
    App ->> GW: GET /challenges/{id}/leaderboard
    GW ->> LB: GetLeaderboard
    LB -->> GW: Leaderboard data
    GW -->> App: Leaderboard
    App -->> U: Показать позиции в челлендже
```

## 6. Получение персональных рекомендаций

### Описание
Пользователь получает рекомендации по тренировкам, челленджам и активности.

### Основной поток
1. Приложение запрашивает рекомендации через **External API Gateway**.
2. **Recommendation Service** использует:
   - данные профиля,
   - историю тренировок,
   - социальный граф,
   - аналитику из **Data Lake**.
3. Сервис возвращает персонализированные предложения.

### Ценность
Улучшение результатов и удержание пользователя.

```mermaid
sequenceDiagram
    participant U as User
    participant App as Mobile/Web App
    participant GW as External API Gateway
    participant RS as Recommendation Service
    participant US as User Service
    participant AN as Analytics Service
    participant DL as Data Lake

    U ->> App: Открыть экран "Рекомендации"
    App ->> GW: GET /recommendations
    GW ->> RS: GetRecommendations(userId)
    RS ->> US: Get user profile
    US -->> RS: User profile
    RS ->> AN: Get user stats / segments
    AN -->> RS: Stats / segments
    RS ->> DL: (optional) Query global models
    DL -->> RS: Model outputs
    RS -->> GW: Recommendations list
    GW -->> App: Recommendations
    App -->> U: Показать персональные рекомендации
```

## 7. Покупка товаров и участие в промо

### Описание
Пользователь получает персональные предложения и совершает покупку.

### Основной поток
1. Приложение запрашивает промо-предложения.
2. **Promo Service** формирует персональные офферы.
3. Пользователь переходит к покупке.
4. **Commerce Integration Service** взаимодействует с e-commerce платформой.
5. Покупка отображается в профиле пользователя.

### Ценность
Монетизация и персонализированный маркетинг.

```mermaid
sequenceDiagram
    participant U as User
    participant App as Mobile/Web App
    participant GW as External API Gateway
    participant PR as Promo Service
    participant CI as Commerce Integration
    participant EXT as E‑commerce Platform

    U ->> App: Открыть промо / магазин
    App ->> GW: GET /promotions
    GW ->> PR: GetPromotions(userId)
    PR -->> GW: Personalized offers
    GW -->> App: Promotions
    App -->> U: Показать офферы

    U ->> App: Выбрать товар / оффер
    App ->> GW: POST /orders
    GW ->> CI: CreateOrder(userId, itemId)
    CI ->> EXT: Create order via API
    EXT -->> CI: Order confirmed
    CI -->> GW: Order result
    GW -->> App: Order success
    App -->> U: Показать подтверждение покупки
```

## 8. Управление профилем и настройками

### Описание
Пользователь обновляет личные данные, цели, приватность и уведомления.

### Основной поток
1. Приложение отправляет изменения в **User Service**.
2. Данные обновляются в **User Store**.
3. Событие *UserUpdated* публикуется в **Event Bus**.
4. **Analytics**, **Recommendations** и **Social** обновляют свои модели.

### Ценность
Контроль над данными и персонализация.

```mermaid
sequenceDiagram
    participant U as User
    participant App as Mobile/Web App
    participant GW as External API Gateway
    participant US as User Service
    participant ST as User Store
    participant EB as Event Bus
    participant RS as Recommendation Service
    participant AN as Analytics Service
    participant SS as Social Service

    U ->> App: Открыть настройки профиля
    U ->> App: Изменить данные / цели / приватность
    App ->> GW: PUT /user/profile
    GW ->> US: UpdateUserProfile
    US ->> ST: Save updated profile
    ST -->> US: OK
    US ->> EB: Publish UserUpdated
    EB ->> RS: UserUpdated
    EB ->> AN: UserUpdated
    EB ->> SS: UserUpdated
    RS -->> EB: Update recommendation profile
    AN -->> EB: Update analytics profile
    SS -->> EB: Update social visibility
    GW -->> App: Update success
    App -->> U: Профиль обновлён
```

---

## Таблица соответствия сценариев, доменов и NFR

| № | Сценарий использования | Основные домены / сервисы | Ключевые NFR |
|---|------------------------|----------------------------|---------------|
| **1** | Регистрация и создание профиля | User Service, Auth Service, Recommendation Service, Analytics Service, Event Bus | Производительность (≤200 мс), Надёжность, Наблюдаемость, Безопасность |
| **2** | Подключение устройства и сбор телеметрии | IoT Gateway, Telemetry Ingest, Training Store, Training Service, Analytics, Gamification | Производительность (телеметрия ≤2 сек), Масштабируемость, Надёжность |
| **3** | Проведение тренировки | Training Service, IoT Gateway, Telemetry Ingest, Gamification, Analytics, Event Bus | Производительность, Надёжность (RPO ≤30 сек), Масштабируемость |
| **4** | Просмотр ленты активности | Social Service, Social Graph Store, Recommendation Service, Event Bus | Производительность (лента ≤150 мс), Масштабируемость, Наблюдаемость |
| **5** | Участие в челленджах | Gamification Service, Leaderboard Service, Training Service, Event Bus | Масштабируемость (100k участников), Производительность, Надёжность |
| **6** | Получение персональных рекомендаций | Recommendation Service, Analytics Service, User Service, Data Lake | Производительность, Масштабируемость, Интероперабельность |
| **7** | Покупка товаров и промо | Promo Service, Commerce Integration, External E‑commerce | Безопасность, Интероперабельность, Производительность |
| **8** | Управление профилем и настройками | User Service, Social Service, Analytics, Recommendation Service, Event Bus | Надёжность, Безопасность, Наблюдаемость |

---


Эти сценарии использования:

- отражают ключевые пользовательские ценности,  
- согласованы с концептуальной архитектурой,  
- задействуют все основные домены (Training, Social, Gamification, Analytics, IoT),  
- демонстрируют работу API Gateway, Event Bus, Data Lake и региональных хранилищ,  
- служат основой для проектирования пользовательских потоков, API и тестирования.

