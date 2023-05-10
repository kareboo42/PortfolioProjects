select *
from PortfolioProject..['owid-covid-deaths$'] 
order by 3, 4

--select *
--from PortfolioProject..['owid-covid-vaccinations$']
--order by 3, 4

select location, date, total_cases, new_cases, total_deaths, population_density
from PortfolioProject..['owid-covid-deaths$']
order by 1, 2

--looking at total cases vs total death
--shows liklihood of dying if you got covid in your country

select location, date, total_cases,total_deaths, (cast(total_deaths as int)/total_cases)* 100 as DeathPercentage
from PortfolioProject..['owid-covid-deaths$']
where location like '%United States%'
order by 1, 2

--Looking at total cases vs population
--shows what percentage of population got covid

select location, date, total_cases,population_density, (cast(total_cases as int)/population_density)* 100 as PercentPopulationInfected
from PortfolioProject..['owid-covid-deaths$']
--where location like '%United States%'
order by 1, 2

--Looking at countries with highest infection rate compared to population

select location, MAX(total_cases) AS HighestInfectionCount,population_density, max((cast(total_cases as int)/population_density))* 100 as PercentPopulationInfected
from PortfolioProject..['owid-covid-deaths$']
--where location like '%United States%'
group by location, population_density
order by PercentPopulationInfected desc

--Countries with highest death count per population

select location, MAX(cast(total_deaths as int)) as TotalDeathCount
from PortfolioProject..['owid-covid-deaths$']
--where location like '%United States%'
where continent is not null
group by location, population_density
order by TotalDeathCount desc

--Breakdown by continent

select continent, MAX(cast(total_deaths as int)) as TotalDeathCount
from PortfolioProject..['owid-covid-deaths$']
--where location like '%United States%'
where continent is not null
group by continent
order by TotalDeathCount desc

--Global Numbers

select date, SUM(new_cases) as total_cases, SUM(CAST(new_deaths as int)) as total_deaths, SUM(cast(new_deaths as int))/SUM(new_cases)*100 as DeathPercentage
from PortfolioProject..['owid-covid-deaths$']
--where location like '%United States%'
where continent is not null
group by date
order by 1, 2

--Looking at total population vs vaccinations

SELECT dth.continent, dth.location, dth.date, dth.population_density, vac.new_vaccinations,
  SUM(CONVERT(BIGINT, vac.new_vaccinations)) OVER (PARTITION BY dth.location ORDER BY dth.location, dth.date) AS RollingPeopleVaccinated
FROM PortfolioProject..['owid-covid-deaths$'] dth
JOIN PortfolioProject..['owid-covid-vaccinations$'] vac
  ON dth.location = vac.location
  AND dth.date = vac.date
WHERE dth.continent IS NOT NULL
ORDER BY 2, 3;


-- Use CTE

with PopvsVac (Continent, location, date, population_density, new_vaccinations, RollingPeopleVaccinated)
as
(
select dth.continent, dth.location, dth.date, dth.population_density, vac.new_vaccinations,
SUM(convert(bigint,vac.new_vaccinations)) over (partition by dth.location order by dth.date) 
as RollingPeopleVaccinated
from PortfolioProject..['owid-covid-deaths$'] dth
join PortfolioProject..['owid-covid-vaccinations$'] vac
on dth.location = vac.location
and dth.date = vac.date
where dth.continent is not null
)
select *, (RollingPeopleVaccinated/Population_density)*100
from PopvsVac
order by location, date

-- Temp Table

DROP table if exists #PercentPopulationVaccinated 
create table #PercentPopulationVaccinated
(
continent nvarchar(255),
location nvarchar(255),
date datetime,
population numeric,
new_vaccinations numeric,
RollingPeopleVaccinated numeric,
)
Insert into #PercentPopulationVaccinated
select dth.continent, dth.location, dth.date, dth.population_density, vac.new_vaccinations,
SUM(convert(bigint,vac.new_vaccinations)) over (partition by dth.location order by dth.date) 
as RollingPeopleVaccinated
from PortfolioProject..['owid-covid-deaths$'] dth
join PortfolioProject..['owid-covid-vaccinations$'] vac
on dth.location = vac.location
and dth.date = vac.date
where dth.continent is not null

SELECT *, (RollingPeopleVaccinated / NULLIF(Population, 0)) * 100
FROM #PercentPopulationVaccinated
ORDER BY location, date

--Creating view to store data for later visualizations

-- Batch 1: Create the view
USE PortfolioProject;
GO

CREATE VIEW PercentPopulationVaccinated AS 
SELECT dth.continent, dth.location, dth.date, dth.population_density, vac.new_vaccinations,
SUM(CONVERT(BIGINT, vac.new_vaccinations)) OVER (PARTITION BY dth.location ORDER BY dth.date) 
AS RollingPeopleVaccinated
FROM PortfolioProject..['owid-covid-deaths$'] dth
JOIN PortfolioProject..['owid-covid-vaccinations$'] vac
ON dth.location = vac.location
AND dth.date = vac.date
WHERE dth.continent IS NOT NULL;
GO

-- Batch 2: Query the view
SELECT *
FROM PercentPopulationVaccinated;
