# Data-cleaning
#A SQL project for data cleaning purposes 


-- Data cleaning small project: we have data about housing in usa city, we're gonna focus on make it readable
----------------------------------------------------------------------------------------------------------
-- showing everything in the data base 
select * from NashvileHousing
----------------------------------------------------------------------------------------------------------
-- standardize date Fromat 
-- making a new column in the table to use it as the new out come for the date 
alter table NashvileHousing  add Saledateconverted date;
-- taking the data from the old column and putting it in the new column
update NashvileHousing set Saledateconverted =convert(date,Saledate)
-- showing the new column with the proper way and the proper date
select Saledateconverted
from NashvileHousing 
----------------------------------------------------------------------------------------------------------
-- populate Proprety address date 
-- showing data
select PropertyAddress from NashvileHousing
-- showing null data 
select PropertyAddress from NashvileHousing where PropertyAddress is null 
-- showing data ordered by parceID 
-- we used ParceID to see if there's duplicate Data 
select * from NashvileHousing order by ParcelID


-- to check the duplicate date 
select a.ParcelID, b.ParcelID,a.PropertyAddress,b.PropertyAddress
from NashvileHousing a 
join NashvileHousing b 
on a.ParcelID = b.ParcelID 
and a.[UniqueID ] <> b.[UniqueID ] -- what we've done here is we joined the table on itself to see the duplicate data through the ParceID and we sorted them using Unique ID to make sure they're duplicated 



-- as we have duplicated null data we need to fill them from the non null data 
select a.ParcelID, b.ParcelID,a.PropertyAddress,b.PropertyAddress, ISNULL(a.PropertyAddress,b.PropertyAddress)
from NashvileHousing a 
join NashvileHousing b 
on a.ParcelID = b.ParcelID 
and a.[UniqueID ] <> b.[UniqueID ] -- what we've done here is we joined the table on itself to see the duplicate data through the ParceID and we sorted them using Unique ID to make sure they're duplicated
where a.PropertyAddress is null 
 
 
 -- let's fill up all the null data we have in Property address
 update a
 set PropertyAddress = ISNULL(a.PropertyAddress,b.PropertyAddress) 
 from NashvileHousing a 
join NashvileHousing b 
on a.ParcelID = b.ParcelID 
and a.[UniqueID ] <> b.[UniqueID ] -- what we've done here is we joined the table on itself to see the duplicate data through the ParceID and we sorted them using Unique ID to make sure they're duplicated
where a.PropertyAddress is null 
----------------------------------------------------------------------------------------------------------
-- the address isn't not readable, so let's make it into readable shape: address, city state 

select PropertyAddress
from NashvileHousing 

select SUBSTRING(PropertyAddress, 1 ,CHARINDEX(',',PropertyAddress)-1)as Address
,SUBSTRING(PropertyAddress, CHARINDEX(',',PropertyAddress)+1,len(PropertyAddress))as City
from NashvileHousing 
-- as we sperated the one column to address and city we need to create two new columns to put the outcome in them 

alter table NashvileHousing  add PropertySplitaddress Nvarchar(255);

-- taking the data from the old column and putting it in the new column

update NashvileHousing set PropertySplitaddress =SUBSTRING(PropertyAddress, 1 ,CHARINDEX(',',PropertyAddress)-1)

alter table NashvileHousing  add PropertySplitCity Nvarchar(255);

-- taking the data from the old column and putting it in the new column

update NashvileHousing set PropertySplitCity =SUBSTRING(PropertyAddress, CHARINDEX(',',PropertyAddress)+1,len(PropertyAddress))


select *
from NashvileHousing 
-- moving to owner address 

Select OwnerAddress from NashvileHousing
 -- we will use a different method than substring which is Parsename, but parsename deals with . and we don't have it in our data, so we will change the , to . 
 Select PARSEname(REPLACE(OwnerAddress,',','.'),1) 
 ,PARSEname(REPLACE(OwnerAddress,',','.'),2) 
 ,PARSEname(REPLACE(OwnerAddress,',','.'),3) 
 from NashvileHousing

 alter table NashvileHousing  add OwnerSplitaddress Nvarchar(255);

-- taking the data from the old column and putting it in the new column

update NashvileHousing set OwnerSplitaddress =PARSEname(REPLACE(OwnerAddress,',','.'),3) 

alter table NashvileHousing  add OwnerSplitSplitCity Nvarchar(255);

-- taking the data from the old column and putting it in the new column

update NashvileHousing set OwnerSplitSplitCity =PARSEname(REPLACE(OwnerAddress,',','.'),2) 

alter table NashvileHousing  add OwnerSplitSplitstate Nvarchar(255);

-- taking the data from the old column and putting it in the new column

update NashvileHousing set OwnerSplitSplitstate  = PARSEname(REPLACE(OwnerAddress,',','.'),1) 


select *
from NashvileHousing 

----------------------------------------------------------------------------------------------------------
-- making sure that SoladasVavant contains yes or no if it has other words give it a suitable value 

select Distinct(SoldAsVacant)

from NashvileHousing 
-- checking how many values aren't yes and no in the column
select Distinct(SoldAsVacant),COUNT(SoldAsVacant)

from NashvileHousing Group by SoldAsVacant order by SoldAsVacant
-- Change them to yes and no 
Select SoldAsVacant,
	case 
		when SoldAsVacant = 'Y' then 'Yes'
		when  SoldAsVacant = 'N' then'No'
		else SoldAsVacant
		end 
from NashvileHousing 
-- updating the column with the new update
update NashvileHousing set SoldAsVacant  = 
case 
		when SoldAsVacant = 'Y' then 'Yes'
		when  SoldAsVacant = 'N' then'No'
		else SoldAsVacant
		end 

----------------------------------------------------------------------------------------------------------
--Remove the Duplicate from the database 

-- to show the duplicate data 
WITH RownumCTE AS (
Select*,ROW_NUMBER()over(
Partition by ParcelID,PropertyAddress,SaleDate,LegalReference
Order by UniqueId
)row_num

from NashvileHousing)

Select*from RownumCTE where row_num > 1 order by PropertyAddress
-- for deleting the duplicate date 
-- deleting is not the best choice when you're dealing with duplicating data or missing data 
WITH RownumCTE AS (
Select*,ROW_NUMBER()over(
Partition by ParcelID,PropertyAddress,SaleDate,LegalReference
Order by UniqueId
)row_num

from NashvileHousing)
Delete  from RownumCTE where row_num > 1 
----------------------------------------------------------------------------------------------------------
--Delete unused column 
-- not a something would be done to the raw data at least without permission 

Alter table NashvileHousing Drop column OwnerAddress,TaxDistrict,PropertyAddress
Alter table NashvileHousing Drop column SaleDate
-- show the final shape of the database
Select * from NashvileHousing

