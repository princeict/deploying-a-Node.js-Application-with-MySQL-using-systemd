# Deploying a Node.js Application with MySQL using systemd

## Part 1: Application Setup

### 1. Create a Node.js Project Directory and Initialize It with npm
Create the application directory and set up Node.js:
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
Create the `server.js` file:
```sh
sudo vim server.js
```
Add the following code:
```js
const express = require('express');
const mysql = require('mysql2');
const app = express();
const port = 3000;

const db = mysql.createConnection({
    host: 'localhost',
    user: 'root',
    password: 'Prince@2024',
    database: 'practice_app'
});

app.get('/health', (req, res) => {
    db.connect((err) => {
        if (err) {
            console.error('MySQL Connection Failed: ', err);
            return res.status(500).send('Database Connection Failed.');
        }
        res.status(200).send('Database Connection Successfully Established.');
    });
});

app.get('/users', (req, res) => {
    db.query('SELECT * FROM users', (err, results) => {
        if (err) {
            return res.status(500).send('Database query error');
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

### 1. Install MySQL (If Not Already Installed)
Update the package list and install MySQL:
```sh
sudo apt update
sudo apt install mysql-server -y
```
Check the MySQL service status:
```sh
sudo systemctl status mysql
```
Start MySQL service if it is not running:
```sh
sudo systemctl start mysql
```

### 2. Secure the MySQL Installation
Run the following command and follow the on-screen instructions:
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
USE practice_app;

CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) NOT NULL
);
```

### 4. Add Sample Data to the Table
Insert sample records into the `users` table:
```sql
INSERT INTO users (name, email) VALUES ('Kabir Khan', 'kabir22@gmail.com');
INSERT INTO users (name, email) VALUES ('Jamil Rahman', 'jamil54@gmail.com');
INSERT INTO users (name, email) VALUES ('Polash Ahmed', 'polash89@gmail.com');
INSERT INTO users (name, email) VALUES ('Rakib Ahmed', 'rakib33@gmail.com');
```

### 5. Verify Data Insertion
Check if the data has been inserted correctly:
```sql
SELECT * FROM users;
```

---

Your Node.js application and MySQL database are now set up and ready to be deployed using `systemd`. For further configurations, refer to the main project documentation.

