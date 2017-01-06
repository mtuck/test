 
- [BB](#BootBarn)
  - [Thing1](#thing1)
  - [Thing2](#thing2)
- [TPS](#paper)
  - [Distribution Details Missing](#disterr)
  - [Distribution Details Missing Insert](#distinsrt)
  - [Distribution Did Not Freeze](#DistFreeze)

<div id="BootBarn"/>
## BB


<div id="thing1"/>
### Thing 1
```sql
  SELECT        ExtendedPrice, ProductName, UnitPrice, Quantity
FROM            [Order Details Extended]
```
---
<div id="thing2"/>
### Thing 2

---
<div id="paper"/>
## TPS

<div id="disterr"/>
### Distribution Details Missing

This query finds all distros that have been logged into to work on that did not have the distribution details imported

**Check that the date is current, there are some older ones from 2014 and 2015**
```sql
SELECT W.WorkId,W.CheckIn,P.PoDtlId,P.Distribution_Number,   
P.Po_Receipt_Id,P.Sku_Id,P.UpdateChk,P.FinishWork,   
D.DistDtlId,D.PoDtlId
FROM TblPoDtl P  
LEFT JOIN TblDistDtl D on P.PoDtlId=D.PoDtlId  
INNER JOIN TblPoWork W on P.WorkId=W.WorkId
WHERE (W.WorkStatus=1 or W.WorkStatus=2) AND   D.DistDtlId IS NULL
order by CheckIn
```
---

<div id="distinsrt"/>
### Distribution Details Missing Insert

---

<div id="DistFreeze"/>
### Distro Did Not Freeze

--This is for when a distro did not 'Freeze' in Aptos. Change the distro no and work id with the appropriate ones


--Run the statement, and then copy and paste the XML created to the MSMQ site __INSERT URL__


--Use the Business Entity 'Distribution' 


```sql

	declare @parWorkId int, @parDistroNo varchar(20)
	
	set @parDistroNo = 417057
	set @parWorkId = 45133

	Select '1' as CompanyId, 'Freeze' as EntityAction, 'update' as action,
		(select  Rtrim(Distribution_Number) as Number,
			(select 'update' as action, 
					(Select -- dbo.TblPoDtl.Distribution_Number, 
					      Style_Code as StyleCode, Color_Code as ColorCode
					 FROM      dbo.TblPoDtl 
					 WHERE     dbo.TblPoDtl.Distribution_Number = Dist2.Distribution_Number And Style_Code=Dist2.Style_Code And Color_Code=Dist2.Color_Code
					 GROUP BY  dbo.TblPoDtl.Distribution_Number, dbo.TblPoDtl.Style_Code, dbo.TblPoDtl.Color_Code 						
					 FOR XML Raw(''), TYPE, ELEMENTS)
			 from (SELECT     dbo.TblPoDtl.WorkId, dbo.TblPoDtl.Distribution_Number, dbo.TblPoDtl.Style_Code, dbo.TblPoDtl.Color_Code
					FROM      dbo.TblPoDtl 
					GROUP BY dbo.TblPoDtl.WorkId, dbo.TblPoDtl.Distribution_Number, dbo.TblPoDtl.Color_Code, dbo.TblPoDtl.Style_Code) as Dist2
			 where Dist2.Distribution_Number=Dist1.Distribution_Number			
			 FOR XML Raw ('DistributionLine'), TYPE, Root('DistributionLines'))
		 from TblPoDtl  as Dist1
		 Where Distribution_Number=Dist0.Distribution_Number
		 Group BY Distribution_Number
		 FOR XML Raw (''), TYPE, ELEMENTS)	
	from TblPoDtl as Dist0
	Where WorkId=@parWorkId And Distribution_Number=@parDistroNo 
	Group By Distribution_Number
	for xml raw ('Distribution'), TYPE, Root('Distributions')
 ```
 

