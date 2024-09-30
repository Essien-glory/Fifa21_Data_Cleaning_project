# Fifa21_Data_Cleaning_project
The importance of data cleaning cannot be overemphasized; it simplifies your workflow and ensures accurate analysis. Before starting any analysis, conducting a thorough evaluation of the dataset is essential to identify and address inconsistencies.
  
Steps Taken to Clean the FIFA 21 Dataset
1. *Identify and Remove Duplicates:* Checked for and eliminated duplicate entries to ensure data integrity.
2. *Remove Unwanted Columns:* Deleted columns that were not relevant to the analysis, columns like shortname, photourl, playerurl.
3. *Remove or Replace Funny Characters:* Cleaned the dataset by removing or replacing any unusual characters that could affect data processing.
4. *Replace Nulls:* Addressed missing values by replacing null entries with appropriate substitutes (e.g 0)
5. *Normalize 'Hits' Column:* Converted values in the 'Hits' column, replacing 'K' (thousands) and ensured no errors remain.


```SQL
--Display all columns in the table
select *
from Fifa
order by ID desc

(A Total of 18976 rows)

--1. Identify duplicates 
select players_name, club, nationality, count(*) as duplicate
from Fifa
group by players_name, club, Nationality
having count(*) > 1

(1 duplicate found, after filtering by club, players_name and nationality)

--Remove duplicates(This query deletes the duplicated row)
with cte as (
    select players_name , club, nationality,
           row_number() over (partition by longname, club, nationality order by (select null)) as rn
    from Fifa
)
delete from cte
where rn > 1;

--2. Drop Irrelevant columns(name, photourl, playerurl,loan_date_end)
alter table fifa
drop column photourl

alter table fifa
drop column name, playerurl

alter table fifa
drop column loan_date_end

--columns like name, photourl, playerurl,loan_date_end were dropped because it contained email address and data we didn't need.

--3. Remame the column 'longname' to Players_name
exec sp_rename 'fifa.longname', 'Players_name', 'column';

--4. Update nulls in the Hits column with 0
select *
from fifa
where Hits is null
--This query lists all the rows with null values

update Fifa
set Hits = coalesce(hits, 0)
--This query updates the null values to 0

--4b. Replace the alphabets in hits column 'Thousands' to 'numeric' i.e 'K', 'O'
select *
from fifa
where hits like '%k%'

select *
from fifa
where hits like '%.%'

--This replaces the characters 
update fifa
set Hits = replace(Hits,'k','00') 

update fifa
set Hits = replace(Hits,'.','')

--5. Identify and Remove funny characters in the name columns
select * from Fifa
where Players_name not like '%[A-Za-z%]';

--replace the character '?'
select * from Fifa
where Players_name like '%?%'

select * from Fifa
where Players_name LIKE '?%'

--Replace the ones that start with '?' with 'S'
update fifa
set Players_name = replace(players_name, '?', 'S')
where Players_name like '?%'

select *
from fifa
where Players_name like 'stefan%'

--Replace the ones that end with ?
select * from Fifa
where Players_name LIKE '%?'

begin tran

update fifa
set Players_name = replace(players_name, '%?', '')
where Players_name like '%?';

rollback;

--Replace the following characters with their correct names
begin tran

update fifa
set Players_name = case
    when Players_name = 'Ciprian Tataru?anu' then 'Ciprian Tatarusanu'
    when Players_name = 'Ionu? Andrei Radu' then 'Ionut Andrei Radu'
    when Players_name = 'Alexandru Mitri?a' then 'Alexandru Mitrita'
    when Players_name = 'Vlad Chiriche?' then 'Vlad Chiriches'
    when Players_name = 'Florin Ni?a' then 'Florin Nita'
    when Players_name = 'Artur Ioni?a' then 'Artur Ionita'
    when Players_name = 'Nicu?or Bancu' then 'Nicusor Bancu'
    when Players_name = 'Tudor Balu?a' then 'Tudor Baluta'
    when Players_name = 'Ionu? Nedelcearu' then 'Ionut Nedelcearu'
    when Players_name = 'Ionu? Pan?îru' then 'Ionut Pantiru'
    when Players_name = 'Drago? Nedelcu' then 'Dragos Nedelcu'
    when Players_name = 'Valentin Cre?u' then 'Valentin Cretu'
    when Players_name = 'Razvan Ple?ca' then 'Razvan Plesca'
    when Players_name = 'Bogdan ?îru' then 'Bogdan Tiru'
    when Players_name = 'Drago? Grigore' then 'Dragos Grigore'
    when Players_name = 'Mihai Bala?a' then 'Mihai Balasa'
    when Players_name = 'George Pu?ca?' then 'George Puscas'
    when Players_name = 'Vasile Mogo?' then 'Vasile Mogos'
    when Players_name = 'Alin To?ca' then 'Alin Tosca'
    when Players_name = 'Alexandru Ma?an' then 'Alexandru Matan'
    else Players_name
end;

rollback

--6. Replace ~ with - in the 'contract' column
update fifa
set contract = replace(Contract,'~','-') 

--7. Remove funny characters from W_F, SM, IR Columns
update fifa
set 
    W_F = replace(W_F, '?', ''),
    SM = replace(SM, '?', ''),
    IR = replace(IR, '?', '') 

```
