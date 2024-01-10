CREATE DATABASE «appliances_store»
USE «appliancesstore»;
CREATE TABLE «producer» (
  «id» INT CONSTRAINT «pk_producer_id» PRIMARY KEY,
  «name» VARCHAR2 NOT NULL,
  «address» VARCHAR2,
  «is_active» NUMBER(1) NOT NULL DEFAULT 1
);

CREATE TABLE «appliance» (
  «id» INT CONSTRAINT «pk_appliance_id» PRIMARY KEY,
  «name» VARCHAR2 NOT NULL,
  «description» VARCHAR2,
  «price» NUMBER(10,2) NOT NULL,
  «is_active» NUMBER(1) NOT NULL DEFAULT 1,
  «producer_id» INT NOT NULL,
  CONSTRAINT «fk_appliance_producer_id» FOREIGN KEY (producer_id) REFERENCES «producer» («id»)
);

CREATE INDEX «k_appliance_price» ON «appliance» («price»);

CREATE TABLE «supplier» (
  «id» INT CONSTRAINT «pk_supplier_id» PRIMARY KEY,
  «name» VARCHAR2 NOT NULL,
  «address» VARCHAR2,
  «is_active» NUMBER(1) NOT NULL DEFAULT 1
);

CREATE TABLE «producer» (
  «id» INT CONSTRAINT «pk_producer_id» PRIMARY KEY,
  «name» VARCHAR2 NOT NULL,
  «address» TEXT,
  «is_active» NUMBER(1) NOT NULL DEFAULT 1
);

CREATE TABLE «producer» (
  «id» INT CONSTRAINT «pk_producer_id» PRIMARY KEY,
  «name» VARCHAR2 NOT NULL,
  «address» TEXT,
  «is_active» NUMBER(1) NOT NULL DEFAULT 1
);

INSERT INTO «producer» («name», «address») VALUES
('Producer 1', 'Some country, Some city, Some street, 11'),
('Producer 2', 'Some country, Some city, Some street, 22'),
('Producer 3', 'Some country, Some city, Some street, 33'),
('Producer 4', 'Some country, Some city, Some street, 44'),
('Producer 5', 'Some country, Some city, Some street, 55');

INSERT INTO «appliance» («name», «description», «price», «producer_id») VALUES
('Device 1', 'Some description to device 1', 500.0, 1),
('Device 2', 'Some description to device 2', 1500.0, 1),
('Device 3', 'Some description to device 3', 6500.0, 1),
('Device 4', 'Some description to device 4', 9500.0, 1),
('Device 5', 'Some description to device 5', 500.0, 2),
('Device 6', 'Some description to device 6', 7500.0, 2),
('Device 7', 'Some description to device 7', 2500.0, 3),
('Device 8', 'Some description to device 8', 4500.0, 3),
('Device 9', 'Some description to device 9', 1000.0, 4),
('Device 10', 'Some description to device 10', 1500.0, 4),
('Device 11', 'Some description to device 11', 8500.0, 4);

INSERT INTO «supplier» («name», «address») VALUES
('Supplier 1', 'Some country, Some city, Some street, 1'),
('Supplier 2', 'Some country, Some city, Some street, 2'),
('Supplier 3', 'Some country, Some city, Some street, 3'),
('Supplier 4', 'Some country, Some city, Some street, 4'),
('Supplier 5', 'Some country, Some city, Some street, 5');

INSERT INTO «supplier2appliance» («supplier_id», «appliance_id») VALUES
(1, 1),
(2, 1),
(3, 1),
(1, 2),
(2, 2),
(5, 2),
(2, 3),
(3, 3),
(4, 3),
(5, 3),
(1, 4),
(2, 4),
(5, 4),
(2, 5),
(3, 5),
(4, 5),
(5, 5);

SELECT a.«id» AS appliance_id,
a.«name» AS appliance_name,
a.«proce» AS price,
p.«name» AS producer_name
FROM «appliance» a
INNER JOIN «producer» p ON a.«producer_id» = p.«id»
WHERE a.«is_active» = 1
AND a.«price» > 5000.0

SELECT s.«id» AS supplier_id,
s.«name» as supplier_name
FROM «appliance» a
INNER JOIN «supplier2appliance» s2p ON s2p.«appliance_id» = a.«id»
INNER JOIN «supplier» s ON s2p.«supplier_id» = s.«id»
WHERE a.«is_active» = 1
AND a.«price» > 5000.0 AND a.«price» < 20000.0
GROUP BY s.«id»,
s.«name»


