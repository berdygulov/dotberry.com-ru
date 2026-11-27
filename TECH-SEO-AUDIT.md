# Предварительный технический SEO-анализ dotberry.com

**Дата:** 2026-11-26
**Статус:** предварительный, без доступа к GSC, Яндекс.Вебмастеру и коду сайта.

**Методы:**
- **curl** — HTTP-заголовки, редиректы, исходный HTML, время ответа;
- **живой обход в Chrome** — отрисовка страниц, сетевые запросы, консоль, Performance API, поведение интерфейса.

Замеры скорости — лабораторные, с одного подключения и в небольшом числе попыток. Это ориентир, не данные CrUX. Перед выводами подтвердить через PageSpeed Insights и отчёт Core Web Vitals в GSC.

Сводка по проекту — [OVERVIEW.md](OVERVIEW.md), контентный аудит — [audit/2026-11-26-initial-audit.md](audit/2026-11-26-initial-audit.md).

---

## Сводная таблица

| # | Проблема | Приоритет | Кто исправляет |
|---|---|---|---|
| [1](#p1) | Автоматический редирект на `/ru/...` по языку браузера (307) со всех EN-страниц | 🔴 Высокий | Разработка |
| [2](#p2) | /pricing: зоны подгружаются кнопкой, порядок меняется при каждой загрузке, в списке дубли | 🔴 Высокий | Разработка |
| [3](#p3) | Ошибка 500 на служебных URL (`/favicon.ico`, `/manifest.json`, `/llms.txt`, `/index.html` …) | 🟠 Средний | Разработка |
| [4](#p4) | У `www.dotberry.com` нет DNS-записи | 🟠 Средний | DevOps |
| [5](#p5) | `/en/*` редиректит кодом 307 (временный) вместо 301/308 | 🟠 Средний | Разработка |
| [6](#p6) | Объём JavaScript ~820 КБ на главной | 🟠 Средний | Разработка |
| [7](#p7) | Нет заголовков безопасности (HSTS и др.) | 🟡 Низкий | DevOps |
| [8](#p8) | Кнопки «Купить» и выбор зон — `<button>`, а не ссылки | 🟡 Низкий (станет высоким с TLD-страницами) | Разработка |
| [9](#p9) | Пустой alt у изображений, у поля поиска нет `<label>` | 🟡 Низкий | Разработка |
| [10](#p10) | Предупреждения React в консоли (Tooltip controlled/uncontrolled) | 🟡 Низкий | Разработка |
| [11](#p11) | Sitemap: одинаковый lastmod, нелогичные priority | 🟡 Низкий | Разработка |

---

<a id="p1"></a>

## 1. Редирект по языку браузера (Accept-Language) 🔴

Редирект срабатывает не только на главной, а на **всех EN-страницах**.

**Доказательства: curl.**

```
/          Accept-Language: ru-RU,ru;q=0.9             -> 307 https://dotberry.com/ru
/          Accept-Language: kk-KZ,kk;q=0.9,ru;q=0.8    -> 307 https://dotberry.com/ru
/          Accept-Language: en-US,en;q=0.9             -> 200
/          Accept-Language: de-DE                      -> 200
/          Cookie NEXT_LOCALE=en + Accept-Language: ru -> 200
/pricing   Accept-Language: ru-RU,ru;q=0.9             -> 307 https://dotberry.com/ru/pricing
/domains   Accept-Language: ru-RU,ru;q=0.9             -> 307 https://dotberry.com/ru/domains
любой URL  Cookie NEXT_LOCALE=ru                       -> 307 на /ru/...
/          UA Googlebot + Accept-Language: ru          -> 307 https://dotberry.com/ru
/          UA YandexBot, без Accept-Language           -> 200
/          UA YandexBot + Accept-Language: ru          -> 307 https://dotberry.com/ru
```

**Доказательства: Chrome.** Переход на `https://dotberry.com/` открыл `https://dotberry.com/ru`. Последующие заходы на `/pricing` и `/domains` тоже перебрасывались на `/ru/...`, потому что в cookie записалось `NEXT_LOCALE=ru`.

**Что говорит Google** ([Managing multi-regional and multilingual sites](https://developers.google.com/search/docs/specialty/international/managing-multi-regional-sites)):
- *«Avoid automatically redirecting users from one language version of a site to a different language version of a site»*, *«don't redirect based on what you think the user's language may be»*: не редиректить автоматически на другую языковую версию по предполагаемому языку пользователя.
- Причина: *«These redirections could prevent users (and search engines) from viewing all the versions of your site»* — такие редиректы мешают пользователям и поисковикам увидеть все версии сайта.
- Googlebot *«usually originates from the USA»* и *«sends HTTP requests without setting Accept-Language»*: обходит в основном из США и без Accept-Language. Поэтому сейчас он видит EN-версию, и hreflang работает.
- Рекомендуемая альтернатива: отдельные URL для языков, hreflang и ссылки на другие языковые версии на каждой странице.

**Оговорка: редирект с главной Google допускает** ([Tell Google about localized versions](https://developers.google.com/search/docs/specialty/international/localized-versions)).

Что такое `x-default`. hreflang сообщает Google, какие языковые версии есть у страницы. У Dotberry в `<head>`, в HTTP-заголовке `Link` и в sitemap указано:

```
hreflang="en"        → https://dotberry.com/
hreflang="ru"        → https://dotberry.com/ru
hreflang="x-default" → https://dotberry.com/
```

`x-default` — запасная страница для всех, кому не подошла ни одна языковая версия (например, пользователю с немецким или японским языком браузера).

Какие страницы Google предлагает делать `x-default`: *«language/country selectors or auto-redirecting home pages»*:
- страница выбора языка («English / Русский»);
- **главная с автоматическим редиректом по языку**.

Обе — не отдельная языковая версия, а «развилка», которая направляет пользователя дальше.

Как это совместить с рекомендацией выше:
- Google **не советует** редиректить по языку.
- При этом главную с автоматическим редиректом он сам приводит как обычный пример `x-default`. Значит, этот сценарий Google знает и нарушением не считает, если редирект только с главной и она помечена как `x-default`.

**Что это значит для Dotberry:**
- `x-default` уже указывает на `/`, разметка корректна.
- Редирект **со всех страниц** (`/pricing`, `/domains`…), как сейчас, под это исключение не подпадает. Это не развилки, а конкретные EN-страницы.
- Решение — без редиректа, с баннером выбора языка (см. «Что сделать» ниже).

**Чем это опасно:**
- Пользователь из RU/KZ не может открыть EN-страницу по прямой ссылке: редиректят все URL, а не только главная. Это ломает ссылки, которыми делятся, и мешает сравнивать версии.
- Если какой-то краулер приходит с Accept-Language `ru` (возможно, YandexBot), он не увидит EN-версию.

**Что сделать.** Убрать автоматический редирект по Accept-Language со всех URL, включая главную, и заменить его баннером. Например: «Похоже, вы говорите по-русски. Перейти на русскую версию?» Из поиска пользователь и так попадает на свою версию: благодаря hreflang Google показывает русскоязычным `/ru/...`. Баннер нужен только для прямых заходов. Риска для SEO нет, это совпадает с рекомендацией Google, hreflang уже настроен корректно.

<a id="p2"></a>

## 2. Страница /pricing: подгрузка, случайный порядок, дубли 🔴

**Доказательства: curl** — в исходном HTML только 9 зон:
`.com, .airforce, .bet, .biz, .audio, .digital, .io, .academy, .tech`

**Доказательства: Chrome.**
- После загрузки показано **9 других зон** в другом порядке: `.academy, .ai, .info, .click, .dating, .com, .jewelry, .diamonds, .agency`. Порядок меняется от загрузки к загрузке.
- Остальные зоны раскрываются кнопкой «Показать ещё 9». Это `<button>`; URL не меняется, отдельных страниц или ссылок нет.
- После подгрузки в списке **появились дубли**: `.academy`, `.diamonds`, `.jewelry` встречаются дважды.
- Кнопки «Купить» — `<button>` без href, ссылок на отдельные зоны нет.
- Блок цен на главной тоже, судя по всему, случайный: при разборе HTML на главной были одни зоны, на /pricing — другие.

**Чем это опасно:**
- Поисковик видит только 9 случайных зон, и при каждом обходе разные. Контент страницы нестабилен.
- Цены на большинство зон не индексируются. Ранжироваться по «цена домена .xxx» нечему.
- Дубли в списке — баг интерфейса, который бьёт по доверию.

**Что сделать:**
- отдавать полный список в SSR, отсортированный и стабильный (по алфавиту или популярности), с фильтром и поиском на клиенте;
- исправить дубли;
- в перспективе сделать карточку каждой зоны ссылкой на `/domains/{tld}` (см. п. 8).

<a id="p3"></a>

## 3. Ошибка 500 на служебных URL 🟠

**Доказательства: curl.**

```
/favicon.ico           -> 500
/apple-touch-icon.png  -> 500
/manifest.json         -> 500
/site.webmanifest      -> 500
/llms.txt              -> 500   (тело ответа: "Internal Server Error")
/index.html            -> 500
/Pricing               -> 404   (регистр учитывается — нормально)
/blog, /privacy        -> 404   (нормально)
```

**Чем это опасно:**
- Браузеры и Яндекс запрашивают `/favicon.ico` автоматически.
- Если 500 приходят массово, краулер снижает скорость обхода сайта.
- Яндекс берёт фавикон для выдачи — без него сниппет хуже.

Похоже, обработчик `[locale]` в Next.js воспринимает эти пути как локаль и падает.

**Что сделать:**
- положить `favicon.ico` и `apple-touch-icon.png` в `public/`;
- исключить статические пути из middleware;
- все несуществующие URL должны отдавать 404, а не 500.

<a id="p4"></a>

## 4. У www.dotberry.com нет DNS-записи 🟠

**Доказательства.**

```
curl https://www.dotberry.com/  -> Could not resolve host: www.dotberry.com
nslookup dotberry.com           -> 109.206.182.153
```

**Чем это опасно:** часть пользователей и ссылок ведёт на `www.` — они получают ошибку, а ссылочный вес теряется.

**Что сделать:** добавить DNS-запись `www`, выпустить на неё SSL и настроить 301-редирект на `https://dotberry.com`.

<a id="p5"></a>

## 5. Временные редиректы /en/* 🟠

**Доказательства.**

```
https://dotberry.com/en          -> 307 https://dotberry.com/
https://dotberry.com/en/pricing  -> 307 https://dotberry.com/pricing
```

**Что работает правильно:**

```
http://dotberry.com/              -> 301 https://dotberry.com/
https://dotberry.com/pricing/     -> 308 /pricing   (слеш на конце убирается)
https://dotberry.com/ru/          -> 308 /ru
?utm_source=x, ?x=1               -> 200, canonical без параметров; в robots.txt Disallow: /?
```

**Что сделать:** для `/en/*` отдавать 308 или 301 — это постоянный дубль.

<a id="p6"></a>

## 6. Объём JavaScript 🟠

**Доказательства: Chrome**, `/ru`, Resource Timing (encodedBodySize).

```
JS   ~823 KB, 23 скрипта
CSS  ~27 KB
HTML ~42 KB (сжатый)
DOM  632 узла
Сторонние сервисы: mc.yandex.ru (Яндекс.Метрика), openpanel.njwso.com (OpenPanel)
```

**Вывод.** Для сайта из пяти страниц JS тяжёлый. Это бьёт по INP и времени до интерактивности на мобильных.

**Что сделать:** проверить бандл через `@next/bundle-analyzer`, перевести статичные блоки в серверные компоненты, отложить загрузку аналитики.

<a id="p7"></a>

## 7. Заголовки безопасности 🟡

**Доказательства.** В ответе `https://dotberry.com/` нет ни одного из заголовков: `Strict-Transport-Security`, `X-Frame-Options`, `X-Content-Type-Options`, `Content-Security-Policy`, `Referrer-Policy`, `Permissions-Policy`.

Сертификат: Let's Encrypt (CN=dotberry.com), действует до 01.01.2027 — ок.

Для SEO это не прямой фактор, но для сервиса, который продаёт «приватность и безопасность», это вопрос доверия. Аудитория проверяет такие вещи, например через securityheaders.com.

**Что сделать:** включить HSTS и базовый набор заголовков в nginx.

<a id="p8"></a>

## 8. Ссылки вместо кнопок 🟡

**Доказательства: Chrome.**
- кнопки «Купить» на /pricing — `<button>` без href;
- быстрые фильтры `.biz .com .io …` — `<button>`;
- поиск на /domains работает, но URL после поиска не меняется (`/ru/domains` без `?q=`), результаты рисуются только на клиенте.

Сейчас это не критично: индексировать результаты поиска не нужно. Но при создании TLD-страниц все карточки зон и фильтры должны стать обычными ссылками `<a href="/domains/com">`, иначе краулер их не найдёт.

<a id="p9"></a>

## 9. Доступность (а11y), влияющая на SEO 🟡

**Доказательства.**
- `<img alt="">` — у обоих изображений на главной (curl и Chrome);
- у поля поиска нет `<label>`, есть только `aria-label="Поиск домена"` — допустимо;
- ссылка-логотип оформлена корректно: `aria-label="Dotberry home"`.

<a id="p10"></a>

## 10. Консоль браузера 🟡

**Доказательства: Chrome.** Ошибок (error) нет. Есть повторяющееся предупреждение React — больше 20 раз за загрузку:

```
Tooltip is changing from controlled to uncontrolled. Components should not switch
from controlled to uncontrolled (or vice versa).
```

На SEO не влияет, но это технический долг. Передать разработчику.

<a id="p11"></a>

## 11. Sitemap и robots.txt 🟡

**Доказательства:** снапшоты в `audit/raw/`.
- robots.txt: `User-Agent: * / Disallow: /? / Sitemap: …` — корректно.
- sitemap.xml: 5 URL с hreflang-альтернативами — корректно.
  - У всех страниц одинаковый `lastmod` 2026-11-02T14:36:16Z — похоже на дату сборки, а не на реальное изменение.
  - `priority`: /terms = 0.8 выше, чем /how-it-works = 0.7.

---

## Что работает правильно ✅

| Проверка | Результат |
|---|---|
| TTFB | 0.3–0.45 с на всех страницах, норма ≤ 0.8 с (curl, одна точка; подтвердить через PSI/CrUX) |
| HTTP → HTTPS | 301 |
| Слеш на конце | 308 на адрес без слеша |
| Canonical | самоссылающийся на всех страницах, без параметров |
| hreflang | en / ru / x-default: в HTTP-заголовке `Link`, в `<head>` и в sitemap — согласованы |
| `<html lang>` | `en` на EN, `ru` на RU |
| 404 | честный код 404 + `noindex` (EN и RU) |
| SSR | контент есть в исходном HTML, доступен без JS |
| Googlebot / YandexBot | 200, полный HTML (177 КБ) |
| Сжатие | gzip |
| CLS | 0 |
| Ошибки JS | нет |
| SSL | валиден |
| Schema | Organization, WebSite, FAQPage (главная, /how-it-works) |

## Следующие шаги

1. Получить доступ к GSC и Яндекс.Вебмастеру: отчёты по индексации, ошибкам обхода, Core Web Vitals.
2. Прогнать PageSpeed Insights (mobile/desktop) для `/`, `/ru`, `/pricing`, `/domains`.
3. Передать разработчику пп. 1–5 как ТЗ — отдельные задачи в `tasks/`.
4. Повторить замеры после исправлений.
