# SQL_Banking_Explore
Use SQL to query and analyze data related to savings accounts and generate bank capital mobilization reports

## **1. INTRODUCTION**

The data includes detailed information about customers' savings accounts, such as customer ID, deposit date, maturity date, interest rate, principal amount, interest amount, and account status. Some key points in the data are:

- Deposit and maturity dates: Track the start and maturity dates of the deposits.
- Interest rate: The interest rate applied to each deposit period.
- Total deposit and interest amount: The initial amount deposited and the interest generated from that amount.
- Automatic renewal: Some accounts are set to automatically renew upon maturity.

## **2. THE GOAL OF PROJECT**

 The overall purpose is to help the bank better manage the capital raised through savings accounts, forecast liquidity needs, and analyze customer behavior based on deposit size.
 
## **3. READ AND EXPLAIN DATASET**

| Field Name         |Data Type       |Description|
| :-----------------:|:--------------:|:---------------:|
|Cust_ID	           | NVARCHAR(12)	  |Customer ID generated by the IT system when a new customer is added.|
|Saving_account_ID	 | NVARCHAR(12)	  |Savings account ID automatically generated by the IT system.|
|Sav_date	           | Date	          |Date when the savings account was opened.|
|Sav_end_date	       | Date	          |The maturity date of the savings account.|
|Interest	           | Numeric(5,2)	  |Interest rate, with two decimal places.|
|Value	             | Numeric(18,0)	|Amount deposited by the customer.|
|Sotien_lai	         | Numeric(18,0)	|Interest amount the bank has to pay.|
|Sotien_thucnhan	   | Numeric(18,0)	|The total amount the customer will receive upon maturity.|
|Auto_extend	       | Int	          |Whether the account is set to auto-renew: 0 - No, 1 - Yes.|
|Status	             | Int	          |Account status: 0 - Not settled, 1 - Settled.|
|Withdraw_day	       | Date	          |Actual date when the money is withdrawn.|

## **4. EXPLORING DATASET**

***Query 01:*** **Report on Capital Mobilization in the First 6 Months of the Year**

**CODE:**
```sql

-------------------------------------------
/*TẠO DATA ĐỂ TRUY VẤN*/
-------------------------------------------
-- Đặt tên Data là DATA_TKTIETKIEM
WITH DATA_TKTIETKIEM AS
	(
	SELECT * FROM
		(SELECT * FROM
			-- ĐẦU KỲ
			(
				SELECT 
					Cust_ID						AS [DAUKY_MKH]
					,Saving_account_ID			AS [DAUKY_MATK]
					,Value						AS [DAUKY_TONGTIENTK]
					,Sav_Date					AS [DAUKY_NGAYBATDAU]
					,Sav_end_date				AS [DAUKY_NGAYTATTOAN]
				FROM SAVING_ACCOUNT
				WHERE Sav_Date <= '2023-12-31' AND (Sav_end_date > '2023-12-31' OR withdraw_day > '2023-12-31')
			-- MỞ MỚI
			FULL OUTER JOIN
			(
				SELECT 
					Cust_ID						AS [MOMOI_MKH]
					,Saving_account_ID			AS [MOMOI_MATK]
					,Value						AS [MOMOI_TONGTIENTK]
					,Sav_Date					AS [MOMOI_NGAYBATDAU]
					,Sav_end_date				AS [MOMOI_NGAYTATTOAN]
				FROM SAVING_ACCOUNT
				WHERE Sav_Date BETWEEN '2024-01-01' AND '2024-06-30'  		
			) MOMOI
			ON DAUKY.DAUKY_MATK = MOMOI.MOMOI_MATK) AS X
			
			-- TẤT TOÁN
			LEFT JOIN
			(
				SELECT 
					Cust_ID						AS [TATTOAN_MKH]
					,Saving_account_ID			AS [TATTOAN_MATK]
					,Value						AS [TATTOAN_TONGTIENTK]
					,Sav_Date					AS [TATTOAN_NGAYBATDAU]
					,Sav_end_date				AS [TATTOAN_NGAYTATTOAN]
				FROM SAVING_ACCOUNT
				WHERE Sav_end_date BETWEEN '2024-01-01' AND '2024-06-30'
					OR withdraw_day BETWEEN '2024-01-01' AND '2024-06-30'	
			) TATTOAN
			ON ISNULL(X.DAUKY_MATK,'') + ISNULL(X.MOMOI_MATK,'') = TATTOAN.TATTOAN_MATK
			-- CUỐI KỲ
			LEFT JOIN
			(
				SELECT
					Cust_ID						AS [CUOIKY_MKH]
					,Saving_account_ID			AS [CUOIKY_MATK]
					,Value						AS [CUOIKY_TONGTIENTK]
					,Sav_Date					AS [CUOIKY_NGAYBATDAU]
					,Sav_end_date				AS [CUOIKY_NGAYTATTOAN]
				FROM SAVING_ACCOUNT
				WHERE Sav_date <= '2024-06-30' AND Sav_end_date > '2024-06-30'			
			) CUOIKY
			ON ISNULL(X.DAUKY_MATK,'') + ISNULL(X.MOMOI_MATK,'') = CUOIKY.CUOIKY_MATK
)
--drop table DATA_TKTIETKIEM_FINAL
-- ĐẶT TÊN CHO BẢNG DỮ LIỆU FINAL
SELECT * 
INTO DATA_TKTIETKIEM_FINAL
FROM DATA_TKTIETKIEM

SELECT * FROM DATA_TKTIETKIEM_FINAL
----------------------------------------------------
-- BÁO CÁO TÌNH HÌNH HUY ĐỘNG VỐN 6 THÁNG ĐẦU NĂM
----------------------------------------------------

-- TẠO BẢNG BÁO CÁO THEO TEMPLATE
DROP TABLE [BÁO CÁO TÌNH HÌNH HUY ĐỘNG VỐN 6 THÁNG ĐẦU NĂM]
CREATE TABLE [BÁO CÁO TÌNH HÌNH HUY ĐỘNG VỐN 6 THÁNG ĐẦU NĂM]
(
   STT									Int
  ,[NỘI DUNG]							Nvarchar(100)
  ,[TẠI THỜI ĐIỂM 31/12/2023]			bigint
  ,[MỞ MỚI_Q1_24]						bigint
  ,[TẤT TOÁN_Q1_24]						bigint
  ,[MỞ MỚI_Q2_24]						bigint
  ,[TẤT TOÁN_Q2_24]						bigint
  ,[TẠI THỜI ĐIỂM 30/06/2024]			bigint
);

INSERT INTO [BÁO CÁO TÌNH HÌNH HUY ĐỘNG VỐN 6 THÁNG ĐẦU NĂM]
(
	 STT
	,[NỘI DUNG]
	,[TẠI THỜI ĐIỂM 31/12/2023]
	,[MỞ MỚI_Q1_24]
	,[TẤT TOÁN_Q1_24]
	,[MỞ MỚI_Q2_24]
	,[TẤT TOÁN_Q2_24]
	,[TẠI THỜI ĐIỂM 30/06/2024]
)
VALUES
 (1,N'Số khoản tiết kiệm',NULL,NULL,NULL,NULL,NULL,NULL)
,(2,N'Số lượng khách hàng',NULL,NULL,NULL,NULL,NULL,NULL)
,(3,N'Tổng tiền gửi tiết kiệm',NULL,NULL,NULL,NULL,NULL,NULL)

-- I. UPDATE CÁC THÔNG TIN LIÊN QUAN ĐẾN SỐ KHOẢN TIẾT KIỆM (SKTK)
-- ĐẦU KỲ:
UPDATE [BÁO CÁO TÌNH HÌNH HUY ĐỘNG VỐN 6 THÁNG ĐẦU NĂM]
SET [TẠI THỜI ĐIỂM 31/12/2023] = (
CASE WHEN [NỘI DUNG] = N'Số khoản tiết kiệm'		THEN (SELECT COUNT(DISTINCT(DAUKY_MATK)) FROM DATA_TKTIETKIEM_FINAL)
	 WHEN [NỘI DUNG] = N'Số lượng khách hàng'		THEN (SELECT COUNT(DISTINCT(DAUKY_MKH))  FROM DATA_TKTIETKIEM_FINAL)
	 WHEN [NỘI DUNG] = N'Tổng tiền gửi tiết kiệm'	THEN (SELECT SUM(DAUKY_TONGTIENTK)		 FROM DATA_TKTIETKIEM_FINAL) 
	 END
)	
-- MỞ MỚI
UPDATE [BÁO CÁO TÌNH HÌNH HUY ĐỘNG VỐN 6 THÁNG ĐẦU NĂM]
SET [MỞ MỚI_Q1_24] = (
CASE WHEN [NỘI DUNG] = N'Số khoản tiết kiệm'		THEN (SELECT COUNT(DISTINCT(MOMOI_MATK)) FROM DATA_TKTIETKIEM_FINAL WHERE DAUKY_MATK IS NULL AND DATEPART(QUARTER,MOMOI_NGAYBATDAU) = 1)
	 WHEN [NỘI DUNG] = N'Số lượng khách hàng'		THEN 
		(SELECT 
			COUNT(MOMOI_MKH) AS [MOMOI_SOLUONGKH]
		FROM
			(SELECT
				DISTINCT(MOMOI_MKH)
			FROM DATA_TKTIETKIEM_FINAL
			WHERE DATEPART(QUARTER,MOMOI_NGAYBATDAU) = 1) MM
			LEFT JOIN
			(SELECT
				DISTINCT(DAUKY_MKH)
			FROM DATA_TKTIETKIEM_FINAL) DK
			ON MM.MOMOI_MKH = DK.DAUKY_MKH
			WHERE DK.DAUKY_MKH IS NULL)
	 WHEN [NỘI DUNG] = N'Tổng tiền gửi tiết kiệm'	THEN (SELECT SUM(MOMOI_TONGTIENTK) FROM DATA_TKTIETKIEM_FINAL WHERE DAUKY_MATK IS NULL AND DATEPART(QUARTER,MOMOI_NGAYBATDAU) = 1) 
	 END
)	
UPDATE [BÁO CÁO TÌNH HÌNH HUY ĐỘNG VỐN 6 THÁNG ĐẦU NĂM]
SET [MỞ MỚI_Q2_24] = (
CASE WHEN [NỘI DUNG] = N'Số khoản tiết kiệm'		THEN (SELECT COUNT(DISTINCT(MOMOI_MATK)) FROM DATA_TKTIETKIEM_FINAL WHERE DAUKY_MATK IS NULL AND DATEPART(QUARTER,MOMOI_NGAYBATDAU) = 2)
	 WHEN [NỘI DUNG] = N'Số lượng khách hàng'		THEN 
		(SELECT 
    COUNT(MOMOI_MKHQ2) AS [MOMOI_SOLUONGKH]
FROM
    (SELECT DISTINCT(MOMOI_MKH) AS [MOMOI_MKHQ2]
     FROM DATA_TKTIETKIEM_FINAL
     WHERE DATEPART(QUARTER, MOMOI_NGAYBATDAU) = 2) MM
LEFT JOIN
    (SELECT DISTINCT(DAUKY_MKH)
     FROM DATA_TKTIETKIEM_FINAL) DK
ON MM.MOMOI_MKHQ2 = DK.DAUKY_MKH
LEFT JOIN
    (SELECT DISTINCT(MOMOI_MKH) AS [MOMOI_MKHQ1]
     FROM DATA_TKTIETKIEM_FINAL
     WHERE DATEPART(QUARTER, MOMOI_NGAYBATDAU) = 1) MM_Q1
ON MM.MOMOI_MKHQ2 = MM_Q1.MOMOI_MKHQ1
WHERE DK.DAUKY_MKH IS NULL
  AND MM_Q1.MOMOI_MKHQ1 IS NULL
)
	 WHEN [NỘI DUNG] = N'Tổng tiền gửi tiết kiệm'	THEN (SELECT SUM(MOMOI_TONGTIENTK) FROM DATA_TKTIETKIEM_FINAL WHERE DAUKY_MATK IS NULL AND DATEPART(QUARTER,MOMOI_NGAYBATDAU) = 2) 
	 END
)	
-- TẤT TOÁN:
UPDATE [BÁO CÁO TÌNH HÌNH HUY ĐỘNG VỐN 6 THÁNG ĐẦU NĂM]
SET [TẤT TOÁN_Q1_24] = (
CASE WHEN [NỘI DUNG] = N'Số khoản tiết kiệm'		THEN (SELECT COUNT(DISTINCT(TATTOAN_MATK)) FROM DATA_TKTIETKIEM_FINAL WHERE CUOIKY_MATK IS NULL AND DATEPART(QUARTER,TATTOAN_NGAYTATTOAN) = 1 )
	 WHEN [NỘI DUNG] = N'Số lượng khách hàng'		THEN 
		(SELECT 
			COUNT(TATTOAN_MKH) AS [TATTOAN_SOLUONGKH]
		FROM
			(SELECT
				DISTINCT(TATTOAN_MKH)
			FROM DATA_TKTIETKIEM_FINAL
			WHERE DATEPART(QUARTER,TATTOAN_NGAYTATTOAN) = 1) TT
			LEFT JOIN
			(SELECT
				DISTINCT(CUOIKY_MKH)
			FROM DATA_TKTIETKIEM_FINAL) CK
			ON TT.TATTOAN_MKH = CK.CUOIKY_MKH
			WHERE CK.CUOIKY_MKH IS NULL)
	WHEN [NỘI DUNG] = N'Tổng tiền gửi tiết kiệm'	THEN (SELECT SUM(TATTOAN_TONGTIENTK) FROM DATA_TKTIETKIEM_FINAL WHERE CUOIKY_MATK IS NULL AND DATEPART(QUARTER,TATTOAN_NGAYTATTOAN) = 1) 
	END
)
-- Q2/2024
UPDATE [BÁO CÁO TÌNH HÌNH HUY ĐỘNG VỐN 6 THÁNG ĐẦU NĂM]
SET [TẤT TOÁN_Q2_24] = (
CASE WHEN [NỘI DUNG] = N'Số khoản tiết kiệm'		THEN (SELECT COUNT(DISTINCT(TATTOAN_MATK)) FROM DATA_TKTIETKIEM_FINAL WHERE CUOIKY_MATK IS NULL AND DATEPART(QUARTER,TATTOAN_NGAYTATTOAN) = 2)
	 WHEN [NỘI DUNG] = N'Số lượng khách hàng'		THEN 
		(SELECT 
			COUNT(TATTOAN_MKH) AS [TATTOAN_SOLUONGKH]
		FROM
			(SELECT
				DISTINCT(TATTOAN_MKH)
			FROM DATA_TKTIETKIEM_FINAL
			WHERE DATEPART(QUARTER,TATTOAN_NGAYTATTOAN) = 2) TT
			LEFT JOIN
			(SELECT
				DISTINCT(CUOIKY_MKH)
			FROM DATA_TKTIETKIEM_FINAL) CK
			ON TT.TATTOAN_MKH = CK.CUOIKY_MKH
			WHERE CK.CUOIKY_MKH IS NULL)
	WHEN [NỘI DUNG] = N'Tổng tiền gửi tiết kiệm'	THEN (SELECT SUM(TATTOAN_TONGTIENTK) FROM DATA_TKTIETKIEM_FINAL WHERE CUOIKY_MATK IS NULL AND DATEPART(QUARTER,TATTOAN_NGAYTATTOAN) = 2) 
	END
)	
-- CUỐI KỲ:
UPDATE [BÁO CÁO TÌNH HÌNH HUY ĐỘNG VỐN 6 THÁNG ĐẦU NĂM]
SET [TẠI THỜI ĐIỂM 30/06/2024] = (
CASE WHEN [NỘI DUNG] = N'Số khoản tiết kiệm'		THEN (SELECT COUNT(DISTINCT(CUOIKY_MATK)) FROM DATA_TKTIETKIEM_FINAL)
	 WHEN [NỘI DUNG] = N'Số lượng khách hàng'		THEN (SELECT COUNT(DISTINCT(CUOIKY_MKH))  FROM DATA_TKTIETKIEM_FINAL)
	 WHEN [NỘI DUNG] = N'Tổng tiền gửi tiết kiệm'	THEN (SELECT SUM(CUOIKY_TONGTIENTK)		  FROM DATA_TKTIETKIEM_FINAL) 
	 END
)

-- KẾT QUẢ:
SELECT * FROM [BÁO CÁO TÌNH HÌNH HUY ĐỘNG VỐN 6 THÁNG ĐẦU NĂM]
```
**RESULT**

| STT | NỘI DUNG | 31-12-2023 |QUÝ 1/2024_MỞ MỚI|QUÝ 1/2024_TẤT TOÁN|QUÝ 2/2024_MỞ MỚI|QUÝ 2/2024_TẤT TOÁN|30-06-2024|
| :------------: | :-------------: | :-----: | :-----: | :-----: | :-----: | :-----: | :-----: |
| 1 | Số khoản tiết kiệm | 11 | 25 | 2 | 24 | 3 | 55 |
| 2 | Số lượng khách hàng | 7 | 19 | 0 | 8 | 0 | 34 |
| 3 | Tổng tiền tiết kiệm | 4.424.952.249 | 56.175.392.000 | 900.000.000 | 17.133.670.000 | 650.000.000 | 76.184.014.249 |

**=>** **The data shows fluctuations in the activity of opening new and settling savings accounts during the first half of 2024:**

**- Number of savings accounts:**

+ At the start (31-12-2023), there were 11 savings accounts.
+ In Q1/2024: 25 new accounts were opened, and 2 accounts were settled, leaving 34 accounts.
+ In Q2/2024: 24 new accounts were opened, and 3 accounts were settled, ending with 55 savings accounts by 30-06-2024.
  
=> Insight: The number of savings accounts consistently increased, indicating a trend of more new accounts being opened than settled, with the total rising from 11 to 55 over 6 months.

**- Number of customers:**

+ At the start, there were 7 customers.
+ In Q1/2024: 19 new customers were added, with no settlements.
+ In Q2/2024: 8 new customers were added, with no settlements, resulting in 34 customers by the end of the period.
  
=> Insight: The number of customers steadily increased with no drop-offs, almost quintupling over 6 months.

**- Total savings amount:**

+ Starting with 4.42 billion VND.
+ In Q1/2024: 56.18 billion VND in new savings were added, 900 million VND was settled, resulting in 59.70 billion VND by the end of the quarter.
+ In Q2/2024: An additional 17.13 billion VND in new savings were added, with 650 million VND settled, bringing the total to 76.18 billion VND by 30-06-2024.
  
=> Insight: The total savings amount grew significantly from 4.42 billion to 76.18 billion over 6 months, reflecting substantial growth, especially in the mobilization of new funds.

**=> Summary:** The number of customers, savings accounts, and the total savings amount all showed steady and significant growth during the first half of 2024, indicating a positive trend in capital mobilization.

***Query 02:*** **Report on Settlement Situation in the First 6 Months of the Year**

**CODE:**
```sql

-- TẠO BẢNG BÁO CÁO THEO TEMPLATE
DROP TABLE [BÁO CÁO TÌNH HÌNH TẤT TOÁN 6T CUỐI NĂM]
CREATE TABLE [BÁO CÁO TÌNH HÌNH TẤT TOÁN 6T CUỐI NĂM](
   STT					Int
  ,[TIÊU CHÍ]			Nvarchar(100)
  ,[QUÝ I]			bigint
  ,[QUÝ II]				bigint
  ,[QUÝ III]			bigint
  ,[QUÝ IV]				bigint
);
 
-- INSERT CÁC THÔNG TIN CỐ ĐỊNH VÀO BÁO CÁO

INSERT INTO [BÁO CÁO TÌNH HÌNH TẤT TOÁN 6T CUỐI NĂM]
(
	 STT
	,[TIÊU CHÍ]
	,[QUÝ I]
	,[QUÝ II]	
	,[QUÝ III]
	,[QUÝ IV]
)
VALUES
 (1,N'Số tiền gốc phải trả',NULL,NULL,NULL,NULL)
,(2,N'Số tiền lãi phải trả',NULL,NULL,NULL,NULL)
,(3,N'Tổng',NULL,NULL,NULL,NULL)

UPDATE [BÁO CÁO TÌNH HÌNH TẤT TOÁN 6T CUỐI NĂM]
	SET [QUÝ I] =
	(
		CASE WHEN [TIÊU CHÍ] = N'Số tiền gốc phải trả' THEN (SELECT SUM(Value) FROM SAVING_ACCOUNT WHERE Sav_end_date BETWEEN '2024-01-01' AND '2024-03-31')
		     WHEN [TIÊU CHÍ] = N'Số tiền lãi phải trả' THEN 
				(SELECT 
					ROUND(SUM(((Value * interest/100)/365) * DATEDIFF(DAY,Sav_Date,Sav_end_date)),0) -- Tính toán lãi theo ngày
				FROM SAVING_ACCOUNT
				WHERE DATEPART(QUARTER,SAV_END_DATE) = 1 AND YEAR(Sav_end_date)=2024)
		   	 WHEN [TIÊU CHÍ] = N'Tổng'				   THEN
			 (
			 (SELECT SUM(Value) FROM SAVING_ACCOUNT WHERE Sav_end_date BETWEEN '2024-01-01' AND '2024-03-31')
			 +
			 (SELECT 
				ROUND(SUM(((Value * interest/100)/365) * DATEDIFF(DAY,Sav_Date,Sav_end_date)),0)
			 FROM SAVING_ACCOUNT
			 WHERE DATEPART(QUARTER,SAV_END_DATE) = 1 AND YEAR(Sav_end_date)=2024)
			 )
			 END
	)

UPDATE [BÁO CÁO TÌNH HÌNH TẤT TOÁN 6T CUỐI NĂM]
	SET [QUÝ II] =
	(
		CASE WHEN [TIÊU CHÍ] = N'Số tiền gốc phải trả' THEN (SELECT SUM(Value) FROM SAVING_ACCOUNT WHERE Sav_end_date BETWEEN '2024-04-01' AND '2024-06-30')
		     WHEN [TIÊU CHÍ] = N'Số tiền lãi phải trả' THEN 
				(SELECT 
					ROUND(SUM(((Value * interest/100)/365) * DATEDIFF(DAY,Sav_Date,Sav_end_date)),0) -- Tính toán lãi theo ngày
				FROM SAVING_ACCOUNT
				WHERE DATEPART(QUARTER,SAV_END_DATE) = 2 AND YEAR(Sav_end_date)=2024)
		   	 WHEN [TIÊU CHÍ] = N'Tổng'				   THEN
			 (
			 (SELECT SUM(Value) FROM SAVING_ACCOUNT WHERE Sav_end_date BETWEEN '2024-04-01' AND '2024-06-30')
			 +
			 (SELECT 
				ROUND(SUM(((Value * interest/100)/365) * DATEDIFF(DAY,Sav_Date,Sav_end_date)),0)
			 FROM SAVING_ACCOUNT
			 WHERE DATEPART(QUARTER,SAV_END_DATE) = 2 AND YEAR(Sav_end_date)=2024)
			 )
			 END
	)

UPDATE [BÁO CÁO TÌNH HÌNH TẤT TOÁN 6T CUỐI NĂM]
	SET [QUÝ III] =
	(
		CASE WHEN [TIÊU CHÍ] = N'Số tiền gốc phải trả' THEN (SELECT SUM(Value) FROM SAVING_ACCOUNT WHERE Sav_end_date BETWEEN '2024-07-01' AND '2024-09-30')
		     WHEN [TIÊU CHÍ] = N'Số tiền lãi phải trả' THEN 
				(SELECT 
					ROUND(SUM(((Value * interest/100)/365) * DATEDIFF(DAY,Sav_Date,Sav_end_date)),0) -- Tính toán lãi theo ngày
				FROM SAVING_ACCOUNT
				WHERE DATEPART(QUARTER,SAV_END_DATE) = 3 AND YEAR(Sav_end_date)=2024)
		   	 WHEN [TIÊU CHÍ] = N'Tổng'				   THEN
			 (
			 (SELECT SUM(Value) FROM SAVING_ACCOUNT WHERE Sav_end_date BETWEEN '2024-07-01' AND '2024-09-30')
			 +
			 (SELECT 
				ROUND(SUM(((Value * interest/100)/365) * DATEDIFF(DAY,Sav_Date,Sav_end_date)),0)
			 FROM SAVING_ACCOUNT
			 WHERE DATEPART(QUARTER,SAV_END_DATE) = 3 AND YEAR(Sav_end_date)=2024)
			 )
			 END
	)

UPDATE [BÁO CÁO TÌNH HÌNH TẤT TOÁN 6T CUỐI NĂM]
	SET [QUÝ IV] =
	(
		CASE WHEN [TIÊU CHÍ] = N'Số tiền gốc phải trả' THEN (SELECT SUM(Value) FROM SAVING_ACCOUNT WHERE Sav_end_date BETWEEN '2024-10-01' AND '2024-12-31')
		     WHEN [TIÊU CHÍ] = N'Số tiền lãi phải trả' THEN 
				(SELECT 
					ROUND(SUM(((Value * interest/100)/365) * DATEDIFF(DAY,Sav_Date,Sav_end_date)),0) -- Tính toán lãi theo ngày
				FROM SAVING_ACCOUNT
				WHERE DATEPART(QUARTER,SAV_END_DATE) = 4 AND YEAR(Sav_end_date)=2024)
		   	 WHEN [TIÊU CHÍ] = N'Tổng'				   THEN
			 (
			 (SELECT SUM(Value) FROM SAVING_ACCOUNT WHERE Sav_end_date BETWEEN '2024-10-01' AND '2024-12-31')
			 +
			 (SELECT 
				ROUND(SUM(((Value * interest/100)/365) * DATEDIFF(DAY,Sav_Date,Sav_end_date)),0)
			 FROM SAVING_ACCOUNT
			 WHERE DATEPART(QUARTER,SAV_END_DATE) = 4 AND YEAR(Sav_end_date)=2024)
			 )
			 END
	)

-- KẾT QUẢ

SELECT * FROM [BÁO CÁO TÌNH HÌNH TẤT TOÁN 6T CUỐI NĂM]
```
**RESULT**

|      Tiêu chí       |      Quý I       | Quý II    |Quý III| Quý IV|
| :------------:|:-------------:|:-----:|:-----:|:-----:|
|    Số tiền gốc phải trả          |        900,000,000      |  650,000,000    |  1,131,709,472    |1,041,764,834    |
|     Số tiền lãi phải trả         |        192,672,000      |   60,535,397   |  95,909,301    |86,181,066    |
|     Tổng         | 1,092,672,000             |    710,535,397  |  1,227,618,773    |1,127,945,900    |


**- Principal repayment:**

+ Q1: 900 million VND, decreased to 650 million VND in Q2 (a 27% drop).
+ Q3: A sharp increase to 1.13 billion VND (a 74% rise compared to Q2).
+ Q4: Slight decrease to 1.04 billion VND.
  
=> Insight: There are significant fluctuations in principal repayment, particularly the sharp rise in Q3. This may reflect the due dates for large loans or an increase in loan demand.

**- Interest repayment:**

+ Q1: 192.672 million VND, dropped significantly to 60.535 million VND in Q2 (a 69% decrease).
+ Q3 and Q4: Interest rose to 95.909 million VND and 86.181 million VND.
  
=> Insight: The sharp drop in interest repayment in Q2 could be due to partial debt repayment or lower interest rates, but the interest amount increased again in Q3 and Q4, possibly due to additional borrowing or long-term loans.

**- Total repayment (principal + interest):**

+ Q1: 1.09 billion VND.
+ Q2: Dropped significantly to 710 million VND (a 35% decrease).
+ Q3: Increased to 1.23 billion VND, then slightly decreased to 1.13 billion VND in Q4.
  
=> Insight: Total repayment reached its lowest point in Q2, then increased again in the second half of the year, possibly due to increased loan demand or new loans being taken.

**=> Overview:** Q2 was a period of significantly reduced financial pressure, but repayments surged in Q3. This indicates a proactive phase of debt and interest repayment early in the year, but financial pressure increased again towards the end, possibly due to additional borrowing or the maturity of loans.

**Query 03:*** **Report on Classification by Value Criteria of Savings Accounts**

**CODE:**
```sql

SELECT 
PHANLOAI
,COUNT(Saving_account_ID) AS [SỐ TÀI KHOẢN TIẾT KIỆM]
,SUM(VALUE) AS [SỐ TIỀN TIẾT KIỆM]
FROM(
	SELECT 
	Saving_account_ID
	,Value
	,CASE WHEN Value < 1000000000 THEN 'DUOI_1_TY' 
		WHEN Value BETWEEN 1000000000 AND 10000000000 THEN 'TU_1_DEN_10TYR'
		WHEN Value BETWEEN 10000000000 AND 50000000000 THEN 'TU_10_DEN_50TYR'
		ELSE 'TREN50TY' END AS [PHANLOAI]
	FROM SAVING_ACCOUNT
	WHERE Sav_Date BETWEEN '2024-01-01' AND '2024-06-30') X
GROUP BY PHANLOAI
```
**RESULT**

|      PHÂN LOẠI       |      SỐ TÀI KHOẢN TIẾT KIỆM       | SỐ TIỀN TIẾT KIỆM    |
| :------------:|:-------------:|:-----:|
|    DUOI_1_TY          |        35      |  9,759,062,000    | 
|     TU_1_DEN_10TYR        |        13     |   48,550,000,000   | 
|     TU_10_DEN_50TYR        | 1            |    	15,000,000,000  | 


**=>** The data shows that the majority of savings accounts are under 1 billion VND, accounting for 71% of total accounts but contributing only 13% to the total savings amount. Meanwhile, accounts ranging from 1 to 10 billion VND make up 27% of the accounts but contribute 66% of the total savings. Notably, only 1 account in the 10 to 50 billion VND range (accounting for 2%) holds 21% of the total savings.

=> Insight: Although small accounts are numerous, they contribute little, whereas larger accounts, though fewer in number, hold a significant portion of the savings. This indicates a concentration of wealth among a small group of individuals with large deposits.





