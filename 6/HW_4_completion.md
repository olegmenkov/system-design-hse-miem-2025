### **Дополнение High-Level Design для проекта "Кто твой предок?"**

---

[ДЗ_1 с ФТ и НФТ](https://github.com/nikolaysavelev/system-design-hse-miem-2025/pull/6/files?short_path=c2b1b13#diff-c2b1b137c8b6980993e87a6c87d68e80f5d9287d3828b50d33049d9581a3d7fc)

[ДЗ_2 с сервисами](https://github.com/nikolaysavelev/system-design-hse-miem-2025/pull/15/files?short_path=296604c#diff-296604cd18c064534012a6528e0a93b49ffa6ec41ac137e35ed853adc05e6942)

[ДЗ_3 с БД](https://github.com/nikolaysavelev/system-design-hse-miem-2025/pull/16/files?short_path=86e7793#diff-86e77935846c8085ca2d251f3ed3adc3af3d97b095b558a757f8b908006dd240)

---

## **1. Обязательные компоненты (MUST)**

| Компонент                     | Обоснование                                                                                                                                 | Реализация (опционально)               |
|-------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------|----------------------------------------|
| **Load Balancer**             | Распределение нагрузки между API-серверами и микросервисами. Необходим для горизонтального масштабирования и отказоустойчивости.           | NGINX / Yandex Load Balancer / AWS ALB |
| **API Gateway**               | Единая точка входа для API, управление rate-limiting, аутентификацией, маршрутизацией.                                                      | Kong / Tyk / Yandex API Gateway        |
| **CDN**                       | Ускорение доставки статики (сканы документов, аватары пользователей). Снижает нагрузку на бэкенд.                                           | Cloudflare / Yandex CDN / VK Cloud CDN |
| **Кэш (Redis)**               | Кэширование часто запрашиваемых данных (деревья родословных, профили пользователей). Уменьшает нагрузку на БД.                              | Redis Cluster                          |
| **Identity Provider (IdP)**   | Централизованная аутентификация и авторизация (OAuth 2.0, JWT). Интеграция с соцсетями.                                                     | Keycloak / Auth0                       |
| **WAF (Web Application Firewall)** | Защита от OWASP Top 10 (SQLi, XSS, DDoS). Обязательно для хранения персональных данных.                                              | Cloudflare WAF / Yandex WAF            |
| **CI/CD Pipeline**            | Автоматизация сборки, тестирования и деплоя. Без этого невозможна быстрая разработка и отказоустойчивость.                                  | GitLab CI / GitHub Actions             |
| **Observability Stack**       | Мониторинг (метрики, логи, трейсы). Критично для масштабируемости и отладки.                                                               | Prometheus + Loki + Grafana + Jaeger   |
| **Резервное копирование**     | Соответствие 152-ФЗ. Гарантия восстановления данных (RPO < 1 мин).                                                                          | WAL-G (PostgreSQL) + S3 Snapshots      |
| **Message Broker**            | Асинхронная обработка задач (верификация документов, уведомления). Без него система не справится с пиковой нагрузкой.                       | RabbitMQ / Kafka                       |
| **Шардирование БД**           | Для PostgreSQL и Elasticsearch — обязательно из-за объема данных (500M пользователей, 10B документов).                                      | Citus (PostgreSQL) / Elasticsearch     |
| **S3-совместимое хранилище**  | Хранение сканов документов (до 25 МБ каждый). Объектное хранилище — единственный вариант для таких объемов.                                 | Yandex Object Storage / MinIO          |

---

## **2. Рекомендуемые компоненты (SHOULD)**

| Компонент                     | Обоснование                                                                                                                                 | Реализация (опционально)               |
|-------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------|----------------------------------------|
| **Service Mesh**              | Управление трафиком между микросервисами (retries, timeouts, circuit breaking). Полезно при >5 сервисах.                                    | Istio / Linkerd                        |
| **GeoDNS**                    | Маршрутизация пользователей к ближайшему дата-центру (актуально для РФ и СНГ).                                                              | Yandex DNS / Cloudflare GeoDNS         |
| **Feature Flags**             | Безопасный rollout новых функций (A/B-тестирование, откат без деплоя).                                                                     | LaunchDarkly / Flagsmith               |
| **Graph Database**            | Для сложных запросов к генеалогическому графу (если PostgreSQL + pgRouting недостаточно).                                                   | Neo4j                                  |
| **Distributed Task Queue**    | Для тяжелых задач (построение деревьев, массовая верификация).                                                                              | Celery + Redis                         |
| **Data Warehouse**            | Аналитика и отчеты (например, "сколько пользователей нашли предков до XVIII века").                                                         | ClickHouse                             |
| **Rate Limiter**              | Защита API от злоупотреблений (особенно для публичных эндпоинтов).                                                                          | Redis + Bucket4J                       |
| **Image Processing Service**  | Автоматическое сжатие и оптимизация сканов документов перед сохранением в S3.                                                              | Imgproxy / Thumbor                     |

---

## **3. Обновленная схема HLD**

Вот обновленная HLD схема в исходном формате с добавленными обязательными (MUST) и рекомендуемыми (SHOULD) компонентами:

### High-Level Design (HLD) v2

**Клиенты**
- Веб-клиент (React/Vue)
- Мобильные приложения (iOS/Android)
- Сторонние клиенты через API

**Входной слой** (добавлены MUST-компоненты)
- Load Balancer (NGINX/Yandex Load Balancer) ← NEW MUST
- WAF (Cloudflare/Yandex WAF) ← NEW MUST
- API Gateway (Kong/Tyk)
- OAuth Provider / Auth Service (Keycloak) ← NEW MUST
- CDN (Yandex CDN/VK Cloud CDN) ← NEW MUST

**Бизнес-сервисы** (без изменений)
- Genealogy Service
- Document Service
- Verification Service
- User Profile
- Notification Service
- Audit Service
- Search Service

**Инфраструктура** (расширена)
- PostgreSQL (шардирование через Citus) ← NEW MUST
- Redis Cluster ← NEW MUST
- S3 (Yandex Object Storage/MinIO) ← NEW MUST
- Elasticsearch
- RabbitMQ/Kafka ← NEW MUST
- OpenTelemetry
- Prometheus + Grafana (мониторинг) ← NEW MUST
- Jaeger (трейсы) ← NEW MUST
- Loki (логи) ← NEW MUST
- Backup (WAL-G + S3) ← NEW MUST

**Рекомендуемые компоненты** (SHOULD)
- Service Mesh (Istio/Linkerd) ← NEW SHOULD
- GeoDNS (Yandex DNS) ← NEW SHOULD
- Feature Flags (LaunchDarkly) ← NEW SHOULD
- Graph Database (Neo4j) ← NEW SHOULD
- Distributed Task Queue (Celery+Redis) ← NEW SHOULD
- Data Warehouse (ClickHouse) ← NEW SHOULD
- Rate Limiter (Redis+Bucket4J) ← NEW SHOULD
- Image Processing (Imgproxy) ← NEW SHOULD

### **Интеграции**:
- **Синхронные**: REST API (OpenAPI) между фронтендом и сервисами.  
- **Асинхронные**: RabbitMQ/Kafka для событий (верификация, уведомления).  


## **4. Обоснование ключевых выборов**
- **PostgreSQL + Elasticsearch**:  
  - Реляционная БД для строгих данных (пользователи, верификация), Elasticsearch — для поиска.  
- **Redis**:  
  - Кэш деревьев родословных (часто запрашиваемые данные).  
- **Service Mesh (Istio)**:  
  - По мере роста числа микросервисов (уже 9+) управление трафиком становится критичным.  
- **Neo4j (опционально)**:  
  - Если запросы к графу предков будут слишком сложными для PostgreSQL.  
