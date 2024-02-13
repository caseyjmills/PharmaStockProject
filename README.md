# PharmaStockProject
-- This is a project I completed to familiarize myself furhter with SQL.
-- I have uploaded the database that I created and this README file contains the queries I ran.
-- The project provides an example of how to make informed decisions on investments into Pharmaceutical Companies. I "watched" the stock market, collected data, and performed basic analysis

-- COLLECT DATA --
-- Data from 07/23/2023-01/23-2024 downloaded from Yahoo Finance
-- Date, Open, High, Low, Close, Adj Close, Volume
--			Pfizer Inc. (PFE),
-- 		Johnson & Johnson (JNJ), 
-- 		Merck & Co., Inc. (MRK), 
-- 		Bristol Myers Squibb Company (BMY), 
-- 		Novartis AG (NVS)

CREATE TABLE IF NOT EXISTS stock_data (
    symbol TEXT,
    name TEXT,
    datetime TEXT,
    price REAL
);

-- Import Open prices with datetime yyyy-mm-dd 08:30:00
INSERT INTO stock_data (symbol, name, datetime, price)
SELECT
    'NVS' AS symbol,
    'Novartis AG' AS name,
    Date || ' 08:30:00' AS datetime,
    Open AS price
FROM NVS;

-- Import Adj Close prices with datetime yyyy-mm-dd 15:00:00
INSERT INTO stock_data (symbol, name, datetime, price)
SELECT
    'NVS' AS symbol,
    'Novartis AG' AS name,
    Date || ' 15:00:00' AS datetime,
    AdjClose AS price
FROM NVS;

-- BASIC ANALYSIS --
-- Identify distinct stocks in the table
 SELECT DISTINCT symbol, name FROM stock_data;

-- Query data for the stock with the symbol 'PFE'
SELECT * FROM stock_data WHERE symbol = 'PFE';

-- Price Analysis:
-- 	Rows with price above 100:
SELECT * FROM stock_data WHERE price > 100;
--		Rows with price between 40 and 50:
SELECT * FROM stock_data WHERE price BETWEEN 40 AND 50;

-- Sort the Table by Price
--		Ascending order
SELECT * FROM stock_data ORDER BY price ASC;
--		Descending order
SELECT * FROM stock_data ORDER BY price DESC;

-- Price Analysis
--		Min price
SELECT MIN(price) AS min_price FROM stock_data;
--		Max price
SELECT MAX(price) AS max_price FROM stock_data;
-- 	Median price
SELECT price
FROM stock_data
ORDER BY price
LIMIT 1 OFFSET (SELECT COUNT(*) FROM stock_data) / 2;
-- 	Variance of price
SELECT AVG((price - avg_price) * (price - avg_price)) AS variance
FROM stock_data, (SELECT AVG(price) AS avg_price FROM stock_data);


-- REFACTOR EXAMPLE --
-- Refactor the data into 2 tables
-- 	stock_info to store general info about the stock itself (ie. symbol, name)
-- 	stock_prices to store the collected data on price (ie. symbol, datetime, price)

CREATE TABLE IF NOT EXISTS stock_info (
    symbol TEXT PRIMARY KEY,
    name TEXT
);
INSERT OR IGNORE INTO stock_info (symbol, name)
SELECT DISTINCT symbol, name FROM stock_data;

CREATE TABLE IF NOT EXISTS stock_prices (
    symbol TEXT,
    datetime TEXT,
    price REAL,
    FOREIGN KEY (symbol) REFERENCES stock_info(symbol)
);
INSERT INTO stock_prices (symbol, datetime, price)
SELECT symbol, datetime, price FROM stock_data;

-- Combine data from both the stock_prices and stock_info tables based on a common column
-- 	Common column is symbol
SELECT sp.symbol, si.name, sp.datetime, sp.price
FROM stock_prices sp
JOIN stock_info si ON sp.symbol = si.symbol;
