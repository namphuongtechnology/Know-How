# Database Schema in 1NF

All columns already have atomic values

# Database Schema in 2NF

Columns like Team, Games and NOC do not fully depend on the ID of the athlete. They depend on different entities (like
team, game and NOC details).

# Database Schema in 3NF

With 270k data records, I thnk each table should only contain data directly related to the primary key.
There are no transitive dependencies (non-key attributes depending on other non-key attributes).

## Athletes Table

| Column Name    | Data Type | Description                                           |
|----------------|-----------|-------------------------------------------------------|
| ID (PK)        | INT       | Unique identifier for each athlete.                   |
| Name           | VARCHAR   | Name of the athlete.                                  |
| Sex            | CHAR(1)   | Gender of the athlete ('M' for male, 'F' for female). |
| Age            | INT       | Age of the athlete.                                   |
| Height         | INT       | Height of the athlete in centimeters.                 |
| Weight         | INT       | Weight of the athlete in kilograms.                   |
| CountryID (FK) | INT       | Foreign key referencing the `Countries` table.        |

## Countries Table (Teams)

| Column Name | Data Type | Description                                                                          |
|-------------|-----------|--------------------------------------------------------------------------------------|
| ID (PK)     | INT       | Unique identifier for each country.                                                  |
| Country     | VARCHAR   | Name of the country.                                                                 |
| NOC (FK)    | CHAR(3)   | National Olympic Committee code; references the `National_Olympic_Committees` table. |

## National_Olympic_Committees Table (NOC)

| Column Name | Data Type | Description                                    |
|-------------|-----------|------------------------------------------------|
| NOC (PK)    | CHAR(3)   | National Olympic Committee code (primary key). |
| Region      | VARCHAR   | Region or country represented by the NOC.      |
| Notes       | VARCHAR   | Additional notes, if any.                      |

## Games Table

| Column Name | Data Type | Description                                                   |
|-------------|-----------|---------------------------------------------------------------|
| ID (PK)     | INT       | Unique identifier for each Olympic Games event.               |
| Name        | VARCHAR   | The official name of the Olympic Games (e.g., '1992 Summer'). |
| Year        | INT       | Year the Olympic Games took place.                            |
| Season      | VARCHAR   | Season of the games ('Summer' or 'Winter').                   |
| City        | VARCHAR   | Host city of the Olympic Games.                               |

## Sports Table

| Column Name | Data Type | Description                       |
|-------------|-----------|-----------------------------------|
| ID (PK)     | INT       | Unique identifier for each sport. |
| Name        | VARCHAR   | Sport name.                       |

## Events Table

| Column Name  | Data Type | Description                                     |
|--------------|-----------|-------------------------------------------------|
| ID (PK)      | INT       | Unique identifier for each event.               |
| SportID (FK) | INT       | Foreign key referencing the `Sports` table.     |
| Name         | VARCHAR   | Specific event name (e.g., 'Men's Basketball'). |

## Athlete_Events Table

| Column Name         | Data Type | Description                                      |
|---------------------|-----------|--------------------------------------------------|
| AthleteEventID (PK) | INT       | Unique identifier for each athlete event record. |
| AthleteID (FK)      | INT       | Foreign key referencing the `Athletes` table.    |
| GameID (FK)         | INT       | Foreign key referencing the `Games` table.       |
| EventID (FK)        | INT       | Foreign key referencing the `Events` table.      |
| MedalID (FK)        | INT       | Foreign key referencing the `Medals` table.      |

## Medals table

| Column Name | Data Type | Description                                      |
|-------------|-----------|--------------------------------------------------|
| ID (PK)     | INT       | Unique identifier for each athlete event record. |
| Medal (PK)  | VARCHAR   | Medal won ('Gold', 'Silver', 'Bronze', or NULL). |

## Table Examples

`Athlete Table`

 id | name      | sex | age | height | weight | country_id 
----|-----------|-----|-----|--------|--------|------------
 1  | A Dijiang | M   | 24  | 180    | 80     | 1          

`Country Table`

 id | country | noc 
----|---------|-----
 1  | Algeria | ALG 

`NOC`

 id | noc | region  | note 
----|-----|---------|------
 4  | ALG | Algeria | null 

`Game Table`

 id | name        | year | sesaon | city   
----|-------------|------|--------|--------
 1  | 1994 Winter | 1994 | Winter | London 

`Sport Table`

 id | sport_name 
----|------------
 4  | Archery    

`Event Table`

 id | sport_id | name                             
----|----------|----------------------------------
 5  | 54       | Speed Skating Women's 500 metres 

`Medal:`

 id | medal_name 
----|------------
 1	 | Gold       
 2  | Silver     
 3  | Bronze     

## Tasks

1. Ordering the nations in descending order based on the number of medals won.

```
SELECT 
    c.Country, 
    SUM(CASE 
            WHEN m.Medal = 'Gold' THEN 3 
            WHEN m.Medal = 'Silver' THEN 2 
            WHEN m.Medal = 'Bronze' THEN 1 
            ELSE 0 
        END) AS TotalPoints
FROM 
    Athlete_Events ae
JOIN 
    Athletes a ON ae.AthleteID = a.ID
JOIN 
    Countries c ON a.CountryID = c.ID
JOIN 
    Medals m ON ae.MedalID = m.ID
GROUP BY 
    c.Country
ORDER BY 
    TotalPoints DESC;
```

Create INDEX for this query to tune performance

* athelete_id of Athlete_Events Table is a good candidate for creating INDEX because
  the number of athelete_id is high (high cardinality) and athelete_id is often queried

```
-- Index on Athlete_Events.AthleteID (to speed up join with Athletes table)
CREATE INDEX idx_athlete_events_athlete_id ON Athlete_Events (AthleteID);

```

* country_id in Athletes TABLE can be a good candidate for creating INDEX
because there can be significant number of countres and country_id is often queried

```
-- Index on Athletes.CountryID (to speed up join with Countries table)
CREATE INDEX idx_athletes_countryid ON Athletes (CountryID);
```
* medal_id in Athlete_Events.MedalID and Medals TABLES is a poor choice for creating INDEX because
there can only be 4 possible values GOLD, SILVER, BRONZE, NULL (low cardinality)

``` 
-- Index on Athlete_Events.MedalID (to speed up join with Medals table)
CREATE INDEX idx_athlete_events_medalid ON Athlete_Events (MedalID);

-- Index on Medals.Medal (to speed up the CASE evaluation in the SUM function)
CREATE INDEX idx_medals_medal ON Medals (Medal);
```

2. Who won medals at the 2004 Olympics?

```
SELECT 
    a.Name, 
    m.Medal, 
    g.Name AS GameName, 
    e.Name AS EventName
FROM 
    Athlete_Events ae
JOIN 
    Athletes a ON ae.AthleteID = a.ID
JOIN 
    Games g ON ae.GameID = g.ID
JOIN 
    Medals m ON ae.MedalID = m.ID
WHERE 
    g.Year = 2004 AND m.Medal IS NOT NULL;

```

3. How many Olympics games have been held?

```
SELECT 
    COUNT(DISTINCT Year) AS TotalOlympicGames
FROM 
    Games;

```

4. Identify the sport that was played in all summer Olympics.

``` 
SELECT 
    s.Name AS SportName
FROM 
    Sports s
JOIN 
    Events e ON s.ID = e.SportID
JOIN 
    Athlete_Events ae ON e.ID = ae.EventID
JOIN 
    Games g ON ae.GameID = g.ID
WHERE 
    g.Season = 'Summer'
GROUP BY 
    s.Name
HAVING 
    COUNT(DISTINCT g.Year) = (SELECT COUNT(DISTINCT Year) FROM Games WHERE Season = 'Summer');

```