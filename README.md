# zabbix-ha
Install and Configuration Of Zabbix In Docker With BinLog RFeplication In MySQL


Master Side DataBase
***
1. After Execute docker-compose up -d For First Time Stop All Container Except mysql-server Container
2. Go to mysql-server container with doccker-compose exec And create backup of zabbix DB with mysqldump </br>
   ```mysqldump -u root -p --single-transaction --master-data=2 --routines --triggers --databases zabbix zabbix_proxy --set-gtid-purged=OFF > zabbix_backup.sql```
3. Create MYSQL User As Replic User Type </br>
   ```CREATE USER 'replica_user'@'%' IDENTIFIED WITH mysql_native_password BY 'replica_pass';```</br>
```GRANT REPLICATION SLAVE ON *.* TO 'replica_user'@'%';```</br>
```FLUSH PRIVILEGES;```</br>
4. Use Follow Option For mysql docke-compose For Enabling BinLog Replication 
   ```
    command:
       - mysqld
       - --server-id=1                  # Unique server ID for replication
       - --log-bin=mysql-bin            # Enable binary logging
       - --binlog-format=ROW            # Use ROW-based binary log format
       - --binlog-do-db=zabbix            # Restrict binary logging to 'pdns' database
       - --skip-mysqlx                  # (Optional) Disable MySQL X plugin
       - --character-set-server=utf8mb4
       - --collation-server=utf8mb4_bin
       - --log_bin_trust_function_creators=1
   ``` 
   
  5. Use Follow Command In Mysql Client CLI To Find bin-file and Position
     
     ```
       mysql> SHOW MASTER STATUS;
        +------------------+----------+--------------+------------------+-------------------+
        | File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
        +------------------+----------+--------------+------------------+-------------------+
        | mysql-bin.000003 | 30462129 | zabbix       |                  |                   |
        +------------------+----------+--------------+------------------+-------------------+

     ```

Slave Side DataBase
***
1. Use Follow Option For mysql docke-compose For Enabling BinLog Replication 
   ```
    command:
       - mysqld
       - --server-id=2                 # Unique server ID for replication
       - --log-bin=mysql-bin            # Enable binary logging
       - --binlog-format=ROW            # Use ROW-based binary log format
       - --binlog-do-db=zabbix            # Restrict binary logging to 'pdns' database
       - --skip-mysqlx                  # (Optional) Disable MySQL X plugin
       - --relay_log=relay-bin
       - --read-only=ON
       - --character-set-server=utf8mb4
       - --collation-server=utf8mb4_bin
       - --log_bin_trust_function_creators=1
   ``` 
2. After Execute docker-compose.yaml for mysql container 
3. Go To exec Mode Of mysql Container
4. Restore Backup
   ```
    mysql -u root -p < zabbix_backup.sql

   ```
5. Enable Replication With Follow Command In Mysql Client
   ```
    CHANGE MASTER TO 
      MASTER_HOST='185.212.194.97',
      MASTER_PORT=3306,
      MASTER_USER='replica_user',
      MASTER_PASSWORD='replica_pass',
      MASTER_LOG_FILE='mysql-bin.000003',
      MASTER_LOG_POS=31085635;
   ```
   ```
      START SLAVE;
      SHOW SLAVE STATUS\G

   ```

6. Test Replication With Create Table In zabbix DataBase In Master With:
   ```
      CREATE TABLE zabbix.aaaaaaaa(
    itemid bigint(20) unsigned NOT NULL,
    clock int(11) NOT NULL DEFAULT '0',
    value double(16,4) NOT NULL DEFAULT '0.0000',
    ns int(11) NOT NULL DEFAULT '0',
    PRIMARY KEY (itemid,clock,ns)
    ) ENGINE=InnoDB ROW_FORMAT=COMPRESSED;
   ```

      

   

   
