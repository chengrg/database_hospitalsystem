Update:  
1. Composite type  
We want to change the data type of the ‘tel’ attribute of ‘person’ entity, from a single integer to a composite of area_code and subscriber_numer. This is because the telephone number is by nature a composition of these two fields, and by separating the area code and subscriber number, we made possible the assertion of validity of telephone numbers, and the area code also contains extra (geologic) information of the line holder, which is made explicit by the composite data type.

CREATE TYPE tel_num AS (area_code integer,
subscriber_number integer
);
ALTER TABLE person DROP COLUMN tel;
ALTER TABLE person ADD COLUMN tel tel_num;

2. Array type  
We added a new attribute ‘publication’ to ‘person’, enabling the database to keep records of each hospital staff’s publications, which can be useful in many ways. Since a person can have multiple publications, making it a string array is reasonable. We also enable the database to keep track of, and automatically update the total number of publications of each person by adding a ‘pub_num’ attribute to person, which is updated automatically by a trigger introduced below.

ALTER TABLE person ADD COLUMN publication varchar(127)[];
ALTER TABLE person ADD COLUMN pub_num integer;

3. Trigger  
The trigger is called “check_update” and the function is called “check_pub_num”. It is intended to automatically update the number of publications from the person when there is an update on this person’s publication record, instead of updating the number of publications manually by users. For example, if a person of two publications publishes a new journal, when the journal is added to the person’s publication record, the number of publications will change from two to three by the trigger.

CREATE FUNCTION check_pub_num() RETURNS TRIGGER AS $$
BEGIN
IF (array_length(NEW.publication,1) IS NULL) THEN
NEW.pub_num =0;
ELSE
NEW.pub_num = array_length(NEW.publication,1);
END IF;
return NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER check_update 
    BEFORE INSERT OR UPDATE OF publication ON person
    FOR EACH ROW
    EXECUTE FUNCTION check_pub_num();

4. Loading new columns to person:  
Since a large portion of the database directly or indirectly depends on the ‘person’ entity, we don’t want to simply drop ‘person’, make modifications and re-add. Instead, we create a temporary table, load the new data into this table, then add columns to the original ‘person’ table, and then insert the partial data into the original table to its correct location by referencing the temp table. Below is the loading plan. Thanks to the trigger, pub_num is automatically calculated for each person after the data loading.

CREATE TABLE temp(
id INT,
name VARCHAR(255) NOT NULL,
email VARCHAR(255) NOT NULL,
tel tel_num,
publication varchar(127)[],
pub_num integer,
PRIMARY KEY(id)
);

(loading new table to temp)

UPDATE person AS p SET(tel, publication, pub_num)=(t.tel, t.publication, t.pub_num)
FROM temp t WHERE t.id=p.id;

DROP TABLE temp;
  
       
     
     
    
         
     
##Full SQL Schema:  
CREATE TABLE hospital(
name VARCHAR(255),
address VARCHAR(255) NOT NULL,
PRIMARY KEY(name)
)

CREATE TABLE location(
building VARCHAR(255),
room VARCHAR(127),
hospital_name VARCHAR(255),
PRIMARY KEY(building, room, hospital_name),
FOREIGN KEY(hospital_name) REFERENCES hospital(name)
)

CREATE TABLE department(
dept_name VARCHAR(255),
budget REAL,
hospital_name VARCHAR(255),
PRIMARY KEY(dept_name, hospital_name),
FOREIGN KEY(hospital_name) REFERENCES hospital(name)
)

CREATE TYPE tel_num AS (area_code integer,
subscriber_number integer
)

CREATE TABLE person(
id INT,
name VARCHAR(255) NOT NULL,
email VARCHAR(255) NOT NULL,
tel tel_num NOT NULL,
publication varchar(127)[],
pub_num integer;
PRIMARY KEY(id)
)

CREATE TABLE surgery(
surgery_id INT,
surgery_title VARCHAR(255) NOT NULL,
difficulty VARCHAR(31),
building VARCHAR(255) NOT NULL,
room VARCHAR(127) NOT NULL,
hospital_name VARCHAR(255) NOT NULL,
patient_id INT NOT NULL,
PRIMARY KEY(surgery_id),
FOREIGN KEY(building, room,hospital_name) REFERENCES location(building, room, hospital_name),
FOREIGN KEY(patient_id) REFERENCES patient(id)
)

CREATE TABLE consultation(
consultation_id INT,
category VARCHAR(127) NOT NULL,
building VARCHAR(255) NOT NULL, 
room VARCHAR(127) NOT NULL,
hospital_name VARCHAR(255) NOT NULL,
doctor_id INT NOT NULL,
patient_id INT NOT NULL,
PRIMARY KEY(consultation_id),
FOREIGN KEY(building, room,hospital_name) REFERENCES location(building, room,hospital_name),
FOREIGN KEY(doctor_id) REFERENCES doctor(doctor_id),
FOREIGN KEY(patient_id) REFERENCES patient(id)
)

CREATE TABLE loc_available(
building VARCHAR(255),
room VARCHAR(127),
hospital_name VARCHAR(255),
date VARCHAR(31),
slot INT,
FOREIGN KEY(building, room,hospital_name) REFERENCES location(building, room,hospital_name),
FOREIGN KEY(date, slot) REFERENCES time_slot(date, slot)
)

CREATE TABLE doctor(
doctor_id INT,
name VARCHAR(255) NOT NULL,
level VARCHAR(31),
dept_name VARCHAR(255) NOT NULL,
hospital_name VARCHAR(255) NOT NULL,
person_id INT NOT NULL,
PRIMARY KEY(doctor_id),
FOREIGN KEY(dept_name, hospital_name) REFERENCES department(dept_name, hospital_name),
FOREIGN KEY(person_id) REFERENCES person(id)
)

CREATE TABLE nurse(
nurse_id INT,
name VARCHAR(255) NOT NULL,
dept_name VARCHAR(255) NOT NULL,
hospital_name VARCHAR(255) NOT NULL,
person_id INT NOT NULL,
PRIMARY KEY(nurse_id),
FOREIGN KEY(dept_name, hospital_name) REFERENCES department(dept_name, hospital_name),
FOREIGN KEY(person_id) REFERENCES person(id)
)

CREATE TABLE patient(
id INT,
name VARCHAR(255) NOT NULL,
person_id INT NOT NULL,
PRIMARY KEY(id),
FOREIGN KEY(person_id) REFERENCES person(id)
)

CREATE TABLE pat_available(
date VARCHAR(31),
slot INT,
patient_id INT,
FOREIGN KEY(date,slot) REFERENCES time_slot,
FOREIGN KEY(patient_id) REFERENCES patient(id)
)

CREATE TABLE doctor_available(
doctor_id INT NOT NULL,
date VARCHAR(31) NOT NULL,
slot INT NOT NULL,
FOREIGN KEY(doctor_id) REFERENCES doctor(doctor_id),
FOREIGN KEY(date, slot) REFERENCES time_slot(date, slot)
)

CREATE TABLE surg_time(
surgery_id INT NOT NULL,
date VARCHAR(31) NOT NULL,
slot INT NOT NULL,
FOREIGN KEY(surgery_id) REFERENCES surgery(surgery_id),
FOREIGN KEY(date, slot) REFERENCES time_slot(date, slot)
)

CREATE TABLE cons_time(
consultation_id INT NOT NULL,
date VARCHAR(31) NOT NULL,
slot INT NOT NULL,
FOREIGN KEY(consultation_id) REFERENCES consultation(consultation_id),
FOREIGN KEY(date, slot) REFERENCES time_slot(date, slot)
)

CREATE TABLE operates(
surgery_id INT,
doctor_id INT NOT NULL,
nurse_id INT NOT NULL,
PRIMARY KEY(surgery_id),
FOREIGN KEY(surgery_id) REFERENCES surgery(surgery_id),
FOREIGN KEY(doctor_id) REFERENCES doctor(doctor_id),
FOREIGN KEY(nurse_id) REFERENCES nurse(nurse_id)
)

CREATE TABLE time_slot(
date VARCHAR(31),
slot INT CHECK (slot >= 0 AND slot < 24),
PRIMARY KEY(date, slot)
)

CREATE FUNCTION check_pub_num() RETURNS TRIGGER AS $$
BEGIN
IF (array_length(NEW.publication,1) IS NULL) THEN
NEW.pub_num =0;
ELSE
NEW.pub_num = array_length(NEW.publication,1);
END IF;
return NEW;
END;
$$ LANGUAGE plpgsql

CREATE TRIGGER check_update 
    BEFORE INSERT OR UPDATE OF publication ON person
    FOR EACH ROW
    EXECUTE FUNCTION check_pub_num()

CREATE TABLE temp(
id INT,
name VARCHAR(255) NOT NULL,
email VARCHAR(255) NOT NULL,
tel tel_num,
publication varchar(127)[],
pub_num integer,
PRIMARY KEY(id)
)
