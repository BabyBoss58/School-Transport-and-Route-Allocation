# School-Transport-and-Route-Allocation
Using SQL to solve and reduce repeated route and bus information and allowing management to review route usage and fee status with PostgreSql.

[PROJECT.docx](https://github.com/user-attachments/files/34671304/PROJECT.docx)

[School_Transport_Route_Allocation (1).pptx](https://github.com/user-attachments/files/34671331/School_Transport_Route_Allocation.1.pptx)

[group 8.sql](https://github.com/user-attachments/files/34671245/group.8.sql)
-- Database: SCHOOL TRANSPORT AND ROUTE ALLOCATION SYSTEM

-- DROP DATABASE IF EXISTS "SCHOOL TRANSPORT AND ROUTE ALLOCATION SYSTEM";

-- Database: SCHOOL TRANSPORT AND ROUTE ALLOCATION SYSTEM

-- DROP DATABASE IF EXISTS "SCHOOL TRANSPORT AND ROUTE ALLOCATION SYSTEM";

-- pupils table
CREATE TABLE Pupils(
       pupil_no VARCHAR(10) PRIMARY KEY,
	   pupil_name VARCHAR (100) NOT NULL,
	   class_name VARCHAR (20) NOT NULL
);

-- buses
CREATE TABLE Buses (
        bus_reg VARCHAR (20) PRIMARY KEY
);

-- driver info 
CREATE TABLE Drivers (
		drivers_id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
		driver_name VARCHAR(100) NOT NULL UNIQUE
);

-- routes 
CREATE TABLE Routes (
  	  route_code VARCHAR(10) PRIMARY KEY,
   	  route_name VARCHAR(100) NOT NULL,
      bus_reg VARCHAR(20) NOT NULL,
      drivers_id INTEGER NOT NULL,
     FOREIGN KEY (bus_reg)
        REFERENCES buses(bus_reg),
     FOREIGN KEY (drivers_id)
        REFERENCES drivers(drivers_id)
);

-- transport
CREATE TABLE Transport_records (
    pupil_no VARCHAR(10) NOT NULL,
    fee_month DATE NOT NULL,
    route_code VARCHAR(10) NOT NULL,
    pickup_point VARCHAR(100) NOT NULL,
    payment_status VARCHAR(10) NOT NULL,

    PRIMARY KEY (pupil_no, fee_month),
    FOREIGN KEY (pupil_no)
        REFERENCES Pupils(pupil_no),
    FOREIGN KEY (route_code)
        REFERENCES routes(route_code),
    CHECK (EXTRACT(DAY FROM fee_month) = 1),
    CHECK (payment_status IN ('Paid', 'Unpaid'))
);


--- transport_import table was created to enable the csv file to be extracted into it 
CREATE TABLE transport_import (
    pupil_no VARCHAR(10),
    pupil_name VARCHAR(100),
    class_name VARCHAR(20),
    route_code VARCHAR(10),
    route_name VARCHAR(100),
    bus_reg VARCHAR(20),
    driver_name VARCHAR(100),
    pickup_point VARCHAR(100),
    fee_month DATE,

    payment_status VARCHAR(10)
);


select * 
from transport_import;

select count(*)
from transport_import;


--inserting into each table for all five tables that where identified to hold its records 
-- after extracting the csv file 
--- distinct was used to uniquely extract each record into the right field without duplicates


-- pupils table first
INSERT INTO Pupils (pupil_no, Pupil_name,class_name)
SELECT DISTINCT pupil_no, pupil_name, class_name
from transport_import;

select *
from Pupils;

-- insertion into pupils table 
INSERT INTO Buses (bus_reg)
SELECT DISTINCT bus_reg
FROM transport_import;

select *
from Buses;

-- insertion into drivers table 
INSERT INTO Drivers (driver_name)
SELECT DISTINCT driver_name
FROM transport_import;

select *
from drivers;

-- insertion into routes table includinf the join query
INSERT INTO Routes (route_code, route_name, bus_reg, drivers_id)
SELECT DISTINCT t.route_code, t.route_name,
				t.bus_reg, d.drivers_id
FROM transport_import t 
JOIN drivers d
	ON t.driver_name = d.driver_name;

select *
from routes;

--- insertion into transport records table
INSERT INTO Transport_records (pupil_no, fee_month,
	route_code, pickup_point, 
	payment_status)
SELECT DISTINCT pupil_no, fee_month, route_code, 
				pickup_point, payment_status
From transport_import; 

select *
from Transport_records;


---alter table
ALTER TABLE Transport_records
ADD COLUMN Parent_contact Text;


--- safe delete operation
-- inserting a record into the pupils table 
INSERT INTO pupils(pupil_no, pupil_name, class_name)
VALUES
('M001', 'Hesse Williams', 'G7');

-- insert another record into the transport records table 
INSERT INTO  transport_records (pupil_no, fee_month, route_code, pickup_point, payment_status)
VALUES
('M001', DATE '2026-09-01', 'R01', 'Atomic Juction', 'Unpaid');
									
--- deleting the records creted bith in pupils table and transport_records table
DELETE FROM transport_records
WHERE pupil_no = 'M001';

DELETE FROM pupils
WHERE pupil_no = 'M001';



-- Question 1
-- pupils assigned to each route code
select route_code, count(pupil_no) pupil_count
from transport_records
Group by route_code
order by route_code;
 


--- Question 2
--- all records with an Unpaid payment status
select *
from transport_records
where payment_status = 'Unpaid';


select p.pupil_no, p.pupil_name,
	t.route_code, t.fee_month,
	t.pickup_point, t.payment_status
from transport_records t 
join pupils p
	on p.pupil_no = t.pupil_no
where payment_status = 'Unpaid';


select p.pupil_no, p.pupil_name,
	r.route_name, r.route_code, 
	t.fee_month, t.pickup_point, 
	t.payment_status
from transport_records t 
join pupils p
	on p.pupil_no = t.pupil_no
join routes r
	on r.route_code = t.route_code
where payment_status = 'Unpaid';



-- question 3
--- records associated with each class name
select p.class_name, count(*) total_records
from transport_records t
join pupils p
	on p.pupil_no = t.pupil_no
Group by class_name
order by class_name;



-- question 4
-- fee records in October 2026
select *
from transport_records
where fee_month >= '2026-10-01'
	and fee_month < '2026-11-01';


select p.pupil_name, p.pupil_no, 
	t.fee_month, t.pickup_point, 
	t.payment_status
from transport_records t
join pupils p
	on p.pupil_no = t.pupil_no
where fee_month >= '2026-10-01'
	and fee_month < '2026-11-01'
order by pupil_name asc;



-- question 5
-- pickup points containing 'Junction'
select  pupil_no, 
	fee_month,
	pickup_point,
	payment_status
from transport_records
where pickup_point Ilike '%Junction%';


-- question 6
-- the records in alphabetical order of route name and pupil name
select p.pupil_name, p.class_name,
	r.route_name, t.pickup_point
from pupils p
join transport_records t
	on p.pupil_no = t.pupil_no
join routes r 
	on r.route_code = t.route_code
order by route_name, pupil_name;







	
