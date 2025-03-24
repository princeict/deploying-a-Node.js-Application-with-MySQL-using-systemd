# Deploying a Node.js Application with MySQL using systemd

## Part 1: Application Setup

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

## Part 2: Database Setup

### 1. Install MySQL if Not Already Installed
```sh
sudo apt update
sudo apt install mysql-server -y
sudo systemctl status mysql
sudo systemctl start mysql
```

### 2. Secure the MySQL Installation
```sh
sudo mysql_secure_installation
```

### 3. Create the Required Database, User, and Table
Access MySQL:
```sh
sudo mysql
```
Run the following SQL commands:
```sql
CREATE DATABASE practice_app;
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) NOT NULL
);
```

### 4. Add Sample Data to the Table
```sql
INSERT INTO users (name, email) VALUES ('Kabir Khan', 'kabir22@gmail.com');
INSERT INTO users (name, email) VALUES ('Jamil Rahman', 'jamil54@gmail.com');
INSERT INTO users (name, email) VALUES ('Polash Ahmed', 'polash89@gmail.com');
INSERT INTO users (name, email) VALUES ('Rakib Ahmed', 'rakib33@gmail.com');
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
DB_HOST="$DB_PRIVATE_IP"
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

### 6. Stop Any Running Node.js Server
```sh
sudo kill $(pgrep -f server.js)
```

