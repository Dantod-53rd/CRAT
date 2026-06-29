-- ====================================================================
-- SECTION 1: KUTENGEZA DATABASE NA MEZA ZOTE (SCHEMA)
-- ====================================================================

CREATE DATABASE IF NOT EXISTS kampuni_management_system;
USE kampuni_management_system;

-- 0. Meza ya Matawi / Vituo
CREATE TABLE IF NOT EXISTS branches (
    branch_id INT AUTO_INCREMENT PRIMARY KEY,
    branch_name VARCHAR(100) NOT NULL,
    location VARCHAR(100),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 1. Kitengo cha Ulinzi
CREATE TABLE IF NOT EXISTS security_contracts (
    contract_id INT AUTO_INCREMENT PRIMARY KEY,
    branch_id INT,
    client_name VARCHAR(100) NOT NULL,
    contract_value DECIMAL(12,2) NOT NULL,
    status ENUM('Active', 'Terminated', 'Suspended') DEFAULT 'Active',
    start_date DATE,
    end_date DATE,
    FOREIGN KEY (branch_id) REFERENCES branches(branch_id)
);

CREATE TABLE IF NOT EXISTS security_incidents (
    incident_id INT AUTO_INCREMENT PRIMARY KEY,
    contract_id INT,
    incident_title VARCHAR(150) NOT NULL,
    severity_level ENUM('Low', 'Medium', 'High', 'Critical') DEFAULT 'Low',
    incident_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (contract_id) REFERENCES security_contracts(contract_id)
);

-- 2. Kitengo cha Kilimo na Ufugaji
CREATE TABLE IF NOT EXISTS agri_batches (
    batch_id INT AUTO_INCREMENT PRIMARY KEY,
    branch_id INT,
    batch_type ENUM('Crops', 'Livestock') NOT NULL, 
    item_name VARCHAR(100) NOT NULL, 
    initial_quantity INT NOT NULL, 
    current_quantity INT, 
    start_date DATE,
    FOREIGN KEY (branch_id) REFERENCES branches(branch_id)
);

CREATE TABLE IF NOT EXISTS agri_mortality_or_loss (
    loss_id INT AUTO_INCREMENT PRIMARY KEY,
    batch_id INT,
    quantity_lost INT NOT NULL, 
    reason VARCHAR(255), 
    recorded_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (batch_id) REFERENCES agri_batches(batch_id)
);

-- 3. Kitengo cha Car Wash
CREATE TABLE IF NOT EXISTS carwash_services (
    service_id INT AUTO_INCREMENT PRIMARY KEY,
    service_name VARCHAR(50) NOT NULL, 
    price DECIMAL(10,2) NOT NULL
);

CREATE TABLE IF NOT EXISTS carwash_transactions (
    tx_id INT AUTO_INCREMENT PRIMARY KEY,
    branch_id INT,
    service_id INT,
    vehicle_type ENUM('Bodaboda', 'Bajaj', 'Saloon Car', 'SUV/V8', 'Truck') NOT NULL,
    amount_paid DECIMAL(10,2) NOT NULL,
    washed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (branch_id) REFERENCES branches(branch_id),
    FOREIGN KEY (service_id) REFERENCES carwash_services(service_id)
);

-- 4. Kitengo cha Duka la Simu na Uwakala
CREATE TABLE IF NOT EXISTS products_inventory (
    product_id INT AUTO_INCREMENT PRIMARY KEY,
    branch_id INT,
    product_name VARCHAR(100) NOT NULL,
    buying_price DECIMAL(10,2) NOT NULL,
    selling_price DECIMAL(10,2) NOT NULL,
    quantity_in_stock INT DEFAULT 0,
    FOREIGN KEY (branch_id) REFERENCES branches(branch_id)
);

CREATE TABLE IF NOT EXISTS retail_sales (
    sale_id INT AUTO_INCREMENT PRIMARY KEY,
    product_id INT,
    quantity_sold INT NOT NULL,
    total_amount DECIMAL(10,2) NOT NULL,
    sold_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (product_id) REFERENCES products_inventory(product_id)
);

CREATE TABLE IF NOT EXISTS agency_transactions (
    agency_tx_id INT AUTO_INCREMENT PRIMARY KEY,
    branch_id INT,
    operator_name ENUM('M-Pesa', 'Tigo Pesa', 'Airtel Money', 'HaloPesa', 'NMB Wakala', 'CRDB Wakala') NOT NULL,
    tx_type ENUM('Deposit', 'Withdraw') NOT NULL,
    amount DECIMAL(10,2) NOT NULL,
    expected_commission DECIMAL(10,2) NOT NULL,
    float_balance_after DECIMAL(12,2) NOT NULL,
    tx_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (branch_id) REFERENCES branches(branch_id)
);

-- 5. Meza ya Malengo ya Maendeleo (KPIs)
CREATE TABLE IF NOT EXISTS kpi_targets (
    target_id INT AUTO_INCREMENT PRIMARY KEY,
    branch_id INT,
    metric_name VARCHAR(100) NOT NULL, 
    target_value DECIMAL(12,2) NOT NULL, 
    target_month_year VARCHAR(7) NOT NULL, 
    FOREIGN KEY (branch_id) REFERENCES branches(branch_id)
);


-- ====================================================================
-- SECTION 2: KUINGIZA DATA ZA MFANO (MOCK DATA)
-- ====================================================================

-- Kuingiza Matawi
INSERT INTO branches (branch_id, branch_name, location) VALUES 
(1, 'Ofisi Kuu ya Ulinzi', 'Dar es Salaam'),
(2, 'Shamba la Kuku na Kilimo', 'Kibaha'),
(3, 'Smart Car Wash & FinTech Hub', 'Mwenge');

-- Data za Ulinzi
INSERT INTO security_contracts (branch_id, client_name, contract_value, start_date) VALUES 
(1, 'NMB Bank Head Office', 5000000.00, '2026-01-01'),
(1, 'Supermarket ya Kibo', 1200000.00, '2026-02-15');

INSERT INTO security_incidents (contract_id, incident_title, severity_level) VALUES 
(1, 'Mlinzi kuchelewa zamu ya usiku', 'Medium');

-- Data za Kilimo na Ufugaji
INSERT INTO agri_batches (batch_id, branch_id, batch_type, item_name, initial_quantity, current_quantity, start_date) VALUES 
(1, 2, 'Livestock', 'Kuku wa Nyama (Batch A)', 1000, 950, '2026-05-01');

INSERT INTO agri_mortality_or_loss (batch_id, quantity_lost, reason) VALUES 
(1, 50, 'Ugonjwa wa Mdondo');

-- Data za Car Wash
INSERT INTO carwash_services (service_id, service_name, price) VALUES 
(1, 'Body Wash Only', 5000.00),
(2, 'Full Wash + Vacuum', 15000.00);

INSERT INTO carwash_transactions (branch_id, service_id, vehicle_type, amount_paid) VALUES 
(3, 1, 'Saloon Car', 5000.00),
(3, 2, 'SUV/V8', 15000.00),
(3, 2, 'Saloon Car', 15000.00);

-- Data za Duka la Simu & Uwakala
INSERT INTO products_inventory (product_id, branch_id, product_name, buying_price, selling_price, quantity_in_stock) VALUES 
(1, 3, 'Samsung Galaxy A15', 350000.00, 420000.00, 10);

INSERT INTO retail_sales (product_id, quantity_sold, total_amount) VALUES 
(1, 2, 840000.00);

INSERT INTO agency_transactions (branch_id, operator_name, tx_type, amount, expected_commission, float_balance_after) VALUES 
(3, 'M-Pesa', 'Withdraw', 500000.00, 4500.00, 2500000.00),
(3, 'CRDB Wakala', 'Deposit', 1000000.00, 3000.00, 1500000.00);

-- Kuingiza Malengo ya Mwezi (KPI Targets - June 2026)
INSERT INTO kpi_targets (branch_id, metric_name, target_value, target_month_year) VALUES 
(2, 'Kuku_Mortality_Limit', 5.00, '2026-06'), -- Lengo: Vifo visizidi 5%
(3, 'Carwash_Revenue_Target', 50000.00, '2026-06'), -- Lengo: Kukusanya 50,000/=
(3, 'Agency_Commission_Target', 10000.00, '2026-06'); -- Lengo: Kamisheni ifike 10,000/=


-- ====================================================================
-- SECTION 3: VIPIMO VYA MAENDELEO (AUTOMATED VIEWS FOR KPIS)
-- ====================================================================

-- KIPIMO A: Maendeleo ya Car Wash dhidi ya Lengo uliyojiwekea
CREATE OR REPLACE VIEW view_progress_carwash AS
SELECT 
    b.branch_name,
    SUM(t.amount_paid) AS Mapato_Yaliyopatikana,
    k.target_value AS Lengo_la_Mwezi,
    ROUND((SUM(t.amount_paid) / k.target_value) * 100, 2) AS Asilimia_ya_Maendeleo
FROM carwash_transactions t
JOIN branches b ON t.branch_id = b.branch_id
JOIN kpi_targets k ON t.branch_id = k.branch_id AND k.metric_name = 'Carwash_Revenue_Target'
WHERE DATE_FORMAT(t.washed_at, '%Y-%m') = '2026-06'
GROUP BY b.branch_id, k.target_value;

-- KIPIMO B: Afya ya Ufugaji (Kiwango cha vifo vs Lengo)
CREATE OR REPLACE VIEW view_progress_livestock AS
SELECT 
    b.item_name,
    b.initial_quantity AS Idadi_Mwanzo,
    IFNULL(SUM(l.quantity_lost), 0) AS Jumla_ya_Vifo,
    ROUND((IFNULL(SUM(l.quantity_lost), 0) / b.initial_quantity) * 100, 2) AS Asilimia_ya_Vifo_Uhalisia,
    k.target_value AS Kiwango_Kikomo_Target
FROM agri_batches b
LEFT JOIN agri_mortality_or_loss l ON b.batch_id = l.batch_id
JOIN kpi_targets k ON b.branch_id = k.branch_id AND k.metric_name = 'Kuku_Mortality_Limit'
WHERE b.batch_type = 'Livestock'
GROUP BY b.batch_id, k.target_value;

-- KIPIMO C: Maendeleo ya Kamisheni za Uwakala
CREATE OR REPLACE VIEW view_progress_agency AS
SELECT 
    b.branch_name,
    SUM(a.expected_commission) AS Kamisheni_Iliyopatikana,
    k.target_value AS Lengo_la_Kamisheni,
    ROUND((SUM(a.expected_commission) / k.target_value) * 100, 2) AS Asilimia_ya_Maendeleo
FROM agency_transactions a
JOIN branches b ON a.branch_id = b.branch_id
JOIN kpi_targets k ON a.branch_id = k.branch_id AND k.metric_name = 'Agency_Commission_Target'
WHERE DATE_FORMAT(a.tx_date, '%Y-%m') = '2026-06'
GROUP BY b.branch_id, k.target_value;

-- KIPIMO D: Faida Safi ya Duka la Simu
CREATE OR REPLACE VIEW view_progress_retail_profit AS
SELECT 
    p.product_name,
    SUM(s.quantity_sold) AS Idadi_Zilizouzwa,
    SUM(s.total_amount) AS Jumla_ya_Mauzo,
    SUM(s.total_amount - (p.buying_price * s.quantity_sold)) AS Faida_Safi
FROM retail_sales s
JOIN products_inventory p ON s.product_id = p.product_id,
GROUP BY p.product_id;


