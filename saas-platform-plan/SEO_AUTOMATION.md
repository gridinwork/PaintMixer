# SEO_AUTOMATION.md — Архитектура AI Marketing Manager и SEO-автоматизации

Дата: 2026-10-12.

## 1. Принципы
- Фоновые задачи — **обычные скрипты + планировщик**, не зависят от сессии Claude.
- Все действия пишутся в журнал (`audit_log`), внешние публикации — через очередь утверждений.
- Нет LLM-вызовов в cron по умолчанию (стоят денег); по желанию — локальная LLM на вашем GPU.
- Данные только из официальных API и собственной аналитики.

## 2. Модули

| Модуль | Что делает | Данные/API | Расписание | Кат. |
|---|---|---|---|---|
| **Keyword Scout** | Копит кандидатов (автодополнение вручную/из GSC «запросы без кликов»), сопоставляет с существующими страницами, помечает пробелы | Search Console API, ручной ввод из Trends/Keyword Planner | Еженедельно | B |
| **SEO Manager** | Проверяет: title/description, canonical, hreflang, schema, robots, sitemap, битые ссылки, код ответа, Core Web Vitals; сверяет «sitemap vs индекс» | Краулер по своему сайту, PageSpeed Insights API, URL Inspection API (выборка ≤ 2000/день) | Ежедневно (краул), еженедельно (индекс) | B |
| **Content Generator** | Готовит черновики инструкций/описаний на основе фактических возможностей инструмента + шаблон «что проверить человеку» | Локальная LLM или ручная сессия Claude | По запросу | C (утверждение) |
| **Distribution Manager** | Очередь публикаций (DEV/Hashnode), готовые тексты для ручных каналов, напоминания | DEV/Hashnode API | По событию | B/C |
| **Search Analytics** | Собирает GSC + Bing + собственную аналитику в Postgres | GSC API, Bing Webmaster API, Cloudflare Analytics | Ежедневно | B |
| **Growth Planner** | Рейтинг страниц/инструментов по росту; предлагает, какие функции развивать; генерирует недельный отчёт | БД | Еженедельно | B (правила) + C (решение) |
| **Dashboard** | Веб-панель владельца (внутри админки платформы) | Postgres | Онлайн | B |
| **Automation Scheduler** | `systemd` timers/cron/GitHub Actions; ретраи, алерты на почту/Telegram бот | — | — | B |

## 3. Поток публикации новой страницы
1. Страница добавлена в репозиторий (контент + мета).
2. CI: линтеры, проверка ссылок, schema validation, Lighthouse CI.
3. Деплой (после вашего approve или автоматически для безопасных изменений вроде правки опечаток — правило настраивается).
4. Генерация sitemap, `Last-Modified`.
5. Отправка sitemap (GSC API) + IndexNow (Bing/Yandex).
6. Через 3–7 дней: URL Inspection API — статус; если «Discovered – not indexed» дольше 14 дней → задача «улучшить контент/ссылки».
7. Запись в журнал.

## 4. Технические требования к сайту (минимум)
`robots.txt`, `sitemap.xml` (с lastmod), canonical на каждой странице, `hreflang` (если будут языковые версии), структурированные данные: `SoftwareApplication`/`WebApplication`, `FAQPage` (⚠ сниппеты FAQ в Google сильно ограничены; это не гарантия), `HowTo` (⚠ устарел в выдаче — не рассчитывать), `BreadcrumbList`. SSR/SSG (Next.js), быстрый LCP, отсутствие CLS, адаптив. Чёткая «About/Contact/Privacy/Terms».

## 5. Что НЕ делаем
- Парсинг выдачи Google (нарушает условия) — вместо этого средняя позиция из GSC.
- Массовая генерация шаблонных страниц ради охвата запросов.
- Скрытый текст, клоакинг, PBN, покупные ссылки, спам комментариев.
- Использование Indexing API вне JobPosting/BroadcastEvent.

## 6. Минимальная схема БД
`pages(url, tool, lang, created, updated)`, `gsc_daily(date, url, query, clicks, impressions, ctr, position)`, `bing_daily(...)`, `index_status(url, state, checked_at)`, `tasks(id, type, status, requires_approval, payload)`, `audit_log(ts, actor, action, target, result)`.

## 7. Сроки разработки модулей (❓ оценка)
Search Analytics + SEO Manager — 1–2 недели; Dashboard — 1 неделя; Keyword Scout/Growth Planner — по мере накопления данных (нужно минимум 4–8 недель истории).
