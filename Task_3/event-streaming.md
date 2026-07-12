# 4. Потоковая обработка данных

Kafka используется как поток событий между доменными сервисами, Statistics и Analytics. Позволяет разделить критичный RTB-путь и последующую обработку, а также повторно прочитать события после сбоя.

## Топики

| Топик | События |
| ---| --- |
| `campaign.events`| `campaign.changed`, `bid.changed`, `campaign.deactivated` |
| `budget.events` | `budget.changed`, `budget.reserved`   |
| `auction.events` | `auction.completed`, `auction.no_bid`|
| `click.events`      | `click.recorded` |
| `statistics.events` | `statistics.aggregated`|

Ключ обеспечивает порядок событий внутри конкретной кампании, аккаунта или аукциона.

## Consumer groups

| Группа                    | Читает                       | Назначение                  |
| ------------------------- | ---------------------------- | --------------------------- |
| `statistics-consumers`    | auction, impression, click   | Запись событий и агрегатов  |
| `analytics-consumers`     | statistics, campaign, budget | Построение витрин           |
| `auction-cache-consumers` | campaign, budget             | Обновление Redis и snapshot |
| `billing-consumers`       | auction                      | Резервирование и списание   |
| `fraud-consumers`         | impression, click            | Выявление аномалий          |

Экземпляры внутри одной группы используют competing consumers. Разные группы читают одно событие независимо.

## Политика хранения

| Топики                    |  Retention |
| ------------------------- | ---------: |
| Кампании и бюджеты        |  7–30 дней |
| Аукционы                  |  7–14 дней |
| Показы и клики            |   3–7 дней |
| Агрегированная статистика |    30 дней |
| DLQ                       | 30–90 дней |

Для конфигурационных топиков допускается log compaction, чтобы сохранять последнее состояние сущности.

## Надёжность
- гарантия доставки — at-least-once
- consumers должны быть идемпотентными
- после исчерпания retry событие направляется в DLQ
- Statistics и Analytics могут восстановить данные повторным чтением
