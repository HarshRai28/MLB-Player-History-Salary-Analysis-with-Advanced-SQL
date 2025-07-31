# 1. Which Schools Are the "MLB Factories"?

To find out which schools have produced the most MLB players, I joined the schools & school_details tables and counted the number of players from each school. From there, I wanted to see how the role of college baseball has evolved. I first calculated the number of schools that produced MLB players for each decade.

```sql
SELECT   FLOOR(yearID / 10) * 10 AS decade, 
         COUNT(DISTINCT schoolID) AS num_school
FROM     schools
GROUP BY decade 
ORDER BY decade;
```

**RESULTS:** The results show a clear and steady increase over time. For example, while in the 1950s there might have been around 150 schools producing players, by the 2000s, that number had grown close to 500. This tells us that college baseball has become an increasingly vital pathway to the major leagues.

Next, I wanted to identify the all-time powerhouses. Which schools have produced the most players in MLB history?

```sql
SELECT   sd.name_full, COUNT(DISTINCT s.playerID) AS num_players
FROM     schools s LEFT JOIN school_details sd
                 ON s.schoolID = sd.schoolID
GROUP BY sd.name_full
ORDER BY num_players DESC
LIMIT    5;
```

**RESULTS:** This query gives us the "Top 5" list of MLB factories. These universities are legendary for their baseball programs.

Finally, to get a more detailed historical view, I used a more advanced query with a Common Table Expression (CTE) and a window function. This allowed me to find the top 3 schools that produced the most players for each decade, showing how the landscape has shifted over time.

```sql
WITH ds AS (SELECT   FLOOR(s.yearID / 10) * 10 AS decade, sd.name_full,                                         COUNT(DISTINCT s.playerID) AS num_players  
            FROM     schools s LEFT JOIN school_details sd
                     ON s.schoolID = sd.schoolID
            GROUP BY decade, sd.name_full),
            
rn AS (SELECT  decade, name_full, num_players,
                ROW_NUMBER() OVER(PARTITION BY decade ORDER BY num_players desc) AS row_num
        FROM ds)
        
SELECT decade, name_full, num_players
FROM rn
WHERE row_num <= 3
ORDER BY decade DESC, row_num;
```

**RESULTS:** This powerful query reveals which schools were dominant in different eras.

# 2. How Much Do Teams Spend on Player Salaries?

The world of Major League Baseball is not just about home runs and strikeouts; it's also a big business where money plays a critical role. To understand the financial landscape of the league, I analyzed team salaries to see how spending habits differ and how they've changed over time.

My first goal was to identify the league's "heavy hitters" in terms of spending. I used a window function NTILE, to segment all teams into five groups based on their average annual spending. This allowed me to isolate the top 20% of teams that consistently spend the most.

```sql
WITH ts AS (SELECT teamID, yearID, SUM(salary) AS total_spend
                FROM salaries
                GROUP BY teamID, yearID
                ORDER BY teamID, yearID),
    
        sp AS (SELECT teamID, AVG(total_spend) AS avg_spend,
                  NTILE(5) OVER(ORDER BY AVG(total_spend) DESC) AS spend_pct
            FROM  ts                
            GROUP BY teamID)
    
    SELECT teamID, ROUND(avg_spend / 1000000, 1) AS avg_spend_million 
FROM sp
WHERE spend_pct = 1;
```

**RESULTS:** The results clearly identify the financial powerhouses of the league. These teams consistently have the highest average payrolls, giving them a significant advantage in acquiring top talent.

Next, I wanted to track how each team's financial commitment has grown over the years. I calculated a rolling cumulative sum of spending for each team. This shows a team's total investment in players from their early years up to the present day.

```sql
WITH ts AS (SELECT teamID, yearID, SUM(salary) as total_spend
                        FROM salaries 
                        GROUP BY teamID, yearID
                ORDER BY teamID, yearID)
            
SELECT  teamID, yearID,
                ROUND(SUM(total_spend) OVER(PARTITION BY teamID ORDER BY yearID) / 1000000,1) 
        AS cumulative_sum_millions
FROM ts;
```

**RESULTS:** This query provides a fascinating look at a team's financial history. For a team like the New York Yankees, you can see their spending explode over time.

Finally, I performed a more advanced analysis to pinpoint a major financial milestone: the exact year each team's cumulative spending crossed the $1 billion threshold. Using multiple Common Table Expressions (CTEs), I could isolate this specific moment in time for each franchise.

```sql
WITH ts AS (SELECT teamID, yearID, SUM(salary) as total_spend
            FROM salaries 
            GROUP BY teamID, yearID
            ORDER BY teamID, yearID),
    
        cs AS (SELECT teamID, yearID,
                    SUM(total_spend) OVER(PARTITION BY teamID ORDER BY yearID) AS cumulative_sum
            FROM ts),

    rn AS (SELECT teamID, yearID, cumulative_sum,
                    ROW_NUMBER() OVER(PARTITION BY teamID ORDER BY cumulative_sum) AS rn
            FROM cs
            WHERE cumulative_sum > 1000000000)
            
SELECT teamID, yearID, ROUND(cumulative_sum / 1000000000, 2) AS cumulative_sum_billions
FROM rn
WHERE rn = 1;
```

**RESULTS:** This query reveals the "Billion-Dollar Club" and when each team joined it.

# 3. What Does Each Player's Career Look Like?

Behind every player's stats is a unique career path, with its own beginning, end, and journey in between. To understand these individual stories, I analyzed player careers to measure their longevity, trace their team history, and identify those who showed exceptional loyalty to a single franchise.

First, I wanted to measure the length of a player's time in the major leagues. Using MySQL's TIMESTAMPDIFF function, I calculated each player's age at their debut, their age in their final game, and the total length of their career.

```sql
SELECT  nameGiven,
        CAST(CONCAT(birthYear, '-', birthMonth, '-', birthDay) AS DATE) AS birthDate,
        TIMESTAMPDIFF(YEAR, CAST(CONCAT(birthYear, '-', birthMonth, '-', birthDay) AS DATE), debut) AS starting_age,
        TIMESTAMPDIFF(YEAR, CAST(CONCAT(birthYear, '-', birthMonth, '-', birthDay) AS DATE), finalGame) AS end_age,
        TIMESTAMPDIFF(YEAR, debut, finalGame) AS career_length
FROM players
ORDER BY career_length DESC;
```

**RESULTS:** This query highlights the incredible durability of some athletes. The results show the players with the longest careers, some lasting for well over three decades.

Next, I wanted to see the bookends of each player's career: which team did they start with, and which team did they end with? By joining the players table with the salaries table twice, I could pinpoint their starting and ending teams.

```sql
SELECT  p.nameGiven,
        s.yearID AS starting_year, s.teamID AS starting_team, 
        e.yearID AS ending_year, e.teamID AS ending_team
FROM    players p INNER JOIN salaries s
                            ON p.playerID = s.playerID
                            AND YEAR(p.debut) = s.yearID
                   INNER JOIN salaries e
                            ON p.playerID = e.playerID
                            AND YEAR(p.finalGame) = e.yearID;
```

**RESULTS:** This query traces the full arc of a player's journey.

Finally, I wanted to identify a special group of players: the long-serving legends who stayed with a single team for more than a decade. By filtering the previous query, I could find players who started and ended their long careers with the same franchise.

```sql
SELECT  p.nameGiven,
        s.yearID AS starting_year, s.teamID AS starting_team, 
        e.yearID AS ending_year, e.teamID AS ending_team
FROM    players p INNER JOIN salaries s 
                             ON p.playerID = s.playerID
                             AND YEAR(p.debut) = s.yearID
                  INNER JOIN salaries e 
                             ON p.playerID = e.playerID
                             AND YEAR(p.finalGame) = e.yearID
WHERE   s.teamID = e.teamID
        AND e.yearID - s.yearID > 10;
```

**RESULTS:** This reveals the "one-club" players who became icons for their teams, demonstrating remarkable loyalty and consistency over a long period.

# 4. How Do Player Attributes Compare?

Beyond their performance on the field, what makes MLB players unique are their physical attributes and personal details. In this final part of the analysis, I explored some interesting and fun facts about the players themselves, from shared birthdays to physical changes in athletes over the decades.

First, a fun query to answer a common trivia question: which players share a birthday? I wrote a query to group players by their birth date.

```sql
WITH bd AS (SELECT  CAST(CONCAT(birthYear, '-', birthMonth, '-', birthDay) AS DATE) AS birthdate,
                    nameGiven
            FROM    players)
            
SELECT   birthdate, GROUP_CONCAT(nameGiven SEPARATOR ', ') AS players
FROM     bd
WHERE    YEAR(birthdate) BETWEEN 1980 AND 1990
GROUP BY birthdate
ORDER BY birthdate;
```

**RESULTS:** This query uncovers some fun connections between players who might have celebrated their birthdays on the same day.

Next, I analyzed the distribution of batting preferences across different teams. Are some teams more reliant on left-handed hitters? I calculated the percentage of right-handed, left-handed, and switch-hitters for each team.

```sql
SELECT   s.teamID, 
         ROUND(SUM(CASE WHEN p.bats = 'R' THEN 1 ELSE 0 END) / COUNT(s.playerID)*100, 1) AS bats_right,
         ROUND(SUM(CASE WHEN p.bats = 'L' THEN 1 ELSE 0 END) / COUNT(s.playerID)*100, 1) AS bats_left,
         ROUND(SUM(CASE WHEN p.bats = 'B' THEN 1 ELSE 0 END) / COUNT(s.playerID)*100, 1) AS bats_both
FROM     salaries s LEFT JOIN players p
         ON s.playerID = p.playerID 
GROUP BY s.teamID;
```

**RESULTS:** The output shows the strategic makeup of each team's lineup, with most teams heavily favoring right-handed batters, but with varying percentages of left-handed and switch-hitters.

Finally, I investigated how the physical profile of an MLB player has evolved. Are players getting taller and heavier? Using the LAG window function, I calculated the decade-over-decade change in average height and weight at the time of a player's debut.

```sql
WITH avg_hw AS (SELECT   FLOOR(YEAR(debut) / 10)*10 AS decade,
                     AVG(height) AS avg_height, 
                     AVG(weight) AS avg_weight
            FROM     players
            GROUP BY decade
            ORDER BY decade)
            
SELECT  decade, 
        avg_height - LAG(avg_height) OVER(ORDER BY decade) as height_diff,
        avg_weight - LAG(avg_weight) OVER(ORDER BY decade) as weight_diff
FROM    avg_hw
WHERE   decade IS NOT NULL;
```

**RESULTS:** The results confirm a clear trend - players have, on average, become taller and heavier over time. The query shows the specific increase in inches and pounds from one decade to the next, illustrating the changing physique of the modern athlete.
