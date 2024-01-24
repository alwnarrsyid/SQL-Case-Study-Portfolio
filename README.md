SQL-Case-Study-Portofolio
# All-time Olympic Games Analytics
### Introduction
Embark on a data-driven journey through Olympic Games history! From 1986 to 2016, I've crunched numbers on medal distributions, athlete performances, and historical trends. Get ready for in-depth statistical insights, discovering patterns, breakthroughs, and iconic moments that shaped the Olympics during this exciting era. Join me as we dive into the rich tapestry of Olympic history, exploring sportsmanship evolution and achievements through dynamic data analysis.

  Data source :    
  [Kaggle Dataset : 120 Year of Olympic History athletes and results](https://www.kaggle.com/datasets/heesoo37/120-years-of-olympic-history-athletes-and-results/)
  
---

#### **Q1 - How many teams participate per 'Games'?**
```SQl
SELECT
    ae.games, COUNT(DISTINCT nr.region) AS total_country
FROM
    athlete_events ae
JOIN
    noc_regions nr ON nr.NOC = ae.noc
GROUP BY 1 
ORDER BY 1;
```
Result :
![total country per game](https://github.com/alwnarrsyid/SQL-Case-Study-Portfolio/assets/155807976/b3f7e059-8a5a-42e6-9275-970b8e16e6f7)

---

#### **Q2 - Which year saw the highest and lowest no of countries participating in olympics?**
```sql
WITH highest AS (
    SELECT  
        ae.games, COUNT(DISTINCT nr.region) AS total_country
    FROM
        athlete_events ae
    JOIN noc_regions nr ON nr.NOC = ae.noc
    GROUP BY 1
    ORDER BY 2 DESC
    LIMIT 1
), lowest AS (
    SELECT
        ae.games, COUNT(DISTINCT nr.region) AS total_country
    FROM
        athlete_events ae
    JOIN noc_regions nr ON nr.NOC = ae.noc
    GROUP BY 1
    ORDER BY 2 ASC
    LIMIT 1
)
SELECT
    CONCAT(h.games, ' - ', h.total_country) AS highest,
    CONCAT(l.games, ' - ', l.total_country) AS lowest
FROM
    highest h
    CROSS JOIN lowest l;
```
Result :
![High&Low](https://github.com/alwnarrsyid/SQL-Case-Study-Portfolio/assets/155807976/626398f2-581f-4ee0-92a2-6b2f986e215c)

---

#### **Q3 - Which country has participated in all of the olympic games?**
```sql
SELECT
    region, COUNT(DISTINCT games) AS total_games
FROM
    athlete_events ae
JOIN olympic.noc_regions nr ON ae.noc = nr.NOC
GROUP BY 1
ORDER BY 2 DESC
LIMIT 2;
```
Result :
![all](https://github.com/alwnarrsyid/SQL-Case-Study-Portfolio/assets/155807976/0bcd7b0e-3e95-4ac2-bd00-3ada9f609701)

---

#### **Q4 - Identify the sports which were played in all summer olympics.**
```sql
SELECT
    COUNT(DISTINCT games) AS total_games
FROM
    athlete_events
WHERE season = 'Summer';
```
Result :
![image](https://github.com/alwnarrsyid/SQL-Case-Study-Portfolio/assets/155807976/2f0f3df2-2303-46b2-8ee4-f85eaf9de498)

**AND THEN**
```sql
SELECT
    DISTINCT sport, COUNT(DISTINCT games) AS total
FROM
    athlete_events
WHERE games LIKE '%summer%'
GROUP BY 1
HAVING total = 29
ORDER BY 2 DESC;
```
Result : 
![image](https://github.com/alwnarrsyid/SQL-Case-Study-Portfolio/assets/155807976/247bbe40-0be5-45ab-a10c-cd5159282f11)

---
#### **Q5 - Which Sports were just played only once in the entire olympics?**
```sql
WITH total_olympic AS (
    SELECT
        sport, COUNT(DISTINCT games) AS total_games
    FROM
        athlete_events
    GROUP BY 1
    HAVING total_games = 1
    ORDER BY 2 ASC
)
SELECT 
    DISTINCT ae.sport, ae.games, t.total_games
FROM
    athlete_events ae
JOIN
    total_olympic t ON ae.sport = t.sport
ORDER BY 2 ASC;
```
Result :
![image](https://github.com/alwnarrsyid/SQL-Case-Study-Portfolio/assets/155807976/ee923343-ff0e-4e0b-9560-5abe30e554a8)

---

#### **Q6 - Fetch the total no of sports played in each olympic games.**
```sql
SELECT 
    DISTINCT games, COUNT(DISTINCT sport) AS total_sport
FROM
    athlete_events
GROUP BY 1
ORDER BY 1;
```
Result :
![image](https://github.com/alwnarrsyid/SQL-Case-Study-Portfolio/assets/155807976/4a16d033-a7bb-4b61-9255-80979f36991c)

---

#### **Q7 - Fetch oldest athletes to win a gold medal.**
```sql
SELECT
    DISTINCT name, sex, age, team, games,
    city, sport, event, Medal
FROM
    athlete_events
WHERE medal = 'Gold'
ORDER BY age DESC
LIMIT 2;
``` 
Result : 
|         Athlete         | Gender | Age |    Country    |   Year   |    City    |   Sport   |               Event               | Medal |
|------------------------|--------|-----|---------------|----------|------------|-----------|----------------------------------|-------|
| Charles Jacobus        |   M    |  64 | United States | 1904 Summer |  St. Louis |   Roque   |      Roque Men's Singles        | Gold  |
| Oscar Gomer Swahn      |   M    |  64 |     Sweden    | 1912 Summer | Stockholm  |  Shooting | Shooting Men's Running Target, Single Shot, Team | Gold  |
##### **OR :**
![image](https://github.com/alwnarrsyid/SQL-Case-Study-Portfolio/assets/155807976/7acc1e0c-aec7-4faf-a5be-a053d4f0fecf)

---

#### **Q8 - Find All-time total Athletes participated in the olympic games?**
```sql
SELECT
    COUNT(DISTINCT name) AS total_participants
FROM
    athlete_events;
```
Result :
![image](https://github.com/alwnarrsyid/SQL-Case-Study-Portfolio/assets/155807976/729d842d-0416-4e95-b019-bebd02d19395)

---

#### **Q9 - Find the Ratio of male and female athletes participated in all olympic games?**
```sql
WITH total_m AS (
    SELECT
        sex, COUNT(sex) AS total_male
    FROM
        athlete_events
    WHERE sex = 'M'
    GROUP BY 1
),
total_f AS (
    SELECT
        sex, COUNT(sex) AS total_female
    FROM
        athlete_events
    WHERE sex = 'F'
    GROUP BY 1
),
total_gender AS (
    SELECT
        COUNT(sex) AS total_sex
    FROM
        athlete_events
)
SELECT DISTINCT
    ae.sex,
    COUNT(ae.sex) OVER (PARTITION BY ae.sex) AS total,
    CASE
        WHEN tm.sex = ae.sex THEN ROUND((total_male / total_sex) * 100, 2)
        WHEN tf.sex = ae.sex THEN ROUND((total_female / total_sex) * 100, 2)
    END AS sex_percentage,
    ROUND(total_male / total_female, 2) AS ratio
FROM
    athlete_events ae
CROSS JOIN total_f tf
CROSS JOIN total_m tm
CROSS JOIN total_gender;
```
Result :
![image](https://github.com/alwnarrsyid/SQL-Case-Study-Portfolio/assets/155807976/3fea03fe-7d02-494d-b67f-2e156f1b347b)

---
#### **Q10 - Find the top 5 athletes who have won the most gold medals? (also consider those athletes who have same number of Gold Medals)**
```sql
WITH cte_rank AS (
    SELECT 
        name, COUNT(medal) AS gold_medal
    FROM
        athlete_events
    WHERE medal = 'Gold'
    GROUP BY  1
    ORDER BY 2 DESC
),
cte_ranking AS (
    SELECT 
        *, DENSE_RANK() OVER (ORDER BY gold_medal DESC) AS ranking
    FROM
        cte_rank
)
SELECT * FROM cte_ranking
WHERE ranking <= 5;
```
Result :
![image](https://github.com/alwnarrsyid/SQL-Case-Study-Portfolio/assets/155807976/892740c3-7da7-4739-8b99-38ca0e9da70e)

---

#### **Q11 - Fetch the top 5 athletes who have won the most medals (gold/silver/bronze)**
```sql
WITH cte_medal AS (
    SELECT 
        DISTINCT name, medal,
        COUNT(medal) AS medal_count
    FROM
        athlete_events
    WHERE
        medal IS NOT NULL
    GROUP BY 1, 2
    ORDER BY 3 DESC
),
cte_ranking AS (
    SELECT
        name,
        GROUP_CONCAT(medal_count, ' ', medal SEPARATOR ', ') AS all_time_medal,
        SUM(medal_count) AS total_medal
    FROM
        cte_medal
    GROUP BY 1
    ORDER BY total_medal DESC
),
cte_all AS (
    SELECT
        *,
        DENSE_RANK() OVER (ORDER BY total_medal DESC) AS ranking
    FROM
        cte_ranking
)
SELECT * FROM
    cte_all
WHERE
    ranking <= 5;
```
Result : 
![image](https://github.com/alwnarrsyid/SQL-Case-Study-Portfolio/assets/155807976/f1886952-609c-48c3-b4e5-752caad3fe7d)

---

#### **Q12 - Fetch the top 5 most successful countries in olympics. (Success is defined by Highest no of medals won)**
```sql
WITH cte_total_medal AS (
    SELECT 
        DISTINCT region, medal,
        COUNT(medal) AS total_medal
    FROM
        athlete_events ae
    JOIN
        olympic.noc_regions nr ON ae.noc = nr.NOC
    WHERE medal IS NOT NULL
    GROUP BY 1, 2
    ORDER BY 3 DESC
),
cte_all AS (
    SELECT
        region,
        GROUP_CONCAT(total_medal, ' ', Medal SEPARATOR ', ') AS medal,
        SUM(total_medal) AS total_medal
    FROM
        cte_total_medal
    GROUP BY 1
    ORDER BY 3 DESC
),
cte_ranking AS (
    SELECT *,
        DENSE_RANK() OVER (ORDER BY total_medal DESC) AS ranking
    FROM
        cte_all
)
SELECT * FROM
    cte_ranking
WHERE ranking <= 5;
```
Result :
![image](https://github.com/alwnarrsyid/SQL-Case-Study-Portfolio/assets/155807976/120cdb30-f679-4b02-b446-01443eee5167)

---

#### **Q13 - In which Sport/event, Indonesia has won highest medals.**
```sql
WITH cte_medal AS (
    SELECT
        region, ae.sport, medal,
        COUNT(ae.medal) AS total_medal
    FROM
        athlete_events ae
    JOIN
        olympic.noc_regions nr ON ae.noc = nr.NOC
    WHERE
        region = 'Indonesia' AND medal IS NOT NULL
    GROUP BY 1, 2, 3
    ORDER BY 4 DESC
)
SELECT
    region, Sport,
    GROUP_CONCAT(total_medal, ' ', medal SEPARATOR ', ') AS medal,
    SUM(total_medal) AS total_medal,
    DENSE_RANK() OVER (ORDER BY SUM(total_medal) DESC) AS ranking
FROM
    cte_medal
GROUP BY 1, 2
ORDER BY 4 DESC
LIMIT 3;
```
Result :
![image](https://github.com/alwnarrsyid/SQL-Case-Study-Portfolio/assets/155807976/7b8fea7c-f169-4819-8aaa-9a492e77fc6a)

---

#### **Q14 - Break down all olympic games where India won medal for Hockey and how many medals in each olympic games.**
```sql
SELECT DISTINCT
    n.region, sport, Games,
    medal, event,
    COUNT(medal) OVER (PARTITION BY Games) AS total_medal
FROM
    athlete_events ae
JOIN
    (SELECT * FROM noc_regions WHERE region = 'Indonesia') n ON ae.noc = n.noc
WHERE medal IS NOT NULL;
```
Result :
![image](https://github.com/alwnarrsyid/SQL-Case-Study-Portfolio/assets/155807976/2e03993a-5cac-445d-9a78-af350ffac1be)

### My social media :
[![Gmail Logo](https://img.shields.io/badge/Gmail-D14836?style=for-the-badge&logo=gmail&logoColor=white)](mailto:alwanforjobs@gmail.com) [![Linkedin](https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/alwnarrsyid/) [![Instagram](https://img.shields.io/badge/Instagram-red?style=for-the-badge&logo=instagram&logoColor=white)](https://www.instagram.com/alwnarrsyid/)