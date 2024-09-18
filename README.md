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

***Query 01:*** Calculate total visit, pageview, transaction for Jan, Feb and March 2017 (order by month)

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

**=>** Monitor capital mobilization: The report tracks the number of savings accounts and the amount of capital raised through the quarters, specifically for Q1 and Q2 of 2024, compared to the situation at the end of the previous year (31-12-23) and the end of the reporting period (30-06-24). This gives the bank an overall view of capital mobilization in the first half of the year.
