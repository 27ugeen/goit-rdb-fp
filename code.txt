-- Task 1.
CREATE SCHEMA pandemic;
USE pandemic;

--Task 2.
-- Create table countries
CREATE TABLE countries (
    country_id INT AUTO_INCREMENT PRIMARY KEY,
    Entity VARCHAR(255) NOT NULL,
    Code VARCHAR(10) NOT NULL,
    UNIQUE(Entity, Code)
);
-- Filling with data table countries
INSERT INTO countries (Entity, Code)
SELECT DISTINCT Entity, Code
FROM infectious_cases;

-- Create table cases
CREATE TABLE cases (
    case_id INT AUTO_INCREMENT PRIMARY KEY,
    country_id INT NOT NULL,
    Year INT,
    Number_yaws FLOAT NULL,
    polio_cases FLOAT NULL,
    cases_guinea_worm FLOAT NULL,
    Number_rabies FLOAT NULL,
    Number_malaria FLOAT NULL,
    Number_hiv FLOAT NULL,
    Number_tuberculosis FLOAT NULL,
    Number_smallpox FLOAT NULL,
    Number_cholera_cases FLOAT NULL,
    FOREIGN KEY (country_id) REFERENCES countries(country_id)
);
-- Filling with data table cases
INSERT INTO cases (country_id, Year, Number_yaws, polio_cases, cases_guinea_worm,
                   Number_rabies, Number_malaria, Number_hiv, Number_tuberculosis,
                   Number_smallpox, Number_cholera_cases)
SELECT 
    c.country_id, ic.Year, 
    NULLIF(ic.Number_yaws, ''), 
    NULLIF(ic.polio_cases, ''),
    NULLIF(ic.cases_guinea_worm, ''),
    NULLIF(ic.Number_rabies, ''),
    NULLIF(ic.Number_malaria, ''),
    NULLIF(ic.Number_hiv, ''),
    NULLIF(ic.Number_tuberculosis, ''),
    NULLIF(ic.Number_smallpox, ''),
    NULLIF(ic.Number_cholera_cases, '')
FROM infectious_cases AS ic
JOIN countries AS c ON ic.Entity = c.Entity AND ic.Code = c.Code;

-- Task 3.
SELECT 
    c.Entity,
    c.Code,
    AVG(cs.Number_rabies) AS average_rabies,
    MIN(cs.Number_rabies) AS min_rabies,
    MAX(cs.Number_rabies) AS max_rabies,
    SUM(cs.Number_rabies) AS sum_rabies
FROM 
    cases AS cs
JOIN 
    countries AS c ON cs.country_id = c.country_id
WHERE 
    cs.Number_rabies IS NOT NULL
GROUP BY 
    c.Entity, c.Code
ORDER BY 
    average_rabies DESC
LIMIT 10;

-- Task 4.
SELECT 
    Year,
    first_january_date,
    today_date,
    TIMESTAMPDIFF(YEAR, first_january_date, today_date) AS year_difference
FROM (
    SELECT 
        Year,
        STR_TO_DATE(CONCAT(Year, '-01-01'), '%Y-%m-%d') AS first_january_date,
        CURRENT_DATE() AS today_date
    FROM 
        cases
) AS subquery
LIMIT 10;

-- Task 5.
-- Create function calculate_year_difference
DELIMITER //

CREATE FUNCTION calculate_year_difference(input_year INT)
RETURNS INT
DETERMINISTIC
BEGIN
    DECLARE start_date DATE;
    DECLARE year_diff INT;
    
    SET start_date = STR_TO_DATE(CONCAT(input_year, '-01-01'), '%Y-%m-%d');
    
    SET year_diff = TIMESTAMPDIFF(YEAR, start_date, CURRENT_DATE());
    
    RETURN year_diff;
END;
//

DELIMITER ;

-- Testing function calculate_year_difference
SELECT 
    Year,
    STR_TO_DATE(CONCAT(Year, '-01-01'), '%Y-%m-%d') AS first_january_date,
    CURRENT_DATE() AS today_date,
    calculate_year_difference(Year) AS year_difference
FROM 
    cases
LIMIT 10;

