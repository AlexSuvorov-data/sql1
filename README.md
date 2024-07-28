1. Отобразите все записи из таблицы company по компаниям, которые закрылись.


SELECT *
FROM company
WHERE status = 'closed';

3. Отобразите количество привлечённых средств для новостных компаний США. Используйте данные из таблицы company. Отсортируйте таблицу по убыванию значений в поле funding_total.

SELECT 
        funding_total
FROM company
WHERE country_code = 'USA'
    AND category_code = 'news'
ORDER BY funding_total DESC;

5. Найдите общую сумму сделок по покупке одних компаний другими в долларах. Отберите сделки, которые осуществлялись только за наличные с 2011 по 2013 год включительно.
SELECT SUM(price_amount)
FROM acquisition
WHERE term_code = 'cash'
        AND EXTRACT(YEAR FROM CAST(acquired_at AS timestamp)) IN (2011, 2012, 2013);
   
7. Отобразите имя, фамилию и названия аккаунтов людей в поле network_username, у которых названия аккаунтов начинаются на 'Silver'.

SELECT first_name,
        last_name,
        network_username
FROM people
WHERE network_username LIKE 'Silver%';

5. Выведите на экран всю информацию о людях, у которых названия аккаунтов в поле network_username содержат подстроку 'money', а фамилия начинается на 'K'.

SELECT *
FROM people
WHERE network_username LIKE '%money%'
    AND last_name LIKE 'K%';
6. Для каждой страны отобразите общую сумму привлечённых инвестиций, которые получили компании, зарегистрированные в этой стране. Страну, в которой зарегистрирована компания, можно определить по коду страны. Отсортируйте данные по убыванию суммы.

SELECT country_code,
        SUM(funding_total) as total
FROM company
GROUP BY country_code
ORDER BY total DESC;

7. Составьте таблицу, в которую войдёт дата проведения раунда, а также минимальное и максимальное значения суммы инвестиций, привлечённых в эту дату.
Оставьте в итоговой таблице только те записи, в которых минимальное значение суммы инвестиций не равно нулю и не равно максимальному значению.

SELECT funded_at,
        MIN(raised_amount),
        MAX(raised_amount)
FROM funding_round
GROUP BY funded_at
HAVING MIN(raised_amount) <> 0
        AND MIN(raised_amount) <> MAX(raised_amount);

8. Создайте поле с категориями:

    Для фондов, которые инвестируют в 100 и более компаний, назначьте категорию high_activity.
    Для фондов, которые инвестируют в 20 и более компаний до 100, назначьте категорию middle_activity.
    Если количество инвестируемых компаний фонда не достигает 20, назначьте категорию low_activity.

Отобразите все поля таблицы fund и новое поле с категориями.

SELECT
    *, 
    CASE
        WHEN invested_companies >= 100 THEN 'high_activity'
        WHEN invested_companies >= 20 THEN 'middle_activity'
        ELSE 'low_activity'
    END
FROM fund;

9. Для каждой из категорий, назначенных в предыдущем задании, посчитайте округлённое до ближайшего целого числа среднее количество инвестиционных раундов, в которых фонд принимал участие. Выведите на экран категории и среднее число инвестиционных раундов. Отсортируйте таблицу по возрастанию среднего.

   WITH
table_1 AS (SELECT *,
                  CASE
                       WHEN invested_companies>=100 THEN 'high_activity'
                       WHEN invested_companies>=20 THEN 'middle_activity'
                       ELSE 'low_activity'
                   END AS activity
            FROM fund)
SELECT activity,
        ROUND(AVG(investment_rounds)) AS avg_rounds
FROM table_1
GROUP BY activity
ORDER BY avg_rounds ASC;

10. Проанализируйте, в каких странах находятся фонды, которые чаще всего инвестируют в стартапы. 
Для каждой страны посчитайте минимальное, максимальное и среднее число компаний, в которые инвестировали фонды этой страны, основанные с 2010 по 2012 год включительно. Исключите страны с фондами, у которых минимальное число компаний, получивших инвестиции, равно нулю. 
Выгрузите десять самых активных стран-инвесторов: отсортируйте таблицу по среднему количеству компаний от большего к меньшему. Затем добавьте сортировку по коду страны в лексикографическом порядке.

SELECT country_code,
            MIN(invested_companies),
            MAX(invested_companies),
            AVG(invested_companies) AS avg_companies
    FROM fund
    WHERE EXTRACT(YEAR FROM CAST(founded_at AS timestamp)) IN (2010, 2011, 2012)
    GROUP BY country_code
    HAVING MIN(invested_companies) <> 0
    ORDER BY avg_companies DESC, country_code ASC
    LIMIT 10;  

11. Отобразите имя и фамилию всех сотрудников стартапов. Добавьте поле с названием учебного заведения, которое окончил сотрудник, если эта информация известна.

SELECT p.first_name,
    p.last_name,
    e.instituition
FROM education AS e 
    RIGHT JOIN people AS p ON e.person_id = p.id;

12. Для каждой компании найдите количество учебных заведений, которые окончили её сотрудники. Выведите название компании и число уникальных названий учебных заведений. Составьте топ-5 компаний по количеству университетов.

SELECT c.name,
        COUNT(DISTINCT instituition) AS num_inst
FROM company AS c
    INNER JOIN people AS p ON c.id = p.company_id
    INNER JOIN education AS e ON p.id = e.person_id
GROUP BY c.name    
ORDER BY num_inst DESC
LIMIT 5;

13. Составьте список с уникальными названиями закрытых компаний, для которых первый раунд финансирования оказался последним.

SELECT DISTINCT c.name
FROM company AS c
WHERE status = 'closed'
    AND id IN 
    (SELECT DISTINCT company_id
    FROM funding_round
    WHERE is_first_round = 1
        AND is_last_round = 1);

14. Составьте список уникальных номеров сотрудников, которые работают в компаниях, отобранных в предыдущем задании.

 SELECT DISTINCT id
FROM people
WHERE company_id IN
    (SELECT DISTINCT c.id
    FROM company AS c
    WHERE status = 'closed'
        AND id IN 
        (SELECT DISTINCT company_id
        FROM funding_round
        WHERE is_first_round = 1
            AND is_last_round = 1));

15. Составьте таблицу, куда войдут уникальные пары с номерами сотрудников из предыдущей задачи и учебным заведением, которое окончил сотрудник.


SELECT p.id,
        e.instituition
FROM people AS p 
    INNER JOIN education AS e ON p.id = e.person_id
WHERE p.id IN  (SELECT DISTINCT id
            FROM people
            WHERE company_id IN
                (SELECT DISTINCT c.id
                FROM company AS c
                WHERE status = 'closed'
                    AND id IN 
                    (SELECT DISTINCT company_id
                    FROM funding_round
                    WHERE is_first_round = 1
                        AND is_last_round = 1)))   
GROUP BY p.id, e.instituition;  


16. Посчитайте количество учебных заведений для каждого сотрудника из предыдущего задания. При подсчёте учитывайте, что некоторые сотрудники могли окончить одно и то же заведение дважды.

SELECT p.id,
        COUNT(e.instituition)
FROM people AS p 
    INNER JOIN education AS e ON p.id = e.person_id
WHERE p.id IN  (SELECT DISTINCT id
            FROM people
            WHERE company_id IN
                (SELECT DISTINCT c.id
                FROM company AS c
                WHERE status = 'closed'
                    AND id IN 
                    (SELECT DISTINCT company_id
                    FROM funding_round
                    WHERE is_first_round = 1
                        AND is_last_round = 1)))   
GROUP BY p.id;


17. Дополните предыдущий запрос и выведите среднее число учебных заведений (всех, не только уникальных), которые окончили сотрудники разных компаний. Нужно вывести только одну запись, группировка здесь не понадобится.

SELECT AVG(inst)
FROM
    (SELECT p.id,
            COUNT(e.instituition) AS inst
    FROM people AS p 
        INNER JOIN education AS e ON p.id = e.person_id
    WHERE p.id IN  (SELECT DISTINCT id
                FROM people
                WHERE company_id IN
                    (SELECT DISTINCT c.id
                    FROM company AS c
                    WHERE status = 'closed'
                        AND id IN 
                        (SELECT DISTINCT company_id
                        FROM funding_round
                        WHERE is_first_round = 1
                            AND is_last_round = 1)))   
    GROUP BY p.id) AS table_inst;

18. Напишите похожий запрос: выведите среднее число учебных заведений (всех, не только уникальных), которые окончили сотрудники Socialnet.

SELECT AVG(inst)
FROM
    (SELECT p.id,
            COUNT(e.instituition) AS inst
    FROM people AS p 
        INNER JOIN education AS e ON p.id = e.person_id
    WHERE p.id IN  (SELECT DISTINCT id
                FROM people
                WHERE company_id IN
                    (SELECT DISTINCT c.id
                    FROM company AS c
                    WHERE name = 'Socialnet'))   
    GROUP BY p.id) AS table_inst; 

19. Составьте таблицу из полей:

    name_of_fund — название фонда;
    name_of_company — название компании;
    amount — сумма инвестиций, которую привлекла компания в раунде.

В таблицу войдут данные о компаниях, в истории которых было больше шести важных этапов, а раунды финансирования проходили с 2012 по 2013 год включительно.


WITH
table_1 AS (SELECT c.name,
                    c.id
           FROM company AS c
           WHERE milestones > 6),
table_2 AS (SELECT fr.raised_amount AS amount,
                fr.company_id,
            fr.id
           FROM funding_round AS fr
           WHERE EXTRACT(YEAR FROM CAST(funded_at AS timestamp)) IN (2012, 2013)),
table_3 AS (SELECT fund_id,
                funding_round_id
           FROM investment),
table_4 AS (SELECT name,
                   id
           FROM fund)           
SELECT table_4.name AS name_of_fund,
        table_1.name AS name_of_company,
        table_2.amount
FROM table_1 
       INNER JOIN table_2 ON table_1.id = table_2.company_id
       INNER JOIN table_3 ON table_3.funding_round_id = table_2.id
       INNER JOIN table_4 ON table_4.id = table_3.fund_id;

20. Выгрузите таблицу, в которой будут такие поля:

    название компании-покупателя;
    сумма сделки;
    название компании, которую купили;
    сумма инвестиций, вложенных в купленную компанию;
    доля, которая отображает, во сколько раз сумма покупки превысила сумму вложенных в компанию инвестиций, округлённая до ближайшего целого числа.

Не учитывайте те сделки, в которых сумма покупки равна нулю. Если сумма инвестиций в компанию равна нулю, исключите такую компанию из таблицы. 
Отсортируйте таблицу по сумме сделки от большей к меньшей, а затем по названию купленной компании в лексикографическом порядке. Ограничьте таблицу первыми десятью записями.



WITH
table_1 AS (SELECT id,
                   name AS acquiring_name
           FROM company), --- поиск имени по id компании
table_2 AS (SELECT acquiring_company_id,
                   price_amount,
                   acquired_company_id
           FROM acquisition), ---данные покупок компаний
table_3 AS (SELECT id,
                   name AS acquired_name,
                    funding_total
           FROM company
           WHERE funding_total <> 0)    --- поиск имени по id компании
SELECT table_1.acquiring_name,
        table_2.price_amount,
        table_3.acquired_name,
        table_3.funding_total,
        ROUND(table_2.price_amount / table_3.funding_total)
FROM table_1 
    INNER JOIN table_2 ON table_1.id = table_2.acquiring_company_id
    INNER JOIN table_3 ON table_3.id = table_2.acquired_company_id
WHERE table_2.price_amount <> 0   
ORDER BY table_2.price_amount DESC, table_3.acquired_name ASC
LIMIT 10;

21. Выгрузите таблицу, в которую войдут названия компаний из категории social, получившие финансирование с 2010 по 2013 год включительно. Проверьте, что сумма инвестиций не равна нулю. Выведите также номер месяца, в котором проходил раунд финансирования.

WITH
table_1 AS (SELECT id AS company_id,
                   name
           FROM company
           WHERE category_code = 'social'
               AND funding_total <> 0),
table_2 AS (SELECT company_id,
                   EXTRACT(MONTH FROM CAST(funded_at AS timestamp)) AS month
            FROM funding_round
           WHERE EXTRACT(YEAR FROM CAST(funded_at AS timestamp)) BETWEEN 2010 AND 2013
               AND raised_amount <> 0)
SELECT table_1.name,
        table_2.month
FROM table_1
    INNER JOIN table_2 ON table_1.company_id = table_2.company_id;


22. Отберите данные по месяцам с 2010 по 2013 год, когда проходили инвестиционные раунды. Сгруппируйте данные по номеру месяца и получите таблицу, в которой будут поля:

    номер месяца, в котором проходили раунды;
    количество уникальных названий фондов из США, которые инвестировали в этом месяце;
    количество компаний, купленных за этот месяц;
    общая сумма сделок по покупкам в этом месяце.


WITH
table_1 AS (SELECT id AS fund_id,
                   name
           FROM fund
           WHERE country_code = 'USA'),
table_2 AS (SELECT fund_id,
                   funding_round_id
           FROM investment),
table_3 AS (SELECT EXTRACT(MONTH FROM CAST(funded_at AS timestamp)) AS month,
                   id AS funding_round_id
           FROM funding_round
           WHERE EXTRACT(YEAR FROM CAST(funded_at AS timestamp)) BETWEEN 2010 AND 2013),
table_4 AS (SELECT COUNT(acquired_company_id) AS num_companies,
                   EXTRACT(MONTH FROM CAST(acquired_at AS timestamp)) AS month,
                    SUM(price_amount) AS amount
            FROM acquisition
            WHERE EXTRACT(YEAR FROM CAST(acquired_at AS timestamp)) BETWEEN 2010 AND 2013
            GROUP BY month)       
SELECT table_5.month,
        table_5.names,
        table_4.num_companies,
        table_4.amount
FROM (SELECT table_3.month,
            COUNT(DISTINCT table_1.name) AS names 
            FROM table_1
                INNER JOIN table_2 ON table_1.fund_id = table_2.fund_id
                INNER JOIN table_3 ON table_3.funding_round_id = table_2.funding_round_id
                INNER JOIN table_4 ON table_3.month = table_4.month
            GROUP BY table_3.month) AS table_5
            INNER JOIN table_4 ON table_5.month = table_4.month;


23. Составьте сводную таблицу и выведите среднюю сумму инвестиций для стран, в которых есть стартапы, зарегистрированные в 2011, 2012 и 2013 годах. Данные за каждый год должны быть в отдельном поле. Отсортируйте таблицу по среднему значению инвестиций за 2011 год от большего к меньшему.


WITH
inv_2011 AS (SELECT country_code,
                    AVG(funding_total) AS total
            FROM company
            WHERE EXTRACT(YEAR FROM CAST(founded_at AS timestamp)) = 2011
            GROUP BY country_code),
inv_2012 AS (SELECT country_code,
                    AVG(funding_total) AS total
            FROM company
            WHERE EXTRACT(YEAR FROM CAST(founded_at AS timestamp)) = 2012
            GROUP BY country_code),
inv_2013 AS (SELECT country_code,
                    AVG(funding_total) AS total
            FROM company
            WHERE EXTRACT(YEAR FROM CAST(founded_at AS timestamp)) = 2013
            GROUP BY country_code)            
SELECT inv_2011.country_code,
        inv_2011.total,
      inv_2012.total,
      inv_2013.total
FROM inv_2011
INNER JOIN inv_2012 ON inv_2011.country_code = inv_2012.country_code
INNER JOIN inv_2013 ON inv_2011.country_code = inv_2013.country_code
ORDER BY inv_2011.total DESC;
