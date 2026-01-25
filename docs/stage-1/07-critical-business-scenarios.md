# Критические бизнес-сценарии

## 1. Сценарий: Запись и завершение тренировки

### Описание
Пользователь запускает тренировку, система получает телеметрию, фиксирует результаты и обновляет аналитику.

### Почему критично
- Основная ценность продукта.  
- Источник данных для рекомендаций, геймификации и социальной активности.  
- Высокая частота использования.

### Требования к архитектуре
- Низкая задержка при приёме данных.  
- Буферизация телеметрии.  
- Устойчивость к нестабильным каналам связи.  
- Масштабируемость при пиковых нагрузках.

```mermaid
sequenceDiagram
    autonumber
    participant User as Пользователь
    participant App as Mobile App
    participant API as External API Gateway
    participant Train as Training Service
    participant Telemetry as Telemetry Ingest
    participant Store as Training Store
    participant Bus as Event Bus
    participant Analytics as Analytics Service

    User ->> App: Запуск тренировки
    App ->> API: Отправка телеметрии
    API ->> Telemetry: Нормализация данных
    Telemetry ->> Store: Сохранение телеметрии
    Telemetry ->> Bus: Событие training.updated
    User ->> App: Завершение тренировки
    App ->> API: Итоговые данные
    API ->> Train: Создание записи тренировки
    Train ->> Store: Сохранение результата
    Train ->> Bus: Событие training.completed
    Bus ->> Analytics: Обновление агрегатов
```

## 2. Сценарий: Публикация результата тренировки в социальную ленту

### Описание
После завершения тренировки событие отображается в ленте активности друзей и групп.

### Почему критично
- Основной драйвер вовлечённости.  
- Формирует социальную ткань платформы.  
- Влияет на удержание и восприятие бренда.
- Является рекламой для других пользователей

### Требования к архитектуре
- Event-driven взаимодействие.  
- Быстрая генерация ленты (кэширование, денормализация).  
- Низкая задержка уведомлений.

```mermaid
sequenceDiagram
    autonumber
    participant Bus as Event Bus
    participant Social as Social Service
    participant Group as Group Service
    participant Notify as Notification Service
    participant App as Mobile App

    Bus ->> Social: Событие training.completed
    Social ->> Social: Обновление ленты активности
    Social ->> Group: Обновление активности групп
    Social ->> Notify: Уведомления друзьям и участникам групп
    Notify ->> App: Push-уведомления
```	


## 3. Сценарий: Получение персональных рекомендаций и промо

### Описание
Пользователь получает рекомендации по тренировкам, восстановлению и инвентарю, включая персональные промо.

### Почему критично
- Прямое влияние на вовлечённость.  
- Прямое влияние на монетизацию.  
- Требует качественных данных и ML-моделей.

### Требования к архитектуре
- Доступ к агрегированным данным.  
- Быстрый отклик Recommendation Service.  
- Интеграция с e-commerce.  
- Региональные каталоги.

```mermaid
sequenceDiagram
    autonumber
    participant User as Пользователь
    participant App as Mobile App
    participant API as External API Gateway
    participant Rec as Recommendation Service
    participant Promo as Promo Service
    participant Analytics as Analytics Service
    participant Commerce as Commerce Integration

    User ->> App: Открытие экрана рекомендаций
    App ->> API: Запрос рекомендаций
    API ->> Rec: Получение персональных рекомендаций
    Rec ->> Analytics: Запрос агрегированных данных
    Rec ->> Promo: Запрос персональных промо
    Promo ->> Commerce: Проверка доступности товаров
    Rec ->> API: Возврат рекомендаций и промо
    API ->> App: Отображение пользователю
```	
	
## 4. Сценарий: Массовые челленджи и соревнования

### Описание
Пользователь участвует в челленджах, результаты обновляются в реальном времени, система выдерживает пиковые нагрузки.

### Почему критично
- Сильный мотиватор для регулярных тренировок.  
- Высокие нагрузки (до 100k+ участников).  
- Важный элемент позиционирования бренда.

### Требования к архитектуре
- Горизонтальное масштабирование.  
- Асинхронная обработка событий.  
- Оптимизация рейтингов и агрегатов.  
- Защита от мошенничества.

```mermaid
sequenceDiagram
    autonumber
    participant User as Пользователь
    participant App as Mobile App
    participant API as External API Gateway
    participant Challenge as Challenge Service
    participant Leader as Leaderboard Service
    participant Bus as Event Bus
    participant Analytics as Analytics Service

    User ->> App: Участие в челлендже
    App ->> API: Регистрация в событии
    API ->> Challenge: Добавление участника
    Challenge ->> Bus: Событие challenge.joined

    Bus ->> Leader: Обновление рейтингов
    Bus ->> Analytics: Обновление статистики

    User ->> App: Завершение тренировки в рамках челленджа
    App ->> API: Данные тренировки
    API ->> Challenge: Обновление прогресса
    Challenge ->> Bus: Событие challenge.progress
    Bus ->> Leader: Пересчёт рейтингов
    Leader ->> App: Обновлённый рейтинг
```	
---

Эти четыре сценария покрывают:
- ядро продукта (тренировки),  
- вовлечённость (социальная лента),  
- монетизацию (рекомендации и промо),  
- масштабируемость (массовые события).

Они являются основой для проектирования архитектуры и проверки её жизнеспособности.
