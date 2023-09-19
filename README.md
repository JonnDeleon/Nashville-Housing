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
```
### Reformat Sold As Vacant


