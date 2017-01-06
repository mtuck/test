- [BootBarn](#BootBarn)
  - [Thing1](#thing1)
  - [Thing2](#thing2)
- [The Paper Store](#paper)
  - [Distribution Details Missing](#disterr)


<div id="BootBarn"/>
## Boot Barn


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
## The Paper Store

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
