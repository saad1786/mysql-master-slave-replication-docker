# MySQL Master–Slave Replication with Docker

This project demonstrates how to configure **MySQL replication** between two Docker containers:
- `master` → acts as the **primary server (source)**.
- `replica` → acts as the **replica server (slave)**.

Once configured, all changes made on the master are automatically replicated to the replica.

## 📂 Project Structure
```bash
├── docker-compose.yml
├── master/
│ └── my.cnf
├── replica/
│ └── my.cnf
```

## 🚀 How to Run
### Start containers
```bash 
docker-compose up -d
```
### Check master status
```bash
docker exec -it mysql-master mysql -uroot -prootpass

# Inside MySQL shell:
CREATE USER 'repl'@'%' IDENTIFIED WITH mysql_native_password BY 'replpass';
GRANT REPLICATION SLAVE ON *.* TO 'repl'@'%';
FLUSH PRIVILEGES;

# Lock and get master status
FLUSH TABLES WITH READ LOCK;
SHOW MASTER STATUS;
```
Note down the File and Position (e.g. mysql-bin.000003, 157)
### Configure replica
```bash
docker exec -it mysql-replica mysql -uroot -prootpass

# Inside MySQL shell:
CHANGE REPLICATION SOURCE TO
  SOURCE_HOST='mysql-master',
  SOURCE_USER='repl',
  SOURCE_PASSWORD='replpass',
  SOURCE_PORT=3306,
  SOURCE_LOG_FILE='mysql-bin.000003',
  SOURCE_LOG_POS=157;

# Start Replica
START REPLICA;

# Verify Replication
SHOW REPLICA STATUS\G

# Look for
Replica_IO_Running: Yes
Replica_SQL_Running: Yes
```

### Test Replication

Run on master:
```bash
docker exec -it mysql-master mysql -uroot -prootpass -e "CREATE DATABASE testdb; USE testdb; CREATE TABLE t1(id INT); INSERT INTO t1 VALUES(1);"
```
Check on replica:
```bash
docker exec -it mysql-replica mysql -uroot -prootpass -e "SHOW DATABASES;"
docker exec -it mysql-replica mysql -uroot -prootpass -e "SELECT * FROM testdb.t1;"
```
You should see the testdb database and the row replicated.
