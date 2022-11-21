# andreavalen
PORTFOLIO DATA ANALYSIS COVID CASES on SQL

Select *
FROM Covid_Deaths.dbo.CovidDeaths$

--Select data that we are going to be using

Select date, location, total_cases , new_cases , total_deaths , population
FROM Covid_Deaths.dbo.CovidDeaths$
order by 1,2

--Total Cases vs Total Deaths
--Likelihood of dying if you contract covid in ypur country
Select total_cases , population , date, location, total_deaths, (total_deaths/total_cases)*100 as Deathpercentage
FROM Covid_Deaths.dbo.CovidDeaths$
Where location like '%olombia'
order by 1,2


-- Total Cases VS Population
--Shows what percentage of population got covid
Select total_cases , population , date, location, total_deaths, (total_cases/population)*100 as Deathpercentage
FROM Covid_Deaths.dbo.CovidDeaths$
Where location like '%olombia'
order by 1,2


-- Countries with Highest Infection Rate
Select MAX(total_cases)  as HighestInfectionCount, location , population , MAX(Total_cases/population)*100 as InfectionRatePercentage
FROM Covid_Deaths.dbo.CovidDeaths$
Where continent is not NULL
Group by location, population
order by HighestInfectionCount desc

--Showing countries with Highest death count per population

Select MAX(CAST( total_deaths as int))  as HighestDeathCount, location
FROM Covid_Deaths.dbo.CovidDeaths$
Where continent is not NULL
Group by location
order by HighestDeathCount desc

-- Showing Continets with Highest Death count per Population
Select MAX(CAST( total_deaths as int))  as HighestDeathCount, continent
FROM Covid_Deaths.dbo.CovidDeaths$
Where continent is not NULL
Group by  continent
order by HighestDeathCount desc

-- Showing Continets with Highest Infection Rate count per Population
Select MAX(CAST( total_cases as int))  as HighestInfectionRate, continent
FROM Covid_Deaths.dbo.CovidDeaths$
Where continent is not NULL
Group by  continent
order by HighestInfectionRate  desc

--GLOBAL NUMBERS
Select  SUM(new_cases) as totalcases , SUM(cast(new_deaths as int)) as totaldeaths , SUM(cast(new_deaths as int)) / SUM(New_cases)*100 as Deathpercentage
From Covid_Deaths.dbo.CovidDeaths$
where continent is not null
order by 1,2

-- TOTAL VACCINATIONS PER COUNTRY
Select dea.continent , dea.location, dea.date , population , vac.new_vaccinations , SUM(CAST(vac.new_vaccinations as int )) OVER (Partition by dea.location ) as TotalVaccinations
From Covid_Deaths.dbo.CovidVaccinations$ vac
Join Covid_Deaths.dbo.CovidDeaths$ dea
On dea.location = vac.location
and dea.date = vac.date
where dea.continent is not null
order by cast(vac.total_vaccinations as int) desc

--Table TotalVaccinations
With PopVsVac (Continent , Location , Date , Population , New_Vaccinations , TotalVaccinations )
as
(
Select dea.continent , dea.location, dea.date , population , vac.new_vaccinations , SUM(CAST(vac.new_vaccinations as int )) OVER (Partition by dea.location ) as TotalVaccinations
From Covid_Deaths.dbo.CovidVaccinations$ vac
Join Covid_Deaths.dbo.CovidDeaths$ dea
On dea.location = vac.location
and dea.date = vac.date
where dea.continent is not null
--order by 2,3

)

Select location , continent , date , population, new_vaccinations, (TotalVaccinations/Population)*100 as PercentageTotalVacc
From PopVsVac
where continent is not null
order by PercentageTotalVacc desc

--TEMP TABLE
DROP table if exists #PercentagepopulationVaccinated
Create Table #PercentagepopulationVaccinated
(
continent nvarchar (255),
location nvarchar(255),
date datetime,
population numeric,
new_vaccinations numeric,
TotalVaccinations numeric,

)
Insert into #PercentagepopulationVaccinated
Select dea.continent , dea.location, dea.date , population , vac.new_vaccinations , SUM(CAST(vac.new_vaccinations as int )) OVER (Partition by dea.location ) as TotalVaccinations
From Covid_Deaths.dbo.CovidVaccinations$ vac
Join Covid_Deaths.dbo.CovidDeaths$ dea
On dea.location = vac.location
and dea.date = vac.date
where dea.continent is not null
order by cast(vac.total_vaccinations as int) desc

Select location , continent , date , population, new_vaccinations, (TotalVaccinations/Population)*100 as PercentageTotalVacc
From #PercentagepopulationVaccinated
where continent is not null
order by PercentageTotalVacc desc

--Creating Views
Create View PercentagepopulationVaccinated as
Select dea.continent , dea.location, dea.date , population , vac.new_vaccinations , SUM(CAST(vac.new_vaccinations as int )) OVER (Partition by dea.location ) as TotalVaccinations
From Covid_Deaths.dbo.CovidVaccinations$ vac
Join Covid_Deaths.dbo.CovidDeaths$ dea
On dea.location = vac.location
and dea.date = vac.date
where dea.continent is not null
--order by cast(vac.total_vaccinations as int) desc
