# Project on COVID 19 data 

#### Skills used: Joins, CTE's, Temp Tables, Windows Functions, Aggregate Functions, Creating Views, Converting Data Types


### Looking at the death dataset and some of its columns.

```sql
SELECT location, date, total_cases, new_cases, total_deaths, population
FROM ProjectPortfolio..CovidDeaths
ORDER BY 1, 2;

Select Location, date, total_cases, new_cases, total_deaths, population
From ProjectPortfolio..CovidDeaths
Where continent is not null 
order by 1,2;
```

### What percentage of people died who got covid in United States ?

```sql
SELECT location, date, total_cases, total_deaths, ROUND((total_deaths/total_cases)*100,2) AS death_percentage
FROM ProjectPortfolio..CovidDeaths
WHERE location = 'United States' AND continent is not null 
ORDER BY 1, 2;
```

### What percentage of population got covid in United States ?

```sql
SELECT location, date, total_cases, ROUND((total_cases/population)*100,2) AS case_percentage
FROM ProjectPortfolio..CovidDeaths
WHERE location = 'United States' AND continent is not null 
ORDER BY 1, 2;
```


### Countries with Highest Infection Rate compared to Population

```sql
SELECT location, population, MAX(total_cases) AS most_recent_count, ROUND(MAX((total_cases/population)*100),2) AS percentage_poluation_infected
FROM ProjectPortfolio..CovidDeaths
GROUP BY location, population
ORDER BY ROUND(MAX((total_cases/population)*100),2) DESC;
```

### Top 5 Countries with Highest Death Count per Population

```sql
SELECT location, population, MAX(CAST(total_deaths AS INT)) AS total_death_count
FROM ProjectPortfolio..CovidDeaths
WHERE location NOT IN ('World', 'Europe', 'South America', 'Asia', 'North America', 'European Union')
GROUP BY location, population
ORDER BY MAX(CAST(total_deaths AS INT)) DESC
OFFSET 0 ROWS FETCH FIRST 5 ROWS ONLY;
```

### Showing contintents with the highest death count per population

```sql
SELECT location, population, MAX(CAST(total_deaths AS INT)) AS total_death_count
FROM ProjectPortfolio..CovidDeaths
WHERE location IN ('Europe', 'South America', 'Asia', 'North America', 'European Union')
GROUP BY location, population
ORDER BY MAX(CAST(total_deaths AS INT)) DESC
```

### JOINING two datasets using date and location column. 

```sql 
SELECT * 
FROM ProjectPortfolio..CovidDeaths deaths
JOIN ProjectPortfolio..CovidVaccinations vaccines 
	ON deaths.location = vaccines.location AND deaths.date = vaccines.date
```

### Total Population VS Vaccinations 

```sql
SELECT death.continent, death.location, death.date, death.population, vaccine.new_vaccinations,
		SUM(CONVERT(INT,vaccine.new_vaccinations)) OVER (PARTITION BY death.location ORDER BY death.location, death.date) AS rolling_total_of_vaccination
FROM ProjectPortfolio..CovidDeaths death
JOIN ProjectPortfolio..CovidVaccinations vaccine 
	ON death.location = vaccine.location AND death.date = vaccine.date
WHERE death.continent IS NOT NULL
ORDER BY 2, 3
```

### Calculation Rate of people vaccinated versus total population using CTE

```sql
WITH population_vaccine AS (
SELECT death.continent, death.location, death.date, death.population, vaccine.new_vaccinations,
		SUM(CONVERT(INT,vaccine.new_vaccinations)) OVER (PARTITION BY death.location ORDER BY death.location, death.date) AS rolling_total_of_vaccination
FROM ProjectPortfolio..CovidDeaths death
JOIN ProjectPortfolio..CovidVaccinations vaccine 
	ON death.location = vaccine.location AND death.date = vaccine.date
WHERE death.continent IS NOT NULL
)
SELECT continent, location, date, population, ROUND((rolling_total_of_vaccination/population)*100,2) AS rate_of_vacciantion
FROM population_vaccine
ORDER BY 2,3;
```



