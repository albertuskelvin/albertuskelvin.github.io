---
title: 'Little Note on MySQL and Adminer'
date: 2021-02-16
permalink: /posts/2021/02/mysql-and-adminer/
tags:
  - mysql
  - adminer
---

Just a little note about MySQL and Adminer.

# How to Run Adminer

- Go to the `adminer.php` directory
- Open Terminal and execute `php -S localhost:8080`
- Go to `localhost:8080/adminer.php`

---

# How to Import SQL Dump File

Run the following command:

```
<PATH_TO_MYSQL_BIN_DIR>/mysql -h localhost -u root -p < <PATH_TO_SQL_FILE>
```

# How to Show Table Columns Information

- <b>Approach 1.</b> Run `<PATH_TO_MYSQL_BIN_DIR>/mysqlshow -h localhost -u root -p catapa_benefit_management religions`

- <b>Approach 2.</b> Login to the mysql server by running `<PATH_TO_MYSQL_BIN_DIR>/mysql -h localhost -u root -p`

And run the following commands:

```
SHOW DATABASES;
USE <db_name>;
SHOW TABLES;
SHOW COLUMNS from <table_name>;
```

To enable Adminer to login to mysql, just login to the mysql server and add `ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY ‘<root_password>’;`

---

# Load CSV Into MySQL

- Firstly check whether the local infile variable is on by running `SHOW GLOBAL VARIABLES LIKE 'local_infile';`. If off, then do the followings:

```
> SET GLOBAL local_infile=1;
> Quit
> <PATH_TO_MYSQL_BIN_DIR>/mysql --local-infile=1 -u root -p

Load the CSV!
```

---

# How to Dump MySQL Table

Run the following command: `<PATH_TO_MYSQL_BIN_DIR>/mysqldump -u [uname] -p db_name table1 table2 > table_backup.sql`
