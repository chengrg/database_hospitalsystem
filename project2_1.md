SQL Schema:
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

CREATE TABLE person(
id INT,
name VARCHAR(255) NOT NULL,
email VARCHAR(255) NOT NULL,
tel BIGINT NOT NULL,
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
