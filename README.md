# 📊 COVID-19 Data Exploration with SQL
## 📌 Project Overview

This project explores global COVID-19 data using SQL.

The dataset includes information about cases, deaths, and vaccinations.

The goal is to analyze key metrics such as:

- Infection rates
- Death percentages
- Vaccination progress
- Global trends

## 🔹Key Insights

- Identified countries with the highest infection rate vs. population
- Analyzed death counts per country & continent
- Tracked global case and death trends

# 📂 Dataset
The data comes from Our World in Data – COVID-19 dataset.

Two tables are used:

- CovidDeaths → contains location, date, cases, deaths, and population.
- CovidVaccinations → contains vaccination records.

# ⚙️SQL Techniques Used

- CAST() and CONVERT() for datatype conversions
- Aggregate functions (SUM, MAX)
- Window functions (OVER, PARTITION BY)
- Common Table Expressions (CTEs)
- Temporary tables
- Views for visualization

# Next step
- Extend analysis with trends over time (moving averages, growth rates)

# SQL Code 

```sql

-- Skills used: Joins, CTE's, Temp Tables, Windows Functions, Aggregate Functions, Creating Views, Converting Data Types


SELECT * FROM portfolioproject.coviddeaths
order by 3, 4;

select * from covidvaccinations
order by 3, 4;

-- Select data that i'm going to be using 
select location, date, total_cases, total_deaths, population
from coviddeaths
order by 1, 2;


-- Looking at Total Cases vs Total Deaths
-- Showss likelihood of dying if you contract covid in your country

select location, date, total_cases, total_deaths, (total_deaths/total_cases)*100 As Death_Percentage
from coviddeaths
where location like 'India'
order by 1, 2;

-- Shows what percentage of population infected with Covid

select location, date, population, total_cases, (total_cases/population)*100 As Percent_Population_infected
from coviddeaths
where location LIKE 'India'
order by 1, 2;


-- Looking at Countries with Highest Infection Rate Compared to Population
select location, population, MAX(total_cases) As Highest_Infected_Count, MAX((total_cases/population))*100 AS Percent_Population_infected
from coviddeaths
Group by location, population
order by Percent_Population_infected DESC;


-- Countries with Highest Death Counts per population

Select location, MAX(cast(total_deaths as SIGNED)) As Death_Count
from portfolioproject.coviddeaths
Group by location
order by Death_Count DESC;


-- LET'S Break down by CONTINENT

-- Showing continents with the highest death count per population
Select continent, MAX(cast(total_deaths as SIGNED)) as Death_Count
from coviddeaths
Group by continent
order by Death_Count DESC;

-- Global Numbers

select date, SUM(new_cases), SUM(new_deaths), SUM(new_deaths)/SUM(new_cases)*100 AS Death_Percentage
from coviddeaths
Group by date
order by 1, 2;

-- Total Population vs Vaccinations
-- Shows Percentage of Population that has recieved at least one Covid Vaccine

Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations,
SUM(vac.new_vaccinations) Over (PARTITION BY dea.location ORDER BY dea.location, dea.date) AS Rolling_people_Vaccinated
FROM coviddeaths dea
JOIN covidvaccinations vac
	ON dea.location = vac.location
    and dea.date = vac.date
    order by 1, 2;

-- Using CTE to perform Calculation on Partition By in previous query

With PopvsVac (continent, location, date, population, new_vaccination, Rolling_people_Vaccinated)
as(
Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations,
SUM(vac.new_vaccinations) Over (PARTITION BY dea.location ORDER BY dea.location, dea.date) AS Rolling_people_Vaccinated
FROM coviddeaths dea
JOIN covidvaccinations vac
	ON dea.location = vac.location
    and dea.date = vac.date
)
select *, (Rolling_people_Vaccinated/population)*100 as population_vaccinated_percent
from PopvsVac;

-- Using Temp Table to perform Calculation on Partition By in previous query

DROP TABLE if exists PercentagePopulationVaccinated;
CREATE TABLE PercentagePopulationVaccinated (
  Continent VARCHAR(255),
  Location VARCHAR(255),
  ReportDate DATETIME,
  Population BIGINT,
  New_Vaccinations BIGINT,
  RollingPeopleVaccinated BIGINT
);

INSERT INTO PercentagePopulationVaccinated
    value (Continent, Location, ReportDate, Population, New_Vaccinations, RollingPeopleVaccinated);
SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations,
       SUM(CAST(vac.new_vaccinations AS SIGNED))
       OVER (PARTITION BY dea.location ORDER BY dea.date) AS RollingPeopleVaccinated
FROM coviddeaths dea
JOIN covidvaccinations vac
  ON dea.location = vac.location
 AND dea.date = vac.date;
 
Select *, (Rolling_people_Vaccinated/population)*100 as population_vaccinated_percent
from PercentPopulationVaccinated;

-- Creating View to store data for later visualizations
-- Just created one view at this time
create View PercentPopulationVaccinated as 
SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations,
       SUM(CAST(vac.new_vaccinations AS SIGNED))
       OVER (PARTITION BY dea.location ORDER BY dea.date) AS RollingPeopleVaccinated
FROM coviddeaths dea
JOIN covidvaccinations vac
  ON dea.location = vac.location
 AND dea.date = vac.date;
 
 select * from PercentPopulationVaccinated;
```

# 💡Outcome: 
*This project not only strengthened my SQL skills but also showed how raw data can be transformed into meaningful insights.*
