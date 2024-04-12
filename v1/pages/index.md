---
title: A First Experiment with Evidence
---

_I came across Evidence while looking through a list of YCombinator backed companies operating out of Canada. As someone who is a lot more comfortable with data (as opposed to web development), this was an incredibly fulfilling experience to be able to create data analyses and publish it, without writing a single line of JS_


## The Dataset

I didn't want to spend time hooking up a postgres DB, so I decided to go with a [CSV file from Guilherme Samora](https://raw.githubusercontent.com/justmarkham/DAT8/master/data/drinks.csv) containing data regarding alcohol consumption across countries and continents (which I've duly named `drinks.csv`).

I'm currently a graduate student at McGill University. As someone who went to college in the United States, I thought I was accustomed to large-scale college drinking. I should admit that drinking culture seems a lot more ingrained into the social culture here at McGill: I don't mean to be judgemental at all; in fact, I love beer (especially when you can get one for $2 on university grounds).

### Data Table

Taking a look at the data as-is.

<DataTable data={simple_preview} rows=6/>

I absolutely love this - to get the table above, I can just simply run SQL directly from a markdown document!

```sql simple_preview
SELECT * FROM evidence_tinkering.drinks;
```

## Trying a CASE Statement: Available Continents

I'm generating a list of continents that are represented in this dataset using a new SQL query as can be seen below. I'm doing this here specifically to see if my usual way of writing CASE statements will work without any issues.

<DataTable data={continents} rows=10/>


```sql continents
SELECT Continent AS "Continent Code",
  CASE WHEN Continent = 'AS' THEN 'Asia'
       WHEN Continent = 'EU' THEN 'Europe'
       WHEN Continent = 'AF' THEN 'Africa'
       WHEN Continent = 'NA' THEN 'North America'
       WHEN Continent = 'SA' THEN 'South America'
       WHEN Continent = 'OC' THEN 'Oceania'
       ELSE 'Unknown' END AS "Continent Name"
FROM evidence_tinkering.drinks
GROUP BY 1;
```

## Using an Aggregation: Finding the Top 10 Consumers of Beer

Taking a look at the top 10 consumers of beer as per the available data. Using a simple SQL GROUP BY and ORDER BY with a LIMIT 10 clause.

<DataTable data={top_10_consumers_of_beer} rows=10/>


```sql top_10_consumers_of_beer
SELECT country
   , SUM(beer_servings) AS "Beer Consumed"
FROM evidence_tinkering.drinks
GROUP BY 1
ORDER BY 2 DESC
LIMIT 10;
```

Pretty surprised to see Namibia leading this list (though I do have to admit that I have been trying to get my hands on a bottle of [Windhoek Beer](https://www.windhoekbeer.com) for over two years with no luck). Czechia isn't as much of a surprise considering they're renowned for Pilsners. Wondering what the trend is if the aggregation is performed by continent:

<DataTable data={top_consumers_of_beer_cont} rows=10/>


```sql top_consumers_of_beer_cont
SELECT continent
   , SUM(beer_servings) AS "Beer Consumed"
FROM evidence_tinkering.drinks
GROUP BY 1
ORDER BY 2 DESC;
```

## Average Beer Consumption by Continent

<DataTable data={avg_beer_cont} rows=7/>


```sql avg_beer_cont
SELECT continent
   , AVG(beer_servings) AS "Average Beer Consumption"
FROM evidence_tinkering.drinks
GROUP BY 1
ORDER BY 2 DESC;
```

## Ranking Top 5 Consumers of Beer by Continent

Attempting the use of Window Functions to get the top 5 consumers of beer by continent which can be chosen using a filter:

<Dropdown
  name=selected_continent
  data={conti}
  value=continent
/>

<DataTable data={top_5_beer_cont}>
  <Column id=country/>
  <Column id=continent/>
  <Column id=continent_rank/>
  <Column id=beer_servings contentType=colorscale/>
</DataTable>


```sql top_5_beer_cont

WITH rankings AS
(
  SELECT *
    , DENSE_RANK() OVER (PARTITION BY continent ORDER BY beer_servings DESC) AS continent_rank
  FROM evidence_tinkering.drinks
)
SELECT country,
       continent,
       continent_rank,
       beer_servings
FROM
rankings
WHERE continent = '${inputs.selected_continent.value}'
AND continent_rank <= 5
```

## Playing around with available visualization components

### Creating a Bar Plot of Beer Servings per year 

<BarChart 
  data={top_consumers_of_beer_cont} 
  x=continent
  y="Beer Consumed" 
  fillColor="#488f96"
  xAxisTitle = Continent
  yAxisTitle = "Beer Consumed"
>
</BarChart>

This is so neat! Especially considering tooltips that already come along with a beautiful plot!

### Creating a Big Value KPI Card

```sql total_booze_counts
SELECT SUM(beer_servings) AS total_beer_servings
   , SUM(wine_servings) AS total_wine_servings
   , SUM(spirit_servings) AS total_spirit_servings
FROM evidence_tinkering.drinks
```
<BigValue data={total_booze_counts} value=total_beer_servings fmt = '#,###'/>
<BigValue data={total_booze_counts} value=total_wine_servings fmt = '#,###'/>
<BigValue data={total_booze_counts} value=total_spirit_servings fmt = '#,###'/>


### Chart with Filter 


```sql countries
select
    country
from evidence_tinkering.drinks
group by country
```

```sql conti
SELECT continent
FROM evidence_tinkering.drinks
GROUP BY 1
```

<Dropdown data={countries} name=country value=country>
    <DropdownOption value="%" valueLabel="All Countries"/>
</Dropdown>

<Dropdown data = {conti} name=continent value = continent>
    <DropdownOption value="%" valueLabel="All Continents"/>
</Dropdown>

```sql wines
select 
    country,
    sum(wine_servings) as total_wine,
    continent
from evidence_tinkering.drinks
group by all
order by 3 desc
```

<BarChart
    data={wines}
    title="Total Wine Consumption"
    x=continent
    y=total_wine
    series=country
/>


