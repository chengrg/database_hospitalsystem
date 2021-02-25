
SQL Schema:

location(
building VARCHAR(255),
room VARCHAR(127),
PRIMARY KEY(building, room)
)

surgery(
surgery_id INT,
surgery_title VARCHAR(255) NOT NULL,
difficulty VARCHAR(31) CHECK(difficulty IN (‘easy’, ‘medium’, ‘hard’)),
building VARCHAR(255),
room VARCHAR(127),
PRIMARY KEY(surgery_id),
FOREIGN KEY(building, room) REFERENCES location(building, room)
)

consultation(
consultation_id INT,
category VARCHAR(127) NOT NULL,
building VARCHAR(255), 
room VARCHAR(127),
doctor_id INT,
PRIMARY KEY(consultation_id),
FOREIGN KEY(building, room) REFERENCES location(building, room)
FOREIGN KEY(doctor_id) REFERENCES doctor(doctor_id)
)

time_slot(
date VARCHAR(31),
slot INT CHECK (slot >= 0 AND slot < 24),
PRIMARY KEY(date, slot)
)

doctor(
doctor_id INT,
name VARCHAR(255) NOT NULL,
level VARCHAR(31),
PRIMARY KEY(doctor_id)
)

nurse(
nurse_id INT,
name VARCHAR(255) NOT NULL,
PRIMARY KEY(nurse_id)
)

department(
dept_name VARCHAR(255),
budget REAL,
PRIMARY KEY(dept_name)
)

loc_available(
building VARCHAR(255),
room VARCHAR(127),
date VARCHAR(31),
slot INT,
FOREIGN KEY(building, room) REFERENCES location(building, room),
FOREIGN KEY(date, slot) REFERENCES time_slot(date, slot)
)

doctor_available(
doctor_id INT,
date VARCHAR(31),
slot INT,
FOREIGN KEY(doctor_id) REFERENCES doctor(doctor_id),
FOREIGN KEY(date, slot) REFERENCES time_slot(date, slot)
)

surg_time(
surgery_id INT,
date VARCHAR(31),
slot INT,
FOREIGN KEY(surgery_id) REFERENCES surgery(surgery_id),
FOREIGN KEY(date, slot) REFERENCES time_slot(date, slot)
)

cons_time(
consultation_id INT,
date VARCHAR(31),
slot INT,
FOREIGN KEY(consultation_id) REFERENCES consultation(consultation_id),
FOREIGN KEY(date, slot) REFERENCES time_slot(date, slot)
)

operates(
surgery_id INT,
doc_id INT,
nurse_id INT,
PRIMARY KEY(surgery_id),
FOREIGN KEY(surgery_id) REFERENCES surgery(surgery_id),
FOREIGN KEY(doc_id) REFERENCES doctor(doctor_id),
FOREIGN KEY(nurse_id) REFERENCES nurse(nurse_id)
)

