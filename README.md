# Deploying a Node.js Application with MySQL using systemd

## Part 1: Database Setup

### 0. Check if MySQL is Installed
```sh
mysql --version
```
If MySQL is not installed, proceed with the installation steps below.

### 1. Install MySQL
```sh
sudo apt update
sudo apt install mysql-server -y
```
### 2. Start MySQL 
```sh
sudo systemctl start mysql  
```

### 3. Secure MySQL Installation
Run the MySQL secure installation script:
```sh
sudo mysql_secure_installation
```
- Choose **Yes (Y)** for password validation policy setup.
- Remove **anonymous users**.
- Disallow **remote root login**.
- Remove **test database**.
- Reload privilege tables.

### 4. Set Root Password & Authentication Method
```sh
sudo mysql -u root -p
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'YourSecurePassword'; #Poridhi@2025
FLUSH PRIVILEGES;
```

---

### 5. Create a New Database
```sql
CREATE DATABASE practice_app;
USE practice_app;
```

### 6. Create a Users Table
```sql
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) NOT NULL
);
```

### 7. Insert Sample Data
```sql
INSERT INTO users (name, email) VALUES ('Sami Khan', 'sami34@gmail.com');
INSERT INTO users (name, email) VALUES ('Piyal Rahman', 'piyalbd@gmail.com');
INSERT INTO users (name, email) VALUES ('Kabir Ahmed', 'kabir_it@gmail.com');
INSERT INTO users (name, email) VALUES ('Fahim Ahmed', 'fahim33@gmail.com');
```
### 8. Enable MySQL to Start on Boot
```sh
sudo systemctl enable mysql
```
---


## Part 2: Application Setup

### 1. Create a Node.js Project Directory and Initialize It with npm
```sh
sudo mkdir -p /opt/app
cd /opt/app
curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
sudo apt-get install -y nodejs
sudo npm init -y
```

### 2. Install Necessary Dependencies (Express and MySQL)
```sh
sudo npm install express mysql2
```

### 3. Create a Node.js Application That Connects to MySQL
```sh
sudo vim server.js
```

### 4. Implement the Required API Endpoints (Health and Users)
```js
const express = require('express');
const mysql = require('mysql2');
const app = express();
const port = 3000;

const db = mysql.createConnection({
    host: 'localhost',
    user: 'root',
    password: 'Poridhi@2025',
    database: 'practice_app'
});

app.get('/health', (req, res) => {
    db.connect((err) => {
        if (err) {
            console.error('MySQL Connection Failed: ', err);
            return;
        }
        res.status(200).send('Database Connection Successfully Established.');
    });
});

app.get('/users', (req, res) => {
    db.query('SELECT * FROM users', (err, results) => {
        if (err) {
            res.status(500).send('Database query error');
            return;
        }
        res.json(results);
    });
});

app.listen(port, () => {
    console.log(`Server is running at http://localhost:${port}`);
});
```

---

## Part 3: systemd Configuration

### 1. Create a Dedicated System User for Running the Application
```sh
sudo useradd -r -s /usr/sbin/nologin nodejs
```

### 2. Place Your Application in an Appropriate Directory with Proper Permissions
```sh
sudo chown nodejs:nodejs /opt/app
```

### 3. Create a Systemd Service File for Your Application

#### Create `check-mysql.sh` File
```sh
sudo vim /root/code/script/check-mysql.sh
```
Add the following content:
```sh
#!/bin/bash
DB_HOST=localhost
DB_PORT=3306
MAX_RETRIES=30
RETRY_INTERVAL=10

check_mysql() {
    nc -z "$DB_HOST" "$DB_PORT"
    return $?
}

retry_count=0
while [ $retry_count -lt $MAX_RETRIES ]; do
    if check_mysql; then
        echo "Successfully connected to MySQL at $DB_HOST:$DB_PORT"
        exit 0
    fi
    echo "Attempt $((retry_count + 1))/$MAX_RETRIES: Cannot connect to MySQL at $DB_HOST:$DB_PORT. Retrying in $RETRY_INTERVAL seconds..."
    sleep $RETRY_INTERVAL
    retry_count=$((retry_count + 1))
done

echo "Failed to connect to MySQL after $MAX_RETRIES attempts"
exit 1
```
Make the script executable:
```sh
sudo chmod +x /root/code/script/check-mysql.sh
```

#### Create a Systemd Service for the MySQL Check Script
```sh
sudo vim /etc/systemd/system/mysql-check.service
```
Add the following content:
```ini
[Unit]
Description=MySQL Availability Check
After=network.target

[Service]
Type=oneshot
EnvironmentFile=/etc/environment
ExecStart=/root/code/script/check-mysql.sh
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```

Reload and restart the service:
```sh
sudo systemctl daemon-reload
sudo systemctl start mysql-check
```
Check logs:
```sh
sudo journalctl -u mysql-check -f
```

### 4. Configure the Node.js Application Service
```sh
sudo vim /etc/systemd/system/nodejs-app.service
```
Add the following content:
```ini
[Unit]
Description=Node.js Application
After=mysql-check.service
Requires=mysql-check.service

[Service]
Type=simple
User=nodejs
WorkingDirectory=/opt/app
ExecStart=/usr/bin/node /opt/app/server.js
Restart=always
RestartSec=10
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

Reload and restart the service:
```sh
sudo systemctl daemon-reload
sudo systemctl start nodejs-app
sudo systemctl status nodejs-app
```

### 5. Enable the Service to Start at Boot Time
```sh
sudo systemctl enable nodejs-app
```

### 6. check logs
```sh
sudo journalctl -u nodejs-app -f
```

## Part 4: Testing

### 1. Start Your Service and Verify It's Running
```sh
systemctl status nodejs-app
```

### 2. Test That Your Application Endpoints Work Correctly
Get All Users
```sh
curl --location --request GET 'http://localhost:3000/users'
```
**Response:**
```json
[
  {"id":1,"name":"Sami Khan","email":"sami34@gmail.com"},
  {"id":2,"name":"Piyal Rahman","email":"piyalbd@gmail.com"},
  {"id":3,"name":"Kabir Ahmed","email":"kabir_it@gmail.com"},
  {"id":4,"name":"Fahim Ahmed","email":"fahim33@gmail.com"}
]
```

Health Check
```sh
curl --location --request GET 'http://localhost:3000/health'
```
**Response:**
```sh
Database Connection Successfully Established.
```

---

### 3.Test That Your Service Restarts if the Application Crashes
Stop the application manually and check if it restarts automatically.
```sh
systemctl stop nodejs-app
systemctl status nodejs-app
# Wait a moment and check if it restarts
```

### 4. Reboot Your System and Verify the Service Starts Automatically
```sh
sudo reboot
# After reboot
systemctl status nodejs-app
```

