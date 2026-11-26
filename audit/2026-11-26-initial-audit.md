# Первичный SEO-аудит dotberry.com

**Дата:** 2026-11-26
**Метод:** ручной разбор HTML (curl), robots.txt, sitemap.xml, заголовков ответа. Без доступа к GSC / Яндекс.Вебмастеру / аналитике.

---

## 1. Что за проект

**Dotberry** — анонимный доменный регистратор (реселлер), **без KYC**, оплата **только криптовалютой**.

| Параметр | Значение |
|---|---|
| Юрлицо | Dotberry Ltd., Гонконг (из /terms) |
| УТП | Регистрация только по email, без имени/телефона/документов; WHOIS-приватность, базовый DNS и переадресация почты бесплатно; мгновенная активация после подтверждения в блокчейне |
| Оплата | XMR, BTC, USDT (в мета-описании ещё TON) |
| Модель WHOIS | В реестре указаны корпоративные данные Dotberry (proxy-администратор), пользователь — юридический владелец по ToS |
| Ограничения | Нет исходящего трансфера к другим регистраторам; нет автопродления (только пополнение баланса и ручное продление); запрещены фишинг, малварь, спам, CSAM |
| Языки | EN (основной, `/`) и RU (`/ru/...`) |
| Ниша / ЦА | Privacy-ориентированные пользователи, крипто-аудитория, веб-мастера, арбитражники, проекты, которым важна анонимность владельца |

### Цены (примеры с сайта, регистрация / продление)
- .com — $12.78 / $12.78
- .io — $23.54 / $58.85
- .biz — $1.65 / $20.82
- .digital — $2.75 / $39.08
- .bet — $10.84 / $26.14
- .help — $1.09, .store / .website — $2.75, .info — $4.39, .cafe / .city — $4.95
- Восстановление ~ $111–330, grace period 30 дней, restore 30 дней

Важно: дешёвая первая регистрация и сильно дороже продление на многих зонах — это нужно учитывать в контенте (честное сравнение цен = доверие).

---

## 2. Технический стек и инфраструктура

- **Next.js** (App Router, ISR: `x-nextjs-cache`, `s-maxage=300, stale-while-revalidate`), за **nginx**.
- Локализация через middleware (`NEXT_LOCALE` cookie, rewrite `/` → `/en`).
- `http://` → 301 на `https://` ✅; у `www.dotberry.com` **нет DNS-записи** (уточнено в [TECH-SEO-AUDIT.md](../TECH-SEO-AUDIT.md), п. 5). Canonical на страницах указывает на `https://dotberry.com` ✅.
- gzip ✅. Аналитика: **Яндекс.Метрика** (GA/GTM не обнаружены).
- Шрифты — self-hosted woff2 через next/font ✅.

## 3. Структура сайта (индексируемые страницы)

В sitemap всего **5 URL × 2 языка = 10 страниц**:

| URL | Title | H1 | Слов | Schema |
|---|---|---|---|---|
| `/` | Buy Domain Anonymously with Crypto — No KYC \| Dotberry | Your domain. Your business only. | ~690 | Organization, WebSite, FAQPage |
| `/domains` | Anonymous Domain Search — Check & Buy \| Dotberry | Domains Search. | ~150 | Organization, WebSite |
| `/pricing` | Domain Pricing — Cost of Anonymous Registration \| Dotberry | TLD List. | ~350 (9 TLD в SSR) | Organization, WebSite |
| `/how-it-works` | How to Buy a Domain Anonymously with Crypto & Benefits \| Dotberry | How It Works. | ~630 | Organization, WebSite, FAQPage |
| `/terms` | Terms of Service, Privacy Policy and Abuse Policy \| Dotberry | Terms of Service. | ~3300 | Organization, WebSite |

RU-версия: `/ru` — «Купить домен анонимно и за криптовалюту — без верификации | Dotberry», H1 «Ваш домен. Только ваше дело.» и т.д. — полноценный перевод.

## 4. Что сделано хорошо ✅

- Уникальные title/description на каждой странице, ключ «buy domain anonymously / crypto / no KYC» в title главной.
- hreflang (en / ru / x-default) в трёх местах: HTTP-заголовок `Link`, `<head>`, sitemap — согласованы.
- Canonical на каждой странице, самоссылающийся.
- OG/Twitter-разметка, OG-картинка 1200×630.
- JSON-LD: Organization + WebSite + FAQPage (главная, how-it-works).
- 404 отдаёт честный код 404 и `noindex`.
- SSR — контент доступен без JS.
- robots.txt открыт, указан sitemap, закрыты URL с параметрами (`Disallow: /?`).

## 5. Проблемы и точки роста ⚠️

### Критичные / высокий приоритет
1. **Очень маленький сайт — 5 страниц.** Нет посадочных под коммерческие запросы:
   - нет страниц под каждую доменную зону (`/domains/com`, `/tld/io` и т.д.) — главный упущенный трафик («buy .com domain with bitcoin», «anonymous .io domain», «купить домен .com за крипту»);
   - нет страниц под способы оплаты (`buy domain with Monero / Bitcoin / USDT / TON`);
   - нет страниц под интенты/кейсы (no-KYC domain, private WHOIS, offshore domain registrar, domain for VPN/crypto project и т.п.);
   - нет блога / базы знаний (гайды, сравнения).
2. **Скорость ответа сервера:** TTFB главной ~3 с, `/domains` ~7 с (при первом запросе, `x-nextjs-cache: STALE`). Нужен замер CWV (PageSpeed / CrUX) и проверка кэша ISR.
3. **Тонкие страницы:** `/domains` — ~150 слов, по сути только форма поиска; H1 «Domains Search.» не содержит ключа. `/pricing` в SSR выводит только 9 зон — остальной список, видимо, подгружается на клиенте/не индексируется.
4. **Слабые H1** на внутренних: «Domains Search.», «TLD List.», «How It Works.» — без поисковых ключей.

### Средний приоритет
5. **Несоответствие оплаты:** в мета-описании «BTC, USDT, TON, XMR», в контенте — только XMR, BTC, USDT. Уточнить у клиента, принимается ли TON.
6. **Schema:** нет `Product`/`Offer` с ценами на /pricing, нет `BreadcrumbList`, у Organization нет `sameAs`, `contactPoint`, адреса.
7. **E-E-A-T / доверие:** нет страницы About, контактов, соцсетей (Telegram/X), отзывов, статуса/аккредитации (через какого регистратора ICANN работают). Для ниши «анонимность + крипта» доверие — ключевой фактор конверсии и ранжирования.
8. **Изображения без alt** (alt="").
9. **`/llms.txt` отдаёт 500** — лучше 404 или создать файл (для AI-поиска).
10. **Юридические разделы** (Privacy, AUP) — якоря на одной странице `/terms#...`; ок, но отдельные URL полезнее для ссылок и ссылочного профиля.
11. **Нет трансфера наружу** — частое возражение; в контенте нужно аккуратно отрабатывать (сейчас FAQ уже объясняет).

### Низкий приоритет
12. Приоритеты в sitemap нелогичны (`/terms` 0.8 > `/how-it-works` 0.7), `lastmod` одинаковый у всех.
13. 404-страница с title просто «Dotberry».
14. Проверить `www.dotberry.com` → 301 на без-www.
15. Двойная точка в заголовке «Frequently asked questions..» (декоративная точка + текст).

## 6. Конкуренты (для дальнейшего анализа)

Ниша «anonymous / no-KYC domain registration, pay with crypto»:
Njalla, OrangeWebsite, 1984 Hosting, Incognet, NiceNIC, Namecheap (принимает BTC), Porkbun, Epik, а также RU-сегмент — анонимные регистраторы / реселлеры с оплатой криптой. Задача: собрать их структуру (TLD-страницы, блог), ссылочный профиль и семантику.

## 7. Нужно получить от клиента

- Доступы: Google Search Console, Яндекс.Вебмастер, Яндекс.Метрика, (GA4 при наличии).
- Доступ к репозиторию / CMS или контакт разработчика (Next.js) — для внедрения правок.
- Подтверждение: TON принимается? Какие TLD приоритетны (маржа)? Есть ли ограничения по тематикам контента / рекламным площадкам?
- Целевые рынки/гео и приоритет EN vs RU.
- Соцсети / Telegram-канал, брендовые упоминания.
