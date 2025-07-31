# MLB Player History Data Analysis Project with Advanced SQL

## About this project

This project takes a deep dive into the history of Major League Baseball (MLB) using a rich dataset of player information. As a data analyst for MLB, I used advanced SQL queries in MySQL to explore decades of player statistics, from the colleges that produce the most professional players to the spending habits of teams on player salaries.

The main questions I answered are:
- Which schools are the "MLB factories"?
- How much do teams really spend on their stars?
- What does a player's journey through the league look like?
- How do the physical attributes of players compare across different eras?

This analysis reveals fascinating trends over time and across different teams, offering a unique, data-driven look at America's favorite pastime.

**RESULTS:** The results confirm a clear trend - players have, on average, become taller and heavier over time. The query shows the specific increase in inches and pounds from one decade to the next, illustrating the changing physique of the modern athlete.

## Key Insights and Conclusions

This comprehensive analysis of MLB data reveals several important trends:

1. **Educational Pipeline**: College baseball has become increasingly important as a pathway to the major leagues, with the number of schools producing MLB players growing dramatically from the 1950s to the 2000s.

2. **Financial Landscape**: There's a clear divide between high-spending teams and others, with some franchises crossing the billion-dollar cumulative spending threshold, creating competitive advantages in talent acquisition.

3. **Career Patterns**: While most players have relatively short careers, some demonstrate remarkable longevity and loyalty, with certain players spending their entire careers with single franchises.

4. **Player Evolution**: The physical profile of MLB players has evolved significantly over time, with modern players being taller and heavier on average than their predecessors.

## Technologies Used

- **Database**: MySQL
- **Key SQL Concepts**: 
  - Common Table Expressions (CTEs)
  - Window Functions (ROW_NUMBER, NTILE, LAG, SUM OVER)
  - Complex JOINs
  - Date/Time functions (TIMESTAMPDIFF)
  - Aggregate functions and GROUP BY
  - Subqueries and nested queries

## Database Schema

The analysis utilized several key tables:
- `players`: Player biographical information and career dates
- `schools`: Player educational background
- `school_details`: Detailed school information
- `salaries`: Player salary information by year and team

This project demonstrates advanced SQL techniques applied to real-world sports data, providing actionable insights into the business and competitive landscape of Major League Baseball.
