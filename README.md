# Deploying a Node.js Application with MySQL using systemd

## Part 2: Database Setup

### 1. Install MySQL (if not already installed)
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
Run the following command to secure MySQL and follow the on-screen instructions:
```sh
sudo mysql_secure_installation
```

### 3. Create the Required Database, User, and Table
Access MySQL:
```sh
sudo mysql
```
Run the following SQL commands to create the database and the `users` table:
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

Your MySQL database is now set up and ready for your Node.js application. Proceed with configuring your application to connect to this database and setting up `systemd` for automatic service management.

For more details, refer to the main project documentation.

