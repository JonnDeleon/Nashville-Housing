# Nashville-Housing
Cleaning and Restructuring a Nashville Housing Dataset

## Purpose of Project
To create a cleanier, more readable, and easier to understand dataset.

## Languages and Tools Used
This project was completed with the use of Microsoft SQL Server Management Studio

## Process 
- Convert Data to Date Type
- Fill NULLS for Property Address
- Verify that all have TN address
- Split OwnerAddress and PropertyAddres into Street/City for each
- Fill NULLS Acreage
- Change Date Format
- Reformat SoldAsVacant
- Delete Duplicates using a CTE
- Delete Unused Columns

### Convert Data to Data Type
```
select SaleDate, convert(date, SaleDate) as NewDate
from housingData
```
!housingData/DateTypeConvert.png

### Fill NULLS for Property Address

The following code will clear NULL values from Property Address
```
update a
set PropertyAddress = isnull(a.PropertyAddress, b.PropertyAddress)
from housingData a
join housingData b
on  a.ParcelId = b.ParcelID
AND a.UniqueID <> b.UniqueID
where a.PropertyAddress is null
```

### Verify that all have TN Address
The query returned no results
```
select *
from (
select OwnerAddress, substring(OwnerAddress, charindex(',' , OwnerAddress, (charindex(',', OwnerAddress,1))+1) + 1, LEN(PropertyAddress)) as O from housingData)
as a
where a.O not like '%TN%';
```

### Split Owner and Property Address
After creating a new column for both, the following query was used for Property Address Splits
```
-- Property Street
update housingData
set PropertyStreet = parsename(replace(PropertyAddress, ',','.'), 2);

-- Property City
update housingData
set PropertyCity = parsename(replace(PropertyAddress, ',','.'), 1); 
```
Owner Address required a slightly more difficult query
```
-- OWNER STREET
update housingData
set 
OwnerStreet = parsename(
replace(substring(OwnerAddress, -1,  charindex(', TN', OwnerAddress) + 1 ), ',', '.'),2
);

-- OWNER CITY
update housingData
set 
OwnerCity= parsename(
replace(substring(OwnerAddress, -1,  charindex(', TN', OwnerAddress) + 1 ), ',', '.'),1
);
```
### Fill NULLS for acreage
Use PropertyStreet to find if Acreage is NULL for any of its rows
```
select PropertyStreet, Acreage from housingData
where PropertyStreet IN ( select PropertyStreet from housingData where Acreage is null)
and Acreage is not null;
```

The resulting table showed there are NULLs present so they are removed in a identical process to PropertyAddress NULLS


### Change Date Format
```
update housingData
set SaleDate_ = Concat(DATENAME(month, SaleDate), ' ', datepart(day,SaleDate), ',', datepart(year, SaleDate)) from housingData;
```

After creating a new column SaleDate_, fill its values by parting SaleDate and adding month name

### Reformat Sold As Vacant
Some values for SoldAsVacant are registered as 'Y' and 'N' rather than 'Yes' and 'No'.
This is easily fixed by using a case statement.
```
update housingData
set SoldAsVacant =
case 
	when SoldAsVacant like 'Y' then 'Yes'
	when SoldAsVacant like 'N' then 'No'
	else SoldAsVacant
	end

select distinct SoldAsVacant from housingData; 
```
Now there are no 'Y' and 'N' values present.

### Remove Duplicates
By using a CTE, check the ParcelID, PropertyStreet, SaleDate_, SalePrice, and LegalReferance, if rowN is greater than 1, then that is a duplicate row.

```
with dupe_cte as (
select *, 
	row_number() over ( 
	partition by ParcelID, PropertyStreet, SaleDate_, SalePrice, LegalReference 
	order by UniqueID
	) rowN
from  housingData )
delete
from dupe_cte 
where rowN > 1;
```

### Delete Unused Columns
```
alter table housingData
drop column OwnerAddress, PropertyAddress, SaleDate
```

## The Process is now complete!
The new dataset can be downloaded as the housingData_clean.xls


