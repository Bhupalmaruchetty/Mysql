**World_Layoff_Data Data Cleaning Using Mysql**

**Data Loading**: Imported the layoffs CSV file into MySQL:

 **1.Create Schema**: Set up the database schema and table structure for the world layoffs data.

**2.Prepare CSV**: Ensure the CSV file is clean and properly formatted.

**3.Use Table Data Wizard**: Upload the CSV file into MySQL using the Table 
Data Wizard, which helps map and import the data.

**4. Load Data**: Complete the data import process using the wizard.

**5. Verify Import**: Check the table to ensure all data has been correctly imported.

**1.Retriving Data from table**

select * from layoffs;


**2.Total Rows count**

select count(*)
from layoffs;


 **Data Cleaning Steps**:

   **1.Remove Duplicates**

   **2.Standardize the Data**

   **3.Handling Null Values or Blank values**

   **4.Remove Unnessary columns**


**Stagging and Raw Data set creation**

Create table layoff_stagging
like layoffs;

**Checking layoff_stagging table** 

select * from layoff_stagging;


**Insert Data into stagging or Raw table from Layoffs table**

      Insert layoff_stagging
      select * from layoffs;



**Retriving Data from layoff_stagging table **

      select * from layoff_stagging;


**1.Removing Duplicates**

select *,
row_number() over(
Partition by company,location,industry,total_laid_off,percentage_laid_off, 'date') as row_num
 from layoff_stagging;
 
 
 
**Identifing Duplicate using windows functions and using CTE statement**
 
 WITH duplicate_cte AS
 (select *,
row_number() over(
Partition by company,location,industry,total_laid_off,percentage_laid_off, 'date',stage,country,funds_raised_millions) as row_num
 from layoff_stagging
 )
 select *
 from duplicate_cte
 where row_num >1;
 
 
**Checking particular company name duplicates using Where function**
 
 select * from layoff_stagging
 where company="Ola";
 
 
 
 
**Removing duplicates **
 
  WITH duplicate_cte AS
 (select *,
row_number() over(
Partition by company,location,industry,total_laid_off,percentage_laid_off, 'date',stage,country,funds_raised_millions) as row_num
 from layoff_stagging
 )
 delete 
 from duplicate_cte
 where row_num >1;
 
 
 
 
**Creating one more stagging Table**
 
 CREATE TABLE `layoff_stagging2` (
  `company` text,
  `location` text,
  `industry` text,
  `total_laid_off` int DEFAULT NULL,
  `percentage_laid_off` text,
  `date` text,
  `stage` text,
  `country` text,
  `funds_raised_millions` int DEFAULT NULL,
  `row_num` int
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;



**Checking layoff_stagging2 table**

select * from layoff_stagging2;




**Insert data into stagging2 table to from stagging 1**

insert into layoff_stagging2
select *,
row_number() over(
Partition by company,location,industry,        total_laid_off,percentage_laid_off, 'date',stage,country,funds_raised_millions) as row_num
 from layoff_stagging;

 
 **Retrive Data from layoff_stagging2 **
 select *
 from layoff_stagging2;
 
 
 **Searching for Duplicate values**
 select *
 from layoff_stagging2
 where row_num >1;
 
 
 
 
** Disable safe update mode **

 SET SQL_SAFE_UPDATES = 0;
 
 

 
**Removing Duplicates**
 Delete
 from layoff_stagging2
 where row_num >1;


 **Searching for Duplicate values after deleting **
 select *
 from layoff_stagging2
 where row_num >1;




**2.Standardizing Data**

 **Removing spaces between letters using Trim **

select company,trim(company)
from layoff_stagging2;




**Updating statements**

update layoff_stagging2
set company=trim(company);



** Checking for Duplicate industry **

select distinct industry
from layoff_stagging2
order by 1;




**Retrieving particular industry from industry column**

select
from layoff_stagging2
Where industry like 'crypto%';



**Updating similar industry names**
Update layoff_stagging2
set industry ='crypto'
where industry like 'crypto%';



**Checking layoff_stagging2 table**

select *
from layoff_stagging2;



**Searching for Distinct values in location**

select distinct location
from layoff_stagging2
order by 1;



**Searching for Distinct values in country**

select distinct country
from layoff_stagging2
order by 1;



select *
from layoff_stagging2
where country like 'United States%'
order by 1;




**Using Trim and Trailing function to fix the issue**

select distinct country,trim(trailing '.' from country)
from layoff_stagging2
order by 1;

**updating statement **

update layoff_stagging2
set country =trim(trailing '.' from country)
where country like 'United States%';




**Changing Date formate**

update layoff_stagging2
set date =str_to_date(`date`, '%m/%d/%Y');


**Converting text formate to date formate using** 

str_to_date function on Date column --
  SELECT 
 date,
 CASE 
 WHEN date LIKE '%/%/%' THEN STR_TO_DATE(date, '%m/%d/%Y')
 WHEN date LIKE '%-%-%' THEN STR_TO_DATE(date, '%m-%d-%Y')
 ELSE NULL
 END AS converted_date
FROM 
 layoff_stagging2;


**Update statement for Dates fomating changing**

UPDATE layoff_stagging2
SET date = CASE 
    WHEN date LIKE '%/%/%' THEN STR_TO_DATE(date, '%m/%d/%Y')
    WHEN date LIKE '%-%-%' THEN STR_TO_DATE(date, '%m-%d-%Y')
    ELSE NULL
END;
 
 
 
 
**Changing Data types**
 
 alter table layoff_stagging2
 modify column `date` DATE;
 


select * from
layoff_stagging2;




**3.Handling Null and Blank values **

select * from 
layoff_stagging2
where total_laid_off is null
and percentage_laid_off is null;

**Checking for missing values**

select *
from layoff_stagging2
where industry is null
or industry ='';


**Checking names having null values **

select *
from layoff_stagging2
where company like 'Bally%';




**Update Statement**

update layoff_stagging2
set industry='Travel'
where industry is null;



**self join to fill that blank with relavant industry name**

select t1.industry,t2.industry
from layoff_stagging2 t1
join layoff_stagging2 t2
on t1.company=t2.company
and t1.location=t2.location
where (t1.industry is null or t1.industry='')
and t2.industry is not null;



**Update statement**

update layoff_stagging2 t1
join layoff_stagging2 t2
on t1.company=t2.company
set t1.industry =t2.industry
where t1.industry is null 
and t2.industry is not null;




**Finding null values**

select * from 
layoff_stagging2
where total_laid_off is null
and percentage_laid_off is null;




**Deleting Null values**

Delete from 
layoff_stagging2
where total_laid_off is null
and percentage_laid_off is null;



**Droping Column from the table**
   
alter table layoff_stagging2
 drop column row_num;


**Retrieving layoff_stagging2 Table to final checking **

select * from 
layoff_stagging2;
