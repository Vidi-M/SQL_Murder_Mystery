# SQL_Murder_Mystery

## Introduction
There's been a Murder in SQL City! The [SQL Murder Mystery](https://mystery.knightlab.com/) is designed to be both a self-directed lesson to learn SQL concepts and commands and a fun game for experienced SQL users to solve an intriguing crime.

## Database Structure
The SQL Murder Mystery is built using SQLite. Use this SQL command to find the tables in the Murder Mystery database.

### Table Names
```
SELECT name 
  FROM sqlite_master
 where type = 'table'
```
| **name**             |
|--------------------------|
| crime_scene_report       |
| drivers_license          |
| facebook_event_checkin   |
| interview                |
| get_fit_now_member       |
| get_fit_now_check_in     |
| solution                 |
| income                   |
| person                   |

### Schema Diagram

![image](https://github.com/user-attachments/assets/4b17de24-74ba-4d56-ad70-31b3fe515ed0)

## Starting Information
A crime has taken place and the detective needs your help. The detective gave you the crime scene report, but you somehow lost it. You vaguely remember that the crime was a **​murder​** that occurred sometime on **​Jan.15, 2018**​ and that it took place in **​SQL City​**. Start by retrieving the corresponding crime scene report from the police department’s database.

## My Queries

### Initial Crime Scene report
```
SELECT *
  FROM crime_scene_report
WHERE date=20180115
  AND type="murder"
  AND city="SQL City";
```
| Date       | Type   | Description                                                                                                      | City     |
|------------|--------|------------------------------------------------------------------------------------------------------------------|----------|
| 20180115   | Murder | Security footage shows that there were 2 witnesses. The first witness lives at the last house on "Northwestern Dr". The second witness, named Annabel, lives somewhere on "Franklin Ave". | SQL City |

---

### First Witness
```
SELECT *
	FROM person
		WHERE address_street_name="Northwestern Dr"
		ORDER BY address_number DESC 
		LIMIT 1;
```
| id     | name           | license_id | address_number | address_street_name | ssn        |
|--------|----------------|------------|----------------|---------------------|------------|
| 14887  | Morty Schapiro | 118009     | 4919           | Northwestern Dr     | 111564949  |



### Second Witness
```
SELECT *
	FROM person
		WHERE address_street_name="Franklin Ave"
		AND name LIKE "Annabel %"
```
| id     | name           | license_id | address_number | address_street_name | ssn        |
|--------|----------------|------------|----------------|---------------------|------------|
| 16371  | Annabel Miller | 490173     | 103            | Franklin Ave        | 318771143  |

---

Using a subquery ensures that the query dynamically retrieves the correct ID, keeping the database operations accurate and up-to-date without manual intervention. It is a good practice despite this datebase not being regularly updated.

### First Witness Interview
```
SELECT * 
	FROM interview 
		WHERE person_id IN 
			(SELECT id 
			 FROM person 
			 WHERE address_street_name = 'Northwestern Dr' 
			 ORDER BY address_number DESC LIMIT 1)
```
| person_id | transcript                                                                                                                               |
|-----------|-------------------------------------------------------------------------------------------------------------------------------------------|
| 14887     | I heard a gunshot and then saw a man run out. He had a "Get Fit Now Gym" bag. The membership number on the bag started with "48Z". Only gold members have those bags. The man got into a car with a plate that included "H42W". |

### Second Witness Interview
```
SELECT * 
	FROM interview 
		WHERE person_id IN 
			(SELECT id
				FROM person
				WHERE address_street_name="Franklin Ave"
				AND name LIKE "Annabel %")
```
| person_id | transcript                                                                                                                               |
|-----------|-------------------------------------------------------------------------------------------------------------------------------------------|
| 16371     | I saw the murder happen, and I recognized the killer from my gym when I was working out last week on January the 9th. |

---

### Finding Suspects based on Membership Info
```
SELECT get_fit_now_member.*, get_fit_now_check_in.check_in_date
	FROM get_fit_now_member
	JOIN get_fit_now_check_in 
		ON get_fit_now_member.id = get_fit_now_check_in.membership_id
	WHERE id LIKE "48Z%"
	AND membership_status = "gold"
	AND check_in_date = "20180109"
```
| id     | person_id | name          | membership_start_date | membership_status | check_in_date |
|--------|-----------|---------------|-----------------------|-------------------|---------------|
| 48Z7A  | 28819     | Joe Germuska  | 20160305              | gold              | 20180109      |
| 48Z55  | 67318     | Jeremy Bowers | 20160101              | gold              | 20180109      |

### Matching Suspects to Killer's Car
```
SELECT person.*
	FROM person
	JOIN drivers_license 
		ON person.license_id = drivers_license.id
		WHERE plate_number LIKE "%H42W%"
		AND person.id IN
		 (SELECT person_id
			FROM get_fit_now_member
			JOIN get_fit_now_check_in 
				ON get_fit_now_member.id = get_fit_now_check_in.membership_id
			WHERE id LIKE "48Z%"
			AND membership_status = "gold"
			AND check_in_date = "20180109")
```
| id     | name          | license_id | address_number | address_street_name    | ssn        |
|--------|---------------|------------|----------------|------------------------|------------|
| 67318  | Jeremy Bowers | 423327     | 530            | Washington Pl, Apt 3A  | 871539279  |

---
 ## Killer Found: Jeremy Bowers 🎉

---

# Extention
"Congrats, you found the murderer! But wait, there's more... If you think you're up for a challenge, try querying the interview transcript of the murderer to find the real villain behind this crime. If you feel especially confident in your SQL skills, try to complete this final step with no more than 2 queries. Use this same INSERT statement with your new suspect to check your answer."

## My Queries

### Murderer's Interview
```
Select * 
	FROM interview
	WHERE person_id IN
		(SELECT id
		 	FROM person
		 	WHERE name = "Jeremy Bowers")
```
| person_id | transcript                                                                                                                                             |
|-----------|---------------------------------------------------------------------------------------------------------------------------------------------------------|
| 67318     | I was hired by a woman with a lot of money. I don't know her name but I know she's around 5'5" (65") or 5'7" (67"). She has red hair and she drives a Tesla Model S. I know that she attended the SQL Symphony Concert 3 times in December 2017. |


### Find Woman
```
Select * 
	FROM person
	WHERE license_id IN
		(SELECT id
		 	FROM drivers_license
		 	WHERE gender = "female"
		 	AND height BETWEEN 65 AND 67
			AND hair_color = "red"
			AND car_make = "Tesla"
			AND car_model = "Model S")
	AND id IN
		(SELECT person_id
		 	FROM facebook_event_checkin
		 	WHERE event_name = "SQL Symphony Concert"
			AND date LIKE "201712%")
```
| id     | name            | license_id | address_number | address_street_name | ssn        |
|--------|-----------------|------------|----------------|---------------------|------------|
| 99716  | Miranda Priestly | 202298     | 1883           | Golden Ave          | 987756388  |

## TRUE VILLIAN: Miranda Priestly 😈👠



