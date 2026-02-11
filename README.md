# Расчёт метрик сервиса доставки еды

Необходимо разработать дашборд, который с разных сторон отразит состояние клиентской базы в городе Саранске. На основе этого дашборда необходимо составить аналитический отчёт.

---

[**Ссылка на дашборд, разработанный в Yandex DataLens**](https://datalens.yandex/lcna5fnedwgq5)

---

**Задачи:**
1. Подготовить QL-запросы в DataLens для визуализации следующих метрик:
    * DAU (от англ. daily active users) — количество активных пользователей за день.
    * Conversion Rate — коэффициент конверсии.
    * Средний чек — средняя сумма покупки на пользователя.
    * LTV (от англ. lifetime value) — совокупная ценность клиента за период.
    * Retention Rate — коэффициент удержания пользователей.
2. Создать визуализации в DataLens:
    * Для каждой метрики подготовить график для наглядного анализа. Например, линейные тренды, диаграммы или таблицы.
    * Убедиться, что визуализации отражают динамику показателей, а также позволяют выявлять закономерности и аномалии.
    * На основе этих запросов создать дашборд, который включает понятные графики для указанных выше метрик.
3. Проанализировать результаты визуализаций и подготовить выводы в аналитической записке для старшего аналитика. Она должна включать выводы по каждой метрике за период с начала мая до конца июня. Что отразить в записке:
    * Увеличивается или снижается DAU?
    * Наблюдаются ли аномалии в конверсии?
    * Как меняется средний чек с течением времени?
    * Как меняется Retention Rate в зависимости от периода?

## Описание данных
Продукт, который вы будете анализировать в проекте, — сервис доставки еды «Всё.из.кафе». Данные продукта состоят из нескольких таблиц:
- `analytics_events` — журнал аналитических событий, или логи. Напомним, что логами называют записи, которые фиксируют действия пользователей в рамках работы сервиса. Сюда попадают данные о посещении пользователем страниц продукта и покупках.
- `advertisement_budgets` — рекламные затраты по дням и каналам привлечения.
- `partners` — справочник партнёрских сетей и их ресторанов.
- `dishes` — справочник блюд.
- `cities` — справочник городов.

**Таблица `analytics_events`**

В таблице хранятся данные о событиях в период с 30.04.2021 по 02.07.2021. Поля таблицы:
- `visitor_uuid` — идентификатор посетителя. Это идентификатор, который система присваивает любому новому пользователю независимо от того, регистрировался ли он в продукте.
- `user_id` — идентификатор зарегистрированного пользователя. Присваивается посетителю после создания учётной записи: ввода логина, пароля, адреса доставки и контактных данных.
- `device_type` — тип платформы, с которой посетитель зашёл в продукт.
- `city_id` — город, из которого посетитель зашёл в сервис.
- `age` — возрастная группа пользователя. Она указывается при регистрации.
- `source` — рекламный источник привлечения посетителя.
- `first_date` — дата первого посещения продукта.
- `visit_id` — уникальный идентификатор сессии.
- `event` — название аналитического события.
- `datetime` — дата и время события.
- `log_date` — дата события.
- `rest_id` — уникальный идентификатор сети, к которой принадлежит ресторан. Заполняется для заказов, карточек ресторанов и блюд.
- `object_id` — уникальный идентификатор блюда. Заполняется для заказов и карточек блюд.
- `listing_id` — уникальный идентификатор блюда в листинге. Листингом называется набор блюд, которые система рекомендует пользователю при просмотре страницы ресторана. Заполняется только для событий rest_page.
- `position` — позиция блюда в листинге. Чем меньше номер, тем ближе блюдо к началу страницы. Заполняется только для событий rest_page.
- `order_id` — уникальный идентификатор заказа.
- `revenue` — выручка от заказа в рублях. Это та сумма, которую пользователь видит при оплате.
- `delivery` — стоимость доставки в рублях.
- `commission` — комиссия, которую «Всё.из.кафе» берёт с выручки ресторана.

В поле `event` таблицы также хранятся данные о нескольких типах аналитических событий:
- `main_page` — посещение главной страницы приложения.
- `authorization` — ввод логина и пароля, авторизация.
- `rest_page` — просмотр страницы ресторана.
- `object_page` — просмотр карточки блюда.
- `order` — оплата заказа.

**Таблица `advertisement_budgets`**

В таблице хранятся данные о ежедневных затратах на рекламу. Поля таблицы:
- `source` — рекламный источник.
- `date` — дата рекламных затрат.
- `budget` — дневной бюджет в рублях.

**Таблица `partners`**

Это справочник партнёрских сетей и их ресторанов. Поля таблицы:
- `rest_id` — уникальный идентификатор сети, к которой принадлежит ресторан (одна сеть может быть представлена в нескольких городах).
- `chain` — название сети, к которой принадлежит ресторан.
- `type` — тип кухни.
- `city_id` — город, в котором находится ресторан.
- `commission` — комиссия в процентах, которую «Всё.из.кафе» берёт с выручки ресторана.

**Таблица `dishes`**

Это справочник блюд, доступных в партнёрских ресторанах. Поля таблицы:
- `object_id` — уникальный идентификатор блюда.
- `name` — название блюда.
- `spicy` — логический признак острых блюд. 1 — блюдо острое.
- `fish` — логический признак рыбных блюд. 1 — блюдо содержит морепродукты.
- `meat` — логический признак мясных блюд. 1 — блюдо содержит мясо.
- `rest_idv — уникальный идентификатор ресторана, из которого можно заказать блюдо.

**Таблица `cities`**

Справочник населённых пунктов, в которых можно пользоваться продуктом. Поля таблицы:
- `city_id` — уникальный идентификатор населённого пункта.
- `city_name` — название населённого пункта.

---

**1. Расчёт DAU.**
Необходимо рассчитать ежедневное количество активных зарегистрированных клиентов (user_id) за май и июнь 2021 года в городе Саранске. Критерием активности клиента считается размещение заказа. Это позволит оценить эффективность вовлечения клиентов в ключевую бизнес-цель — совершение покупки. Сортируем результат по дате события в возрастающем порядке.
```
SELECT 
    log_date,
    COUNT(DISTINCT user_id) AS DAU
FROM analytics_events
WHERE event = 'order' 
  AND user_id IS NOT NULL 
  AND city_id = (SELECT city_id FROM cities WHERE city_name = 'Саранск') 
  AND log_date BETWEEN '2021-05-01' AND '2021-06-30'
GROUP BY log_date
ORDER BY log_date ASC;
```

**2. Расчёт Conversion Rate.**
Определим активность аудитории: как часто зарегистрированные пользователи переходят к размещению заказа, будет ли одинаковым этот показатель по дням или видны сезонные колебания в поведении пользователей. Рассчитаем конверсию зарегистрированных пользователей, которые посещают приложение, в активных клиентов. Конверсия должна быть рассчитана за каждый день в мае и июне 2021 года для клиентов из Саранска.
Сортируем результат по дню активности в возрастающем порядке. 
```
SELECT log_date,
       ROUND((COUNT(DISTINCT user_id) FILTER (WHERE event = 'order')) / COUNT(DISTINCT user_id)::numeric, 2) AS CR
FROM analytics_events
JOIN cities ON analytics_events.city_id = cities.city_id
WHERE log_date BETWEEN '2021-05-01' AND '2021-06-30'
    AND city_name = 'Саранск'
GROUP BY log_date
ORDER BY log_date;
```

**3. Расчёт среднего чека.**
В рамках этой задачи предстоит рассчитать средний чек активных клиентов в Саранске в мае и в июне. Мы анализируем средний чек сервиса доставки, а не ресторанов. Значит, необходимо вычислить его как среднее значение комиссии со всех заказов за месяц. Таким образом, для корректного расчёта метрики вычислим общий размер комиссии и количество заказов. Разделив сумму комиссии на количество заказов, рассчитаем величину среднего чека за месяц. Результат отсортируем по полю с месяцем в возрастающем порядке.
```
-- Рассчитываем величину комиссии с каждого заказа, отбираем заказы по дате и городу
WITH orders AS
    (SELECT *,
            revenue * commission AS commission_revenue
     FROM analytics_events
     JOIN cities ON analytics_events.city_id = cities.city_id
     WHERE revenue IS NOT NULL
         AND log_date BETWEEN '2021-05-01' AND '2021-06-30'
         AND city_name = 'Саранск')

SELECT CAST(DATE_TRUNC('month', log_date) AS date) AS "Месяц",
    COUNT(DISTINCT order_id) AS "Количество заказов",
    ROUND(SUM(commission_revenue)::numeric, 2) AS "Сумма комиссии",
    ROUND(SUM(commission_revenue)::numeric / COUNT(DISTINCT order_id)::numeric, 2) AS "Средний чек"
FROM orders
GROUP BY "Месяц"
ORDER BY "Месяц";
```

**4. Расчёт LTV ресторанов.**
Определим три ресторана из Саранска с наибольшим LTV с начала мая до конца июня. Как правило, LTV рассчитывается для пользователя приложения. Однако клиентами для сервиса доставки будут и рестораны, как и пользователи, которые делают заказы. В рамках этой задачи посчитаем LTV как суммарную комиссию, которая была получена от заказов в ресторане за эти два месяца. Результат отсортируем по полю LTV в убывающем порядке.
```
-- Рассчитываем величину комиссии с каждого заказа, отбираем заказы по дате и городу
WITH orders AS
    (SELECT analytics_events.rest_id,
            analytics_events.city_id,
            revenue * commission AS commission_revenue
     FROM analytics_events
     JOIN cities ON analytics_events.city_id = cities.city_id
     WHERE revenue IS NOT NULL
         AND log_date BETWEEN '2021-05-01' AND '2021-06-30'
         AND city_name = 'Саранск')

SELECT 
    o.rest_id,
    p.chain AS "Название сети",
    p.type AS "Тип кухни",
    ROUND(SUM(commission_revenue)::numeric, 2) AS LTV
FROM orders o
JOIN partners p ON o.rest_id = p.rest_id AND o.city_id = p.city_id 
GROUP BY 1, 2, 3
ORDER BY LTV DESC
LIMIT 3;
```

**5. Расчёт LTV ресторанов — самые популярные блюда.**
Наибольший LTV с большим отрывом у двух ресторанов Саранска: «Гурманское Наслаждение» и «Гастрономический Шторм». Теперь нужно узнать, сколько LTV принесли пять самых популярных блюд этих ресторанов. При этом популярных блюд должно быть всего пять, а не по пять из каждого ресторана. Необходимо проанализировать данные о ресторанах и их блюдах, чтобы определить вклад самых популярных блюд из двух ресторанов Саранска — «Гурманское Наслаждение» и «Гастрономический Шторм» — в общий показатель LTV. Для этого нужно выбрать пять блюд с максимальным LTV за весь рассматриваемый период, то есть за май — июнь, из этих двух ресторанов. Для каждого блюда требуется вывести название ресторана, название блюда, признаки того, является ли блюдо острым, рыбным или мясным, а также значение LTV, округлённое до копеек. Результаты отсортируем по значению LTV в порядке убывания. 
```
-- Рассчитываем величину комиссии с каждого заказа, отбираем заказы по дате и городу
WITH orders AS
    (SELECT analytics_events.rest_id,
            analytics_events.city_id,
            analytics_events.object_id,
            revenue * commission AS commission_revenue
     FROM analytics_events
     JOIN cities ON analytics_events.city_id = cities.city_id
     WHERE revenue IS NOT NULL
         AND log_date BETWEEN '2021-05-01' AND '2021-06-30'
         AND city_name = 'Саранск'), 

-- Рассчитываем два ресторана с наибольшим LTV 
top_ltv_restaurants AS
    (SELECT orders.rest_id,
            chain,
            type,
            ROUND(SUM(commission_revenue)::numeric, 2) AS LTV
     FROM orders
     JOIN partners ON orders.rest_id = partners.rest_id AND orders.city_id = partners.city_id
     GROUP BY 1, 2, 3
     ORDER BY LTV DESC
     LIMIT 2)

SELECT 
    p.chain AS "Название сети",
    d.name AS "Название блюда",
    d.spicy,
    d.fish,
    d.meat,
    ROUND(SUM(o.commission_revenue)::numeric, 2) AS LTV
FROM orders o
JOIN top_ltv_restaurants t ON o.rest_id = t.rest_id
JOIN partners p ON o.rest_id = p.rest_id AND o.city_id = p.city_id
JOIN dishes d ON o.object_id = d.object_id AND o.rest_id = d.rest_id
GROUP BY p.chain, d.name, d.spicy, d.fish, d.meat
ORDER BY LTV DESC
LIMIT 5;
```

**6. Расчёт Retention Rate.**
В рамках этой задачи предстоит определить показатель возвращаемости: какой процент пользователей возвращается в приложение в течение первой недели после регистрации и в какие дни. Рассчитаем показатель Retention Rate в первую неделю для всех новых пользователей в Саранске.
Напомним, что в проекте мы анализируем данные за май и июнь, и для корректного расчёта недельного Retention Rate нужно, чтобы с момента первого посещения прошла хотя бы неделя. Поэтому для этой задачи ограничим дату первого посещения продукта, выбрав промежуток с начала мая по 24 июня. Retention Rate считаем по любой активности пользователей, а не только по факту размещения заказа.
В данных могут встречаться дубликаты по полю user_id, поэтому для корректного расчёта используем условие log_date >= first_date. Результаты сортируем по полю day_since_install в порядке возрастания. 
```
-- Рассчитываем новых пользователей по дате первого посещения продукта
WITH new_users AS
    (SELECT DISTINCT first_date,
                     user_id
     FROM analytics_events
     JOIN cities ON analytics_events.city_id = cities.city_id
     WHERE first_date BETWEEN '2021-05-01' AND '2021-06-24'
         AND city_name = 'Саранск'),

-- Рассчитываем активных пользователей по дате события
active_users AS
    (SELECT DISTINCT log_date,
                     user_id
     FROM analytics_events
     JOIN cities ON analytics_events.city_id = cities.city_id
     WHERE log_date BETWEEN '2021-05-01' AND '2021-06-30'
         AND city_name = 'Саранск'),

daily_retention AS
    (SELECT new_users.user_id,
            first_date,
            log_date::date - first_date::date AS day_since_install
     FROM new_users
     JOIN active_users ON new_users.user_id = active_users.user_id
     AND log_date >= first_date)
SELECT day_since_install,
       COUNT(DISTINCT user_id) AS retained_users,
       ROUND((1.0 * COUNT(DISTINCT user_id) / MAX(COUNT(DISTINCT user_id)) OVER (ORDER BY day_since_install))::numeric, 2) AS retention_rate
FROM daily_retention
WHERE day_since_install < 8
GROUP BY day_since_install
ORDER BY day_since_install;
```

**7. Сравнение Retention Rate по месяцам.**
Разделим пользователей на две когорты по месяцу первого посещения продукта. Таким образом сравним Retention Rate этих когорт между собой. Результаты отсортируем по полям "Месяц" и day_since_install в порядке возрастания. 
```
-- Рассчитываем новых пользователей по дате первого посещения продукта
WITH new_users AS
    (SELECT DISTINCT first_date,
                     user_id
     FROM analytics_events
     JOIN cities ON analytics_events.city_id = cities.city_id
     WHERE first_date BETWEEN '2021-05-01' AND '2021-06-24'
         AND city_name = 'Саранск'),

-- Рассчитываем активных пользователей по дате события
active_users AS
    (SELECT DISTINCT log_date,
                     user_id
     FROM analytics_events
     JOIN cities ON analytics_events.city_id = cities.city_id
     WHERE log_date BETWEEN '2021-05-01' AND '2021-06-30'
         AND city_name = 'Саранск'),

-- Соединяем таблицы с новыми и активными пользователями
daily_retention AS
    (SELECT new_users.user_id,
            new_users.first_date,
            log_date::date - first_date::date AS day_since_install
     FROM new_users
     JOIN active_users ON new_users.user_id = active_users.user_id
     AND log_date >= first_date)

-- Агрегация по месяцу и дню с расчётом Retention Rate
SELECT DISTINCT CAST(DATE_TRUNC('month', first_date) AS date) AS "Месяц",
                day_since_install,
                COUNT(DISTINCT user_id) AS retained_users,
                ROUND((1.0 * COUNT(DISTINCT user_id) / MAX(COUNT(DISTINCT user_id)) OVER (PARTITION BY CAST(DATE_TRUNC('month', first_date) AS date) ORDER BY day_since_install))::numeric, 2) AS retention_rate
FROM daily_retention
WHERE day_since_install < 8
GROUP BY "Месяц", day_since_install
ORDER BY "Месяц", day_since_install;
```
