# Covid-19 Data Exploration Project
This project involves comprehensive data analysis on Covid-19 statistics using various SQL techniques. The goal is to derive meaningful insights by manipulating and aggregating the data from two main tables: CovidDeaths and CovidVaccinations.

# Skills and Techniques Used
Joins: Combining data from multiple tables

Common Table Expressions (CTEs): Simplifying complex queries

Temporary Tables: Storing intermediate results

Window Functions: Performing calculations across specified partitions of the data

Aggregate Functions: Summarizing data

Data Type Conversions: Ensuring accurate calculations

Creating Views: Streamlining repetitive query structures
# Data Queries and Analysis

Initial Data Selection

Filter and Order Data: Selecting data from CovidDeaths where the continent is not null, and ordering it by specific columns.

Basic Data Retrieval: Retrieving essential columns like Location, Date, Total Cases, New Cases, Total Deaths, and Population.

# Analytical Queries

Total Cases vs Total Deaths:

Calculate the death percentage for countries with "states" in their name to show the likelihood of dying if infected.
Total Cases vs Population:

Determine the percentage of the population infected with Covid-19.
Highest Infection Rates:

Identify countries with the highest infection rates compared to their population.
Highest Death Counts:

Rank countries by total death counts to highlight those most impacted.
Continent Analysis:

Aggregate death counts by continent to identify regions with the highest death tolls.

# Global Summary:

Calculate global totals for new cases and deaths, and derive the global death percentage.

# Vaccination Analysis

Population vs Vaccinations:
Use window functions to calculate the rolling sum of new vaccinations per country and determine the percentage of the population vaccinated.

# Advanced Techniques

Using CTEs:

Simplify the calculation of vaccination percentages by creating a CTE.

Using Temporary Tables:

Store intermediate results in a temporary table for further analysis on vaccination rates.

# Queries

/* Covid 19 Data Exploration 
Skills used: Joins, CTE's, Temp Tables, Windows Functions, Aggregate Functions, Creating Views, Converting Data Types
*/

Select *
From project_1st_Data_Exploring..CovidDeaths
Where continent is not null 
order by 3,4
-- Select Data that we are going to be starting with

Select Location, date, total_cases, new_cases, total_deaths, population
From project_1st_Data_Exploring..CovidDeaths
Where continent is not null 
order by 1,2

-- Total Cases vs Total Deaths
-- Shows likelihood of dying if you contract covid in your country

Select Location, date, total_cases,total_deaths, (total_deaths/total_cases)*100 as DeathPercentage
From project_1st_Data_Exploring..CovidDeaths
Where location like '%states%'
and continent is not null 
order by 1,2

-- Total Cases vs Population
-- Shows what percentage of population infected with Covid

Select Location, date, Population, total_cases,  (total_cases/population)*100 as PercentPopulationInfected
From project_1st_Data_Exploring..CovidDeaths
--Where location like '%states%'
order by 1,2

-- Countries with Highest Infection Rate compared to Population

Select Location, Population, MAX(total_cases) as HighestInfectionCount,  Max((total_cases/population))*100 as PercentPopulationInfected
From project_1st_Data_Exploring..CovidDeaths
--Where location like '%states%'
Group by Location, Population
order by PercentPopulationInfected desc


-- Countries with Highest Death Count per Population

Select Location, MAX(cast(Total_deaths as int)) as TotalDeathCount
From project_1st_Data_Exploring..CovidDeaths
--Where location like '%states%'
Where continent is not null 
Group by Location
order by TotalDeathCount desc

-- BREAKING THINGS DOWN BY CONTINENT

-- Showing contintents with the highest death count per population

Select continent, MAX(cast(Total_deaths as int)) as TotalDeathCount
From project_1st_Data_Exploring..CovidDeaths
--Where location like '%states%'
Where continent is not null 
Group by continent
order by TotalDeathCount desc

-- GLOBAL NUMBERS

Select SUM(new_cases) as total_cases, SUM(cast(new_deaths as int)) as total_deaths, SUM(cast(new_deaths as int))/SUM(New_Cases)*100 as DeathPercentage
From project_1st_Data_Exploring..CovidDeaths
--Where location like '%states%'
where continent is not null 
--Group By date
order by 1,2


-- Total Population vs Vaccinations
-- Shows Percentage of Population that has recieved at least one Covid Vaccine

Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, SUM(CONVERT(int,vac.new_vaccinations)) OVER (Partition by dea.Location Order by dea.location, dea.Date) as RollingPeopleVaccinated
--, (RollingPeopleVaccinated/population)*100
From project_1st_Data_Exploring..CovidDeaths dea
Join project_1st_Data_Exploring..CovidVaccinations vac
	On dea.location = vac.location
	and dea.date = vac.date
where dea.continent is not null 
order by 2,3

-- Using CTE to perform Calculation on Partition By in previous query

With PopvsVac (Continent, Location, Date, Population, New_Vaccinations, RollingPeopleVaccinated)
as
(
Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, SUM(CONVERT(int,vac.new_vaccinations)) OVER (Partition by dea.Location Order by dea.location, dea.Date) as RollingPeopleVaccinated
--, (RollingPeopleVaccinated/population)*100
From project_1st_Data_Exploring..CovidDeaths dea
Join project_1st_Data_Exploring..CovidVaccinations vac
	On dea.location = vac.location
	and dea.date = vac.date
where dea.continent is not null 
--order by 2,3
)
Select *, (RollingPeopleVaccinated/Population)*100
From PopvsVac


-- Using Temp Table to perform Calculation on Partition By in previous query

DROP Table if exists #PercentPopulationVaccinated
Create Table #PercentPopulationVaccinated
(
Continent nvarchar(255),
Location nvarchar(255),
Date datetime,
Population numeric,
New_vaccinations numeric,
RollingPeopleVaccinated numeric
)

Insert into #PercentPopulationVaccinated
Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, SUM(CONVERT(int,vac.new_vaccinations)) OVER (Partition by dea.Location Order by dea.location, dea.Date) as RollingPeopleVaccinated
--, (RollingPeopleVaccinated/population)*100
From project_1st_Data_Exploring..CovidDeaths dea
Join project_1st_Data_Exploring..CovidVaccinations vac
	On dea.location = vac.location
	and dea.date = vac.date
--where dea.continent is not null 
--order by 2,3

Select *, (RollingPeopleVaccinated/Population)*100
From #PercentPopulationVaccinated



