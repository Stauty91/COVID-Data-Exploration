Visualization of the data in Tableau after SQL Scripting: https://public.tableau.com/app/profile/nils.stautmeister/viz/CovidDashboard_17229547249380/Dashboard1?publish=yes

-- SQL DATA EXPLORATION SCRIPT 
-- This script is to explore two COVID datasets, joining them together to get them prepared for visualizations

-- Checking import of csv datasets

Select *
From PortfolioProject..CovidDeaths
where continent is not null
order by 3,4

Select *
From PortfolioProject..CovidVaccinations
order by 3,4

-- Selecting the data for usage

Select Location, date, total_cases, new_cases, total_deaths, population
From PortfolioProject..CovidDeaths
order by 1,2

-- Looking at Total Cases vs. Total Deaths in Germany

Select Location, date, total_cases, total_deaths, (total_deaths/total_cases)*100 as DeathPercentage
From PortfolioProject..CovidDeaths
Where location like 'Germany'
and continent is not null
order by 1,2 DESC

-- Looking at Total Cases vs. Population
-- Shows what percentage of population got Covid in Germany

Select Location, date, population, total_cases, (total_cases/population)*100 as InfectedPercentage,
CAST((total_cases/population)*100 AS Decimal (10,6)) as InfectedPercentageFormatted
From PortfolioProject..CovidDeaths
Where location like 'Germany'
and continent is not null
order by 1,2

-- Looking at Countries with highest Infection Rate compared to Population

Select Location, Population, MAX(total_cases) as HighestInfectionCount, MAX((total_cases/population))*100 as InfectedPercentage
From PortfolioProject..CovidDeaths
where continent is not null
Group by Location, population
order by InfectedPercentage desc

-- Showing Countries with Highest Death Count per Population

Select Location, MAX(total_deaths) as TotalDeathCount
From PortfolioProject..CovidDeaths
where continent is not null
Group by Location, population
order by TotalDeathCount desc

-- BREAKING THE DATA DOWN BY CONTINENT

-- Showing continents with the highest death count per population

Select continent, MAX(total_deaths) as TotalDeathCount
From PortfolioProject..CovidDeaths
where continent is not null
Group by continent
order by TotalDeathCount desc

-- GLOBAL NUMBERS

Select date, 
SUM(new_cases) as total_cases, 
SUM(new_deaths) as total_deaths, 
SUM(new_deaths)/SUM(new_cases)*100 as DeathPercentage
--, total_deaths, (total_deaths/total_cases)*100 as DeathPercentage
From PortfolioProject..CovidDeaths
where continent is not null
Group by date
order by 1,2


-- Looking at Total Population vs Vaccications

Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, SUM(vac.new_vaccinations) OVER (Partition by dea.location Order by dea.location, dea.date)
as RollingPeopleVaccinated
From PortfolioProject..CovidDeaths dea
Join PortfolioProject..CovidVaccinations vac
	On dea.location = vac.location
	and dea.date = vac.date 
where dea.continent is not null
order by 5 DESC

-- USING CTE

With PopvsVac (Continent, Location, Date, Population, new_vaccinations, RollingPeopleVaccinated)
as
(
Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, SUM(vac.new_vaccinations) OVER (Partition by dea.location Order by dea.location, dea.date)
as RollingPeopleVaccinated
From PortfolioProject..CovidDeaths dea
Join PortfolioProject..CovidVaccinations vac
	On dea.location = vac.location
	and dea.date = vac.date 
where dea.continent is not null
--order by 7 DESC
)
Select *, (RollingPeopleVaccinated/Population)*100
From PopvsVac


-- USING TEMP TABLE

DROP Table if exists #PercentPopulation
Create Table #PercentPopulation
(
Continent nvarchar(255),
Location nvarchar(255),
Date datetime,
Population numeric,
New_vaccinations numeric,
RollingPeopleVaccinated numeric
)

Insert into #PercentPopulation
Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, SUM(vac.new_vaccinations) OVER (Partition by dea.location Order by dea.location, dea.date)
as RollingPeopleVaccinated
From PortfolioProject..CovidDeaths dea
Join PortfolioProject..CovidVaccinations vac
	On dea.location = vac.location
	and dea.date = vac.date 
--where dea.continent is not null
--order by 5 DESC

Select *, (RollingPeopleVaccinated/Population)*100
From #PercentPopulation

-- Creating View to store data for later visualizations

Create View PercentPopulationVaccinated as
Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, SUM(vac.new_vaccinations) OVER (Partition by dea.location Order by dea.location, dea.date)
as RollingPeopleVaccinated
From PortfolioProject..CovidDeaths dea
Join PortfolioProject..CovidVaccinations vac
	On dea.location = vac.location
	and dea.date = vac.date 
where dea.continent is not null
--order by 5 DESC


Select *
From PercentPopulationVaccinated
