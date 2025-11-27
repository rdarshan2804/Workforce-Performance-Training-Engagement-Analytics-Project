# Workforce-Performance-Training-Engagement-Analytics-Project
Analyzed Workforce Performance, Training &amp; Engagement data by solving 100 SQL queries and cleaning messy datasets. Built an interactive Power BI dashboard with KPIs, trends, and insights to improve decision-making across recruitment, training outcomes, and employee engagement.

-- employee_data

-- 1.Retrieve the full name and business unit of all active employees.
SELECT CONCAT(FIRSTNAME," ",LASTNAME) AS FULL_NAME,
       BUSINESSUNIT,
       EMPLOYEESTATUS 
       FROM employee_data
       WHERE EmployeeStatus="ACTIVE"
       
-- 2.List employees whose employment duration is more than 5 years.
SELECT EMPLOYEEID,
       FIRSTNAME,
       LASTNAME,
	   TIMESTAMPDIFF(YEAR,STARTDATE,CURRENT_DATE) AS TENURE
       FROM EMPLOYEE_DATA
       WHERE 
	   TIMESTAMPDIFF(YEAR,STARTDATE,CURRENT_DATE)>=5
         
-- 3.Find employees who have no supervisor assigned.
SELECT * FROM EMPLOYEE_DATA
WHERE SUPERVISOR IS NULL
     OR SUPERVISOR= " "
     
-- 4.Count the number of employees in each BusinessUnit.
SELECT BUSINESSUNIT,
       COUNT(*) AS TOTAL_EMPLOYEES
       FROM EMPLOYEE_DATA
	   GROUP BY BUSINESSUNIT

-- 5.Get the average employee rating by DepartmentType.
SELECT DEPARTMENTTYPE,
       AVG(CURRENT_EMPLOYEE_RATING) AS EMPLOYEES_RATING
       FROM EMPLOYEE_DATA
       GROUP BY DEPARTMENTTYPE
       
-- 6.List employees who have “Manager” in their Title.
SELECT * FROM EMPLOYEE_DATA
WHERE TITLE = "MANAGER"

-- 7.Identify employees who exited in the year 2023.
SELECT * FROM EMPLOYEE_DATA
WHERE 
YEAR(EXITDATE) = 2023

-- 8.Show employees whose PayZone is either ‘A’ or ‘B’.
SELECT * FROM EMPLOYEE_DATA
WHERE PAYZONE ="ZONE A" 
	  OR PAYZONE = "ZONE B"
      
-- 9.Retrieve employees who belong to the top 3 highest populated States.
SELECT STATE,
       COUNT(*) AS POPULATION
       FROM EMPLOYEE_DATA
       GROUP BY STATE
       ORDER BY POPULATION DESC
       LIMIT 3
       
-- 10.Find the youngest employee based on DOB.

SELECT *
FROM EMPLOYEE_DATA
ORDER BY DOB DESC
LIMIT 1

-- Performance & Rating

-- 11.Count how many employees have a performance score of “Exceeds Expectations.”
SELECT PERFORMANCE_SCORE,
       COUNT(*) AS EMPLOYEES
       FROM EMPLOYEE_DATA
	   WHERE PERFORMANCE_SCORE = "EXCEEDS"
       GROUP BY PERFORMANCE_SCORE
     
-- 12.Find the average Current_Employee_Rating by Division.
SELECT DIVISION,
       AVG(CURRENT_EMPLOYEE_RATING) AS EMP_RATING
       FROM EMPLOYEE_DATA
       GROUP BY DIVISION
       
-- 13.Get the number of employees with missing performance scores.
SELECT COUNT(*) FROM EMPLOYEE_DATA
WHERE PERFORMANCE_SCORE IS NULL 
OR PERFORMANCE_SCORE = " "

-- 14.Show the top 10 highest-rated employees.
SELECT * FROM EMPLOYEE_DATA
ORDER BY CURRENT_EMPLOYEE_RATING DESC
LIMIT 10

-- 15.Compare average rating between active vs terminated employees.
SELECT EMPLOYEESTATUS,
       AVG(CURRENT_EMPLOYEE_RATING) AS AVG_RATING
       FROM EMPLOYEE_DATA
       GROUP BY EMPLOYEESTATUS
       
-- Date Calculations

-- 16.Calculate employee tenure in years.
SELECT EMPLOYEEID,
       FIRSTNAME,LASTNAME,
       TIMESTAMPDIFF(YEAR,STARTDATE,CURRENT_DATE())AS TENURE FROM EMPLOYEE_DATA
       ORDER BY EMPLOYEEID
       
-- 17.List employees whose tenure is above the average tenure.
SELECT EMPLOYEEID,
       FIRSTNAME,LASTNAME,
       TIMESTAMPDIFF(YEAR,STARTDATE,CURRENT_DATE())AS TENURE FROM EMPLOYEE_DATA
       WHERE TIMESTAMPDIFF(YEAR,STARTDATE,CURRENT_DATE())>(
SELECT AVG(TIMESTAMPDIFF(YEAR,STARTDATE,CURRENT_DATE())) AS AVG_TENURE FROM EMPLOYEE_DATA)

-- 18.Get employees who started in the last 90 days.
SELECT * FROM EMPLOYEE_DATA
WHERE DATEDIFF(CURRENT_DATE(),STARTDATE) <= 90

-- 19.Show how many employees started in each month of 2023.
SELECT MONTH(STARTDATE) AS MONTH,
       COUNT(*) AS EMP_COUNT
       FROM EMPLOYEE_DATA
       WHERE YEAR(STARTDATE)= 2023
       GROUP BY MONTH(STARTDATE)
       ORDER BY MONTH
       
-- 20.Identify the quarter with the highest number of employee exits.
SELECT QUARTER(EXITDATE),
       COUNT(*) FROM EMPLOYEE_DATA
       GROUP BY QUARTER(EXITDATE)

-- Engagement Survey Questions

-- 21.Count the total number of surveys submitted per employee.
SELECT EMPLOYEEID, 
       COUNT(*)AS SURVEY_COUNT
       FROM employee_engagement_survey_data 
       GROUP BY EMPLOYEEID
       
-- 22.Find employees whose Engagement_Score is above average Engagement_Score
SELECT EMPLOYEEID,
       ENGAGEMENT_SCORE FROM  employee_engagement_survey_data 
	   WHERE ENGAGEMENT_SCORE >(
	   SELECT AVG(ENGAGEMENT_SCORE) FROM  employee_engagement_survey_data )

-- 23.Calculate average Engagement_Score by BusinessUnit.
SELECT BUSINESSUNIT,
       AVG(ENGAGEMENT_SCORE) AS AVG_ENGAGEMENT_SCORE
       FROM  employee_engagement_survey_data AS ES
       JOIN EMPLOYEE_DATA AS E
       ON E.EMPLOYEEID=ES.EMPLOYEEID
       GROUP BY BUSINESSUNIT
       
-- 24.Determine the highest average Work_Life_Balance_Score by DepartmentType.
SELECT DEPARTMENTTYPE,
       MAX(WORK_LIFE_BALANCE_SCORE) AS MAX_SCORE
       FROM  employee_engagement_survey_data AS ES
       JOIN EMPLOYEE_DATA AS E
       ON ES.EMPLOYEEID=E.EMPLOYEEID
       GROUP BY DEPARTMENTTYPE
       
-- 25.Identify employees with consistently low Satisfaction_Score (<5).
SELECT EMPLOYEEID,
       SATISFACTION_SCORE FROM  employee_engagement_survey_data 
       WHERE SATISFACTION_SCORE <5
       
-- 26.Show the latest survey result for each employee.
SELECT * FROM (
SELECT EMPLOYEEID,
       SURVEY_DATE,
       RANK()OVER(PARTITION BY EMPLOYEEID ORDER BY SURVEY_DATE DESC) AS RNK 
       FROM  employee_engagement_survey_data ) A
       WHERE A.RNK=1
       
-- 27.Find employees with missing survey date.
SELECT * FROM employee_engagement_survey_data
WHERE SURVEY_DATE IS NULL

-- 28.Compare average engagement of active vs terminated employees.
SELECT E.EMPLOYEESTATUS,
       AVG(ES.ENGAGEMENT_SCORE) AVG_SCORE
       FROM EMPLOYEE_DATA AS E
       JOIN EMPLOYEE_ENGAGEMENT_SURVEY_DATA AS ES
       ON ES.EMPLOYEEID=E.EMPLOYEEID
       GROUP BY E.EMPLOYEESTATUS
       
-- 29.Find correlation-like comparison: list employees whose engagement is below company average but rating is above average.
SELECT E.*
FROM EMPLOYEE_DATA E
JOIN EMPLOYEE_ENGAGEMENT_SURVEY_DATA SD ON E.EMPLOYEEID = SD.EMPLOYEEID
WHERE SD.ENGAGEMENT_SCORE < (SELECT AVG(ENGAGEMENT_SCORE) FROM EMPLOYEE_ENGAGEMENT_SURVEY_DATA)
  AND E.CURRENT_EMPLOYEE_RATING > (SELECT AVG(CURRENT_EMPLOYEE_RATING) FROM EMPLOYEE_DATA);

-- 30.Rank employees by their work_life_balance_score.
SELECT * FROM (
SELECT EMPLOYEEID,  work_life_balance_score,
       RANK() OVER(partition by EMPLOYEEID ORDER BY  work_life_balance_score desc) AS RNK
       FROM employee_engagement_survey_data)A
       WHERE RNK = 1
       
-- Recruitment Data Questions

-- 31.Count applications by Job_Title.
SELECT JOB_TITLE, 
COUNT(*) AS APPLICATION_COUNT
FROM RECRUITMENT_DATA
GROUP BY JOB_TITLE

-- 32.Find the top 5 cities with the highest number of applicants.
SELECT CITY, COUNT(*) AS APPLICANT_COUNT
FROM RECRUITMENT_DATA
GROUP BY CITY
ORDER BY APPLICANT_COUNT DESC
LIMIT 5

-- 33.Calculate the average Desired_Salary by Education_Level.
SELECT EDUCATION_LEVEL,
AVG(DESIRED_SALARY) AS AVG_SALARY
FROM RECRUITMENT_DATA
GROUP BY EDUCATION_LEVEL

-- 34.Show applicants with more than 10 years of experience.
SELECT *
FROM RECRUITMENT_DATA
WHERE YEARS_OF_EXPERIENCE > 10

-- 35.Identify how many applicants were rejected, selected, and are still in process.
SELECT STATUS, 
COUNT(*) AS TOTAL
FROM RECRUITMENT_DATA
GROUP BY STATUS

-- 36.Count male vs female applicants.
SELECT GENDER,
COUNT(*) AS TOTAL
FROM RECRUITMENT_DATA
GROUP BY GENDER

-- 37.Identify applicants older than 30 using Date_of_Birth.
SELECT *
FROM RECRUITMENT_DATA
WHERE TIMESTAMPDIFF(YEAR, Date_of_Birth, CURDATE()) > 30

-- 38.Show the month-wise application trend.
SELECT MONTH(application_date) AS APP_MONTH,
       COUNT(*) AS TOTAL
       FROM RECRUITMENT_DATA
      GROUP BY APP_MONTH

-- 39.Find applicants applying for roles outside their current State.
SELECT *
FROM RECRUITMENT_DATA
WHERE STATE <> COUNTRY

-- 40.List the applicants whose desired salary is above the  average.
SELECT *
FROM RECRUITMENT_DATA 
WHERE DESIRED_SALARY >
	(SELECT AVG(DESIRED_SALARY)
	FROM RECRUITMENT_DATA)
   
-- Employee vs Recruitment (Cross-analysis)

-- 41.Identify employees who were previously applicants (match via First/Last Name OR Email).
SELECT E.*
FROM EMPLOYEE_DATA E
JOIN RECRUITMENT_DATA R
ON E.ADEMAIL = R.EMAIL

-- 43.show job titles with gender classification among applicants.
select job_title,
       gender,
       count(*) applicants
       from recruitment_data
       group by 1,2

-- 43.Show conversion rate: Applicants → Hires (joined employee_data).
select 
(SELECT COUNT(*) FROM EMPLOYEE_DATA) /
(SELECT COUNT(*) FROM RECRUITMENT_DATA) * 100 AS CONVERSION_RATE

-- 44.List departments with the highest applicant-to-hire ratio.
SELECT E.DEPARTMENTTYPE,
(COUNT(E.EMPLOYEEID) / COUNT(R.APPLICANT_ID)) AS HIRE_RATIO
FROM EMPLOYEE_DATA E
JOIN RECRUITMENT_DATA R
ON E.ADEMAIL = R.EMAIL
GROUP BY E.DEPARTMENTTYPE

-- 45.Find the average experience of applicants who were hired.
SELECT AVG(YEARS_OF_EXPERIENCE) AS AVG_EXP
FROM RECRUITMENT_DATA R
JOIN EMPLOYEE_DATA E
ON E.ADEMAIL = R.EMAIL

-- Training & Development

-- 46.Count number of training programs delivered by each Trainer.
SELECT TRAINER,
COUNT(*) AS TOTAL_TRAININGS
FROM TRAINING_AND_DEVELOPMENT_DATA
GROUP BY TRAINER

-- 47.Calculate the total training cost per BusinessUnit (join employee_data).
SELECT E.BUSINESSUNIT, 
SUM(T.TRAINING_COST) AS TOTAL_COST
FROM TRAINING_AND_DEVELOPMENT_DATA T
JOIN EMPLOYEE_DATA E
ON T.EMPLOYEEID = E.EMPLOYEEID
GROUP BY E.BUSINESSUNIT

-- 48.List employees who completed more than 3 trainings.
SELECT EMPLOYEEID,
COUNT(*) AS TRAINING_COUNT
FROM TRAINING_AND_DEVELOPMENT_DATA
GROUP BY EMPLOYEEID
HAVING COUNT(*) > 3

-- 49.Show the most commonly attended Training_Program_Name.
SELECT TRAINING_PROGRAM_NAME, COUNT(*) AS TOTAL
FROM TRAINING_AND_DEVELOPMENT_DATA
GROUP BY TRAINING_PROGRAM_NAME
ORDER BY TOTAL DESC
LIMIT 1

-- 50.Calculate average Training_Duration_Days by Training_Type.
SELECT TRAINING_TYPE,
AVG(TRAINING_DURATION_DAYS) AS AVG_DURATION
FROM TRAINING_AND_DEVELOPMENT_DATA
GROUP BY TRAINING_TYPE

-- 51.Identify departments with the highest training spending.
SELECT E.DEPARTMENTTYPE,
SUM(TD.TRAINING_COST) AS TOTAL_COST
FROM TRAINING_AND_DEVELOPMENT_DATA AS TD
JOIN EMPLOYEE_DATA AS E
ON TD.EMPLOYEEID = E.EMPLOYEEID
GROUP BY E.DEPARTMENTTYPE

-- 52.Find employees with failed training outcomes.
SELECT *
FROM TRAINING_AND_DEVELOPMENT_DATA
WHERE TRAINING_OUTCOME = "FAILED"

-- 53.Show month-wise training cost trend.
SELECT MONTH(TRAINING_DATE) AS TRAINING_MONTH,
       SUM(TRAINING_COST) AS TOTAL_COST
FROM TRAINING_AND_DEVELOPMENT_DATA
GROUP BY TRAINING_MONTH

-- 54.Get training attended by each employee with days.
SELECT EMPLOYEEID,
	   TRAINING_TYPE,
       TRAINING_DURATION_DAYS
       FROM TRAINING_AND_DEVELOPMENT_DATA
       
-- 55.Identify the most expensive training attended by any employee.
SELECT *
FROM TRAINING_AND_DEVELOPMENT_DATA
ORDER BY TRAINING_COST DESC
LIMIT 1

-- Employee + Training + Performance

-- 56.Compare satisfaction_score with each training_type
SELECT TRAINING_TYPE,
	   AVG(SATISFACTION_SCORE) AS AVG_SCORE 
       FROM employee_engagement_survey_data AS E
       JOIN training_and_development_data AS T
       ON E.EMPLOYEEID=T.EMPLOYEEID
       GROUP BY TRAINING_TYPE

-- 57.Show employees with high training participation but low performance rating.
SELECT E.EMPLOYEEID,
E.CURRENT_EMPLOYEE_RATING,
COUNT(T.EMPLOYEEID) AS TRAININGS
FROM EMPLOYEE_DATA AS E
JOIN TRAINING_AND_DEVELOPMENT_DATA AS T
ON E.EMPLOYEEID = T.EMPLOYEEID
WHERE E.CURRENT_EMPLOYEE_RATING < 3
GROUP BY 1,2

-- 58.Calculate average training days per training_program_name
SELECT 
    TRAINING_PROGRAM_NAME,
    AVG(TRAINING_DURATION_DAYS) AS AVG_TRAINING_DAYS
FROM TRAINING_AND_DEVELOPMENT_DATA
GROUP BY 1
ORDER BY AVG_TRAINING_DAYS DESC

-- 59.Identify divisions where training has highest impact on performance.
SELECT DIVISION,
AVG(CURRENT_EMPLOYEE_RATING) AS AVG_RATING
FROM EMPLOYEE_DATA
GROUP BY DIVISION

-- 60.List employees who attended training in the same month they joined.
SELECT E.EMPLOYEEID,
E.STARTDATE,TRAINING_DATE
FROM TRAINING_AND_DEVELOPMENT_DATA AS T
JOIN EMPLOYEE_DATA AS E
ON T.EMPLOYEEID = E.EMPLOYEEID
WHERE MONTH(TRAINING_DATE) =MONTH(E.STARTDATE)

-- Employee Engagement + Training

-- 61.compare traning_outcome with average engagement_score
SELECT TRAINING_OUTCOME,
	   AVG(ENGAGEMENT_SCORE) AS AVG_SCORE
       FROM employee_engagement_survey_data AS E
       JOIN training_and_development_data AS T
       ON E.EMPLOYEEID=T.EMPLOYEEID
       GROUP BY TRAINING_OUTCOME
       
-- 62.Compare Work_Life_Balance_Score with training_outcome
SELECT TRAINING_OUTCOME,
	   AVG(work_life_balance_score) AS AVG_SCORE
       FROM employee_engagement_survey_data AS E
       JOIN training_and_development_data AS T
       ON E.EMPLOYEEID=T.EMPLOYEEID
       GROUP BY TRAINING_OUTCOME

-- 63..Compare satisfaction_Score with training_outcome
SELECT TRAINING_OUTCOME,
	   AVG(satisfaction_score) AS AVG_SCORE
       FROM employee_engagement_survey_data AS E
       JOIN training_and_development_data AS T
       ON E.EMPLOYEEID=T.EMPLOYEEID
       GROUP BY TRAINING_OUTCOME
       
-- 64.Identify training programs that lead to highest engagement improvements.
SELECT TRAINING_PROGRAM_NAME,
       AVG(Engagement_Score) AS AVG_SCORE 
       FROM employee_engagement_survey_data AS E
       JOIN training_and_development_data AS T
       ON T.EMPLOYEEID=E.EMPLOYEEID
       GROUP BY TRAINING_PROGRAM_NAME
       ORDER BY AVG_SCORE DESC
       LIMIT 1
       
-- 65.List employees with poor engagement.
SELECT EMPLOYEEID,
       Engagement_Score
       FROM employee_engagement_survey_data 
       WHERE Engagement_Score <5
      
-- Employee Demographics

-- 66.Count employees by GenderCode and RaceDesc.
SELECT GENDERCODE,
       RACEDESC,
       COUNT(*) AS TOTAL_EMP
       FROM EMPLOYEE_DATA
       GROUP BY 1,2
       
-- 67.Average tenure by MaritalDesc.
SELECT MARITALDESC,
       AVG(TIMESTAMPDIFF(YEAR,STARTDATE,CURRENT_DATE())) AS AVG_TENURE
       FROM EMPLOYEE_DATA
       GROUP BY MARITALDESC
       
-- 68.Show employee distribution across States.
SELECT STATE,
       COUNT(*) AS EMP_COUNT
       FROM EMPLOYEE_DATA 
       GROUP BY 1
       
-- 69.Show employee count by payzone.
SELECT PAYZONE,
       count(*) AS EMP_COUNT
       FROM EMPLOYEE_DATA 
       GROUP BY 1
       
-- 70.Show employee counts by employeestatus.
SELECT EMPLOYEESTATUS,
       COUNT(*) AS EMP_COUNT
       FROM EMPLOYEE_DATA
       GROUP BY 1
       
-- 71 Find  age of employees
SELECT EMPLOYEEID,
       (TIMESTAMPDIFF(YEAR, DOB,CURRENT_DATE())) AS AGE
       FROM EMPLOYEE_DATA

-- 72.List applicants with more than 10 years of experience
SELECT  APPLICANT_ID, 
        FIRST_NAME, LAST_NAME, 
        YEARS_OF_EXPERIENCE
	    FROM RECRUITMENT_DATA
        WHERE YEARS_OF_EXPERIENCE > 10

-- 73.Show average Engagement_Score by DepartmentType
SELECT E.DEPARTMENTTYPE, 
AVG(S.ENGAGEMENT_SCORE) AS AVG_ENGAGEMENT_SCORE
FROM EMPLOYEE_DATA E
JOIN EMPLOYEE_ENGAGEMENT_SURVEY_DATA S
ON E.EMPLOYEEID = S.EMPLOYEEID
GROUP BY E.DEPARTMENTTYPE

-- 74.List employees whose salary expectation above average expectations
SELECT APPLICANT_ID,
       FIRST_NAME,
       LAST_NAME,
       DESIRED_SALARY
FROM RECRUITMENT_DATA
WHERE DESIRED_SALARY > (SELECT AVG(DESIRED_SALARY) FROM RECRUITMENT_DATA)

-- 75.Find employees are working in 2 or more departments 
SELECT EMPLOYEEID,
       FIRSTNAME,
       LASTNAME,
       COUNT(DISTINCT DEPARTMENTTYPE) AS NUM_DEPARTMENTS
FROM EMPLOYEE_DATA
GROUP BY EMPLOYEEID, FIRSTNAME, LASTNAME
HAVING COUNT(DISTINCT DEPARTMENTTYPE) >= 2

-- 76.Identify duplicate applicants by email or phone number.
SELECT EMAIL,
       PHONE_NUMBER,
       COUNT(*) AS DUPLICATE_COUNT
FROM RECRUITMENT_DATA
GROUP BY 1,2
HAVING COUNT(*) > 1

-- 77.Determine the lowest Engagement_Score per employee,jobtitle,departmenttype.
SELECT E.EMPLOYEEID,
       E.FIRSTNAME,
       E.LASTNAME,
       E.TITLE AS JOBTITLE,
       E.DEPARTMENTTYPE,
       MIN(S.ENGAGEMENT_SCORE) AS LOWEST_ENGAGEMENT_SCORE
FROM EMPLOYEE_DATA E
JOIN EMPLOYEE_ENGAGEMENT_SURVEY_DATA S
  ON E.EMPLOYEEID = S.EMPLOYEEID
GROUP BY 1,2,3,4,5
ORDER BY LOWEST_ENGAGEMENT_SCORE

-- 78.Show employees with more than 1 termination record (if possible).
SELECT EMPLOYEEID,
       FIRSTNAME,
       LASTNAME,
       COUNT(*) AS TERMINATION_COUNT
FROM EMPLOYEE_DATA
WHERE TERMINATIONTYPE IS NOT NULL
GROUP BY 1,2,3
HAVING COUNT(*) > 1

-- 79.Compare BusinessUnit with highest hiring vs highest exit rates.
SELECT BUSINESSUNIT,
       COUNT(STARTDATE) AS NUM_HIRED,
       COUNT(EXITDATE) AS NUM_EXITED,
       (COUNT(EXITDATE) / COUNT(STARTDATE)) * 100 AS EXIT_RATE_PERCENT
FROM EMPLOYEE_DATA
GROUP BY BUSINESSUNIT
ORDER BY NUM_HIRED DESC,
         NUM_EXITED DESC

-- 80.List employees with lower engagement rating in each department
SELECT E.DEPARTMENTTYPE,
       E.EMPLOYEEID,
       MIN(Engagement_Score) as min_score
       FROM EMPLOYEE_DATA AS E 
       JOIN employee_engagement_survey_data AS EE
       ON E.EMPLOYEEID=EE.EMPLOYEEID
       GROUP BY 1,2

-- 81.Find employees whose termination type is voluntary
SELECT EMPLOYEEID,
FIRSTNAME, LASTNAME
FROM EMPLOYEE_DATA
WHERE TERMINATIONTYPE LIKE '%Voluntary%'

-- 82. find employee name starting with 'a' 
SELECT * FROM EMPLOYEE_DATA
WHERE FIRSTNAME LIKE 'A%'

-- 83.Categorize employees by age, experience, performance, or training outcome
SELECT EMPLOYEEID, FIRSTNAME, CURRENT_EMPLOYEE_RATING,
       CASE
           WHEN CURRENT_EMPLOYEE_RATING >= 4 THEN 'HIGH'
           WHEN CURRENT_EMPLOYEE_RATING = 3 THEN 'MEDIUM'
           ELSE 'LOW'
       END AS PERFORMANCE_CATEGORY
FROM EMPLOYEE_DATA

-- 84.Show percent contribution of each BusinessUnit to total employees.
SELECT BUSINESSUNIT,
       COUNT(*) AS TOTAL_EMPLOYEES,
       (COUNT(*) / (SELECT COUNT(*) FROM EMPLOYEE_DATA) * 100) AS PERCENT_CONTRIBUTION
FROM EMPLOYEE_DATA
GROUP BY BUSINESSUNIT
ORDER BY PERCENT_CONTRIBUTION DESC

-- 85.Identify employees whose training cost is above company’s average cost
SELECT EMPLOYEEID,
       TRAINING_PROGRAM_NAME,
       TRAINING_COST
FROM TRAINING_AND_DEVELOPMENT_DATA
WHERE TRAINING_COST > (SELECT AVG(TRAINING_COST) FROM TRAINING_AND_DEVELOPMENT_DATA)
ORDER BY TRAINING_COST DESC

-- 86.employees as “Experienced” (>5 years) or “Fresher”
SELECT EMPLOYEEID,
       FIRSTNAME,
       LASTNAME,
       DATEDIFF(CURDATE(), STARTDATE)/365 AS YEARS_WITH_COMPANY,
       CASE
           WHEN DATEDIFF(CURDATE(), STARTDATE)/365 > 5 THEN 'EXPERIENCED'
           ELSE 'FRESHER'
       END AS EXPERIENCE_STATUS
FROM EMPLOYEE_DATA

-- 87.Classify employees as “High”, “Medium”, “Low” performers using Current_Employee_Rating.
SELECT EMPLOYEEID,
       FIRSTNAME,
       LASTNAME,
       CURRENT_EMPLOYEE_RATING,
       CASE
           WHEN CURRENT_EMPLOYEE_RATING >= 4 THEN 'HIGH'
           WHEN CURRENT_EMPLOYEE_RATING = 3 THEN 'MEDIUM'
           ELSE 'LOW'
       END AS PERFORMANCE_CATEGORY
FROM EMPLOYEE_DATA
ORDER BY CURRENT_EMPLOYEE_RATING DESC

-- 89.Tag training outcomes as Success/Fail based on keywords.
SELECT EMPLOYEEID,
       TRAINING_PROGRAM_NAME,
       TRAINING_OUTCOME,
       CASE
           WHEN TRAINING_OUTCOME LIKE '%Completed%' OR TRAINING_OUTCOME LIKE '%Pass%' THEN 'SUCCESS'
           ELSE 'FAIL'
       END AS TRAINING_STATUS
FROM TRAINING_AND_DEVELOPMENT_DATA
ORDER BY EMPLOYEEID;

-- 90.Categorize business units into “Large”, “Medium”, “Small” by employee count.
SELECT BUSINESSUNIT,
       COUNT(*) AS EMPLOYEE_COUNT,
       CASE
           WHEN COUNT(*) >= 100 THEN 'LARGE'
           WHEN COUNT(*) >= 50 THEN 'MEDIUM'
           ELSE 'SMALL'
       END AS BUSINESS_UNIT_SIZE
FROM EMPLOYEE_DATA
GROUP BY BUSINESSUNIT
ORDER BY EMPLOYEE_COUNT DESC

-- 91.Show all active employees
SELECT EMPLOYEEID,
       FIRSTNAME,
       LASTNAME,
       EMPLOYEESTATUS
FROM EMPLOYEE_DATA
WHERE EMPLOYEESTATUS = "ACTIVE"

-- 92.Find employees in the “Finance” department
SELECT EMPLOYEEID,
       FIRSTNAME,
       LASTNAME,
       DEPARTMENTTYPE
FROM EMPLOYEE_DATA
WHERE DEPARTMENTTYPE = "FINANCE"

-- 93.List employees with “Full-Time” employment type
SELECT EMPLOYEEID,
       FIRSTNAME,
       LASTNAME,
       EMPLOYEETYPE
FROM EMPLOYEE_DATA
WHERE EMPLOYEETYPE = "FULL-TIME"


-- 94.Show employees who joined after 2020-01-01
SELECT EMPLOYEEID,
       FIRSTNAME,
       LASTNAME,
       STARTDATE
FROM EMPLOYEE_DATA
WHERE STARTDATE > "2023-01-01"

-- 95.Show employees who resigned voluntarily vs involuntary (TerminationType).
SELECT EMPLOYEEID,
       FIRSTNAME,
       LASTNAME,
       TERMINATIONTYPE,
       CASE
           WHEN TERMINATIONTYPE LIKE '%Voluntary%' THEN 'VOLUNTARY'
           WHEN TERMINATIONTYPE LIKE '%Involuntary%' THEN 'INVOLUNTARY'
           ELSE 'OTHER'
       END AS TERMINATION_CATEGORY
FROM EMPLOYEE_DATA
ORDER BY  EMPLOYEEID

-- 96.compare applicants by education level.
SELECT EDUCATION_LEVEL,
       COUNT(*) AS APLLICANTS
       FROM RECRUITMENT_DATA 
       GROUP BY 1
       
-- 97.categorize employees as  'senior' or 'junior' based on experience
SELECT EMPLOYEEID,
       FIRSTNAME,
       LASTNAME,
       CASE 
           WHEN TIMESTAMPDIFF(YEAR, STARTDATE, CURDATE()) > 5 
           THEN 'SENIOR'
           ELSE 'JUNIOR'
       END AS EXPERIENCE_CATEGORY
FROM EMPLOYEE_DATA

-- 99. trainers per training programs 
SELECT 
    TRAINING_PROGRAM_NAME,
    COUNT(TRAINER) AS TRANIERS
FROM TRAINING_AND_DEVELOPMENT_DATA
GROUP BY TRAINING_PROGRAM_NAME

-- 100.Build a summary table 
SELECT 
    E.EMPLOYEEID,
    E.FIRSTNAME,
    E.LASTNAME,
    E.JOBFUNCTIONDESCRIPTION,
    T.TRAINING_PROGRAM_NAME,
    S.ENGAGEMENT_SCORE,
    E.CURRENT_EMPLOYEE_RATING AS POST_TRAINING_RATING
FROM EMPLOYEE_DATA E
LEFT JOIN TRAINING_AND_DEVELOPMENT_DATA T
    ON E.EMPLOYEEID = T.EMPLOYEEID
LEFT JOIN EMPLOYEE_ENGAGEMENT_SURVEY_DATA S
    ON E.EMPLOYEEID = S.EMPLOYEEID
ORDER BY E.EMPLOYEEID, T.TRAINING_PROGRAM_NAME
