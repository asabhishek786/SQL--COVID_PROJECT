-- SELECTING THE DATA WITH ORDER FOR BETTER READABILITY
USE COVID_project;

SELECT * FROM CovidDeaths
ORDER BY location, date;
SELECT * FROM CovidVaccinations
ORDER BY location, date;

-- SELECTING THE DATABASE WHICH WE ARE GOING TO WORK ON
SELECT location, date, population, total_cases, new_cases, total_deaths
FROM CovidDeaths
ORDER BY location, date;

--1. INDIAN COVID DATA WITH RESPECT TO TOTAL DEATHS PERCENTAGE
SELECT location, date, population, total_cases, new_cases, total_deaths, 
CAST((total_deaths/total_cases)*100 AS DECIMAL(10,2)) AS Death_Rate
FROM CovidDeaths
WHERE location = 'India'
ORDER BY location, date;

--2. INDIAN COVID DATA WITH RESPECT TO POPULATION PERCENTAGE
SELECT location, date, population, total_cases, new_cases, total_deaths, 
CAST((total_cases/population)*100 AS Decimal(10,8)) AS Infected_Rate
FROM CovidDeaths
WHERE location = 'India'
ORDER BY location, date;

--3. LOOKING INTO COUNTRY WITH HIGHEST INFECTION RATE PER POPULATION
SELECT location, population, MAX(total_cases) AS Highest_Infection_Rate, 
CAST(MAX(total_cases/population)*100 AS DECIMAL(10,8)) AS Infected_Rate
FROM CovidDeaths
WHERE continent is not null
GROUP BY location, population
ORDER BY Infected_Rate DESC;

--4. SHOWING COUNTRIES WITH HIGHEST DEATH COUNT PER POPULATION
SELECT location, population, MAX(total_deaths) AS Total_Death_Count
FROM CovidDeaths
WHERE continent is not null
GROUP BY location, population
ORDER BY Total_Death_Count DESC;

--5. SHOWING CONTINENT WITH HIGHEST DEATH COUNT
SELECT continent, MAX(total_deaths) AS Total_Death_Count
FROM CovidDeaths
WHERE continent is not null
GROUP BY continent
ORDER BY Total_Death_Count DESC;

-- CREATING VIEW TO STORE DATA 
CREATE VIEW DEATH_COUNT_VIEW AS
SELECT continent, MAX(total_deaths) AS Total_Death_Count
FROM CovidDeaths
WHERE continent is not null
GROUP BY continent;


--6. GLOBAL NUMBERS TOTAL NEW CASES, TOTAL NEW DEATH, DEATH PERCENTAGE
SELECT SUM(CAST(new_cases AS float)) AS Total_New_Cases, SUM(CAST (new_deaths AS float)) AS Total_New_Deaths,
SUM(CAST (new_deaths AS float)) / SUM(CAST(new_cases AS float)) * 100 AS Total_Death_Percentage
FROM CovidDeaths
WHERE continent is not null;
/*-(1.CASTING WAS DONE BECAUSE THE DATATYPE WAS VARCHAR
	2.We could have done the alter statement, but it will change the datatype permanently.
	3.We could have modified the data table as well)*/

--7. GLOBAL NUMBERS FROM BOTH THE TABLES
SELECT cd.continent, cd.location, cd.date, cd.population, cv.new_vaccinations,
	SUM(CONVERT(int,cv.new_vaccinations)) OVER 
	(PARTITION BY cd.location ORDER BY cd.location, cd.date) AS Rolling_People_Vaccinated
FROM CovidDeaths AS cd
INNER JOIN CovidVaccinations AS cv
ON cd.date = cv.date AND cd.location = cv.location
WHERE cd.continent is not null
ORDER BY cd.location, cd.date;

-- USING CTE TO CALCULATE (Rolling_People_Vaccinated/population)*100

WITH Population_VS_Vaccinations(continent, location, date, population, new_vaccinations, Rolling_People_Vaccinated) AS
(
SELECT cd.continent, cd.location, cd.date, cd.population, cv.new_vaccinations,
	SUM(CONVERT(int,cv.new_vaccinations)) OVER 
	(PARTITION BY cd.location ORDER BY cd.location, cd.date) AS Rolling_People_Vaccinated
FROM CovidDeaths AS cd
INNER JOIN CovidVaccinations AS cv
ON cd.date = cv.date AND cd.location = cv.location
WHERE cd.continent is not null
)
SELECT *, CAST((Rolling_People_Vaccinated/population)*100 AS DECIMAL(10,3)) 
FROM Population_VS_Vaccinations;

-- USING TEMP TABLE TO CALCULATE (Rolling_People_Vaccinated/population)*100
-- TO CHANGE SOMETHING IN TABLE: DROP TABLE IF EXIST population_VS_vaccinations 
CREATE TABLE population_VS_vaccinations
(continent VARCHAR(255), 
location VARCHAR(255), 
date DATETIME2, 
population NUMERIC, 
new_vaccinations NUMERIC, 
Rolling_People_Vaccinated NUMERIC
)
INSERT INTO population_VS_vaccinations
SELECT cd.continent, cd.location, cd.date, cd.population, cv.new_vaccinations,
	SUM(CONVERT(int,cv.new_vaccinations)) OVER 
	(PARTITION BY cd.location ORDER BY cd.location, cd.date) AS Rolling_People_Vaccinated
FROM CovidDeaths AS cd
INNER JOIN CovidVaccinations AS cv
ON cd.date = cv.date AND cd.location = cv.location
WHERE cd.continent is not null
SELECT *, (Rolling_People_Vaccinated/population)*100 
FROM Population_VS_Vaccinations;


