 
- [TPS](#paper)
  - [Distribution Details Missing](#disterr)
  - [Distribution Details Missing Insert](#distinsrt)
  - [Distribution Did Not Freeze](#DistFreeze)
  - [Delete Duplicate Lines](#duplicate)
  - [Payment Tech Exceptions](#payexception)

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

This is for when a distro did not 'Freeze' in Aptos. Change the distro no and work id with the appropriate ones


Run the statement, and then copy and paste the XML created to the MSMQ site __INSERT URL__


Use the Business Entity 'Distribution' 


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
---

<div id="duplicate"/>
### Delete Duplicate Lines

This is for when there are duplicate lines that need to be deleted. Run the below query to get the Distribution and Sku Id
for the duplicate lines

```SQL
SELECT COUNT(DistDtlId),D.Distribution_Number,D.Sku_Id,D.Location_Code --as reccnt
FROM TblReserveDtl R
LEFT JOIN TblReserveDistDtl D on R.RWDtlId=D.RwDtlId
WHERE D.DocumentNo=''
GROUP BY D.Distribution_Number,D.Sku_Id,D.Location_Code
HAVING COUNT(DistDtlId) > 1
ORDER BY D.Distribution_Number,D.Sku_Id
```

Once you have gotten the distribution number and sku id, plug them in below and run the below statement

```SQL
SELECT D.DistDtlId,D.Distribution_Number,D.Sku_Id,D.Location_Code,D.CQuantity,D.FinishWork,D.RWWorkId
FROM TblReserveDtl R LEFT JOIN TblReserveDistDtl D on R.RWDtlId=D.RwDtlId
WHERE D.Distribution_Number='422862' and D.Sku_Id IN ('38979200005')
ORDER BY --D.Location_Code--
D.Sku_Id,D.DistDtlId
```
Once you have the DistDtlId for the duplicated lines, plug them in below and run the statement. The issue will then be fixed

```SQL
DELETE FROM TblReserveDistDtl --SELECT * FROM TblReserveDistDtl 
WHERE DistDtlId IN (7398578, 7398579)
```

<div id='payexception'/>
### Payment Tech Exeptions


```sql
SELECT S.CustomerOrderId,S.OrderAmount,S.ShipCost,S.TaxAmount,L.[LineNo],L.ItemDescription,L.SkuId,L.ExtUnitPrice,L.ProdTaxAmount,
     ISNULL(D.DiscountAmount,0) as DiscAmt,((L.ExtUnitPrice+L.ProdTaxAmount)-ISNULL(D.DiscountAmount,0)) as TotalItemPrice,
     L.ShippedDate, S.PaymentechCapture,S.PaymentechProcessStatusCode,S.PaymentechResponseCode,S.PaymentechOrderId,S.PaymentechTransId
FROM ShippedOrders S
INNER JOIN ShippedOrderLines L on S.CustomerOrderId=L.CustomerOrderId
LEFT JOIN OrderDiscounts D on L.CustomerOrderId=D.CustomerOrderId and L.[LineNo]=D.[LineNo]
--Filter out any lines that have been cancelled
WHERE S.CustomerOrderId = 159237 and L.[Status]<>99
```


 If the issue is a discount issue, type in the amount the discount needs to be updated to, the order no, and the line(s) the new discount should be applied to. If no other issues, update the Paymentech Exceptions
 
  ```SQL
UPDATE OrderDiscounts set DiscountAmount=0.59 where CustomerOrderId=158229 and [LineNo]IN (1,2,3)
  ```
Once all issues have been resolved, fill in your username, what the issue was (discount or tax), and the order no


Once the Paymentech Exceptions table gets update then the order will be captured on the next job run  

```SQL 
 UPDATE PaymentechExceptions Set ResolutionDateTime=GETDATE(),ResolutionUser='THEPAPERSTORE\c-leah.bernas',ResolutionNotes='Discount Issue'
 where OrderNo=158229
```

 If the order has discounts applied to it, but the order discounts table is blank, you will have to insert the discount
 records manually. Fill in the OrderNo, LineNo, OrderDiscountAmount, and the Discount Code
 
 ```SQL
  INSERT INTO OrderDiscounts VALUES (133232, 1, 3.99, 'MAY_YC_SOM')
  ```
  


