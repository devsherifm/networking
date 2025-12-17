#  PostgreSQL + Netiko integration walkthrough

Please follow below steps for the lab.

---

# 🔹 OVERVIEW

You will:

1. Install PostgreSQL on Ubuntu 24.04
2. Create database + table
3. Install Python dependencies (netmiko, psycopg2)
4. Create **two Netmiko scripts** (Here we are running both script in single code)

   * Script 1 → `show ip int br`
   * Script 2 → `show version`
    
5. Each execution inserts a row into PostgreSQL:

   * `id`
   * `host`
   * `username`
   * `script_name`
   * `execution_time`
   * `exec_status`

---

# 1️⃣ Install PostgreSQL on Ubuntu 24.04 LTS

```bash
sudo apt update
sudo apt install -y postgresql postgresql-contrib
```

Start & enable PostgreSQL:

```bash
sudo systemctl enable postgresql
sudo systemctl start postgresql
```

Verify:

```bash
sudo systemctl status postgresql
```

---

# 2️⃣ Create Database and User

Switch to postgres system user:

```bash
sudo -i -u postgres
```

Open psql:

```bash
psql
```

### Create DB user (you will use this in Python)

```sql
CREATE USER netops_user WITH PASSWORD 'netops_pass';
```

### Create database

```sql
CREATE DATABASE netops_db OWNER netops_user;
```

### Grant privileges

```sql
GRANT ALL PRIVILEGES ON DATABASE netops_db TO netops_user;
```

Exit:

```sql
\q
exit
```

---

# 3️⃣ Create Table

Login as postgres again:

```bash
sudo -i -u postgres
psql netops_db
```

Create table:

```sql
CREATE TABLE script_execution_log (
    id SERIAL PRIMARY KEY,
    username TEXT NOT NULL,
    script_name TEXT NOT NULL,
    execution_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    exec_status TEXT NOT NULL
);
```

Verify:

```sql
\d script_execution_log;
```

GRANT PRIVILEGES to the table `script_execution_log`
```sql
GRANT INSERT, SELECT ON script_execution_log TO netops_user;
GRANT USAGE, SELECT ON SEQUENCE script_execution_log_id_seq TO netops_user;
```

Verify:

```sql
\dp script_execution_log
```

Exit:

```sql
\q
exit
```

---

# 4️⃣ Python Environment Setup

From your directory:

```bash
cd /home/kali/
```

Activate venv (assuming already created):

```bash
source pytest/bin/activate
```

Install required libraries:

```bash
pip install netmiko psycopg2-binary
```

---

# 5️⃣ Device Inventory (for Containerlab)

We’ll use **one script for both commands**, controlled by function.

Create file:

```bash
nano netmiko_runner.py
```

---

# 6️⃣ Python Script (Netmiko + PostgreSQL)

### ✅ Features:

* Username → `input()`
* Password → `getpass()`
* DB credentials asked during runtime
* Logs success/failure automatically

### 📄 `netmiko_runner.py`

```python
from netmiko import ConnectHandler
from getpass import getpass
import psycopg2

# ---- DEVICE INVENTORY ---- (here 192.168.100.30 is not exist)
devices = [
    {"device_type": "cisco_ios", "host": "192.168.100.31"},
    {"device_type": "cisco_ios", "host": "192.168.100.32"},
    {"device_type": "cisco_ios", "host": "192.168.100.30"},
    {"device_type": "cisco_ios", "host": "192.168.100.40"},
]

# ---- USER INPUT ----
username = input("Device Username: ")
password = getpass("Device Password: ")

db_user = input("PostgreSQL Username: ")
db_pass = getpass("PostgreSQL Password: ")

# ---- DB CONNECTION ----
conn = psycopg2.connect(
    host="localhost",
    database="netops_db",
    user=db_user,
    password=db_pass
)
conn.autocommit = True
cursor = conn.cursor()

def log_to_db(host, username, script_name, status):
    try:
        cursor.execute(
            """
            INSERT INTO script_execution_log
            (host, username, script_name, exec_status)
            VALUES (%s, %s, %s, %s)
            """,
            (host, username, script_name, status)
        )
    except Exception as e:
        print(f"DB logging failed for {host}: {e}")

def run_command(command, script_name):
    for device in devices:
        device["username"] = username
        device["password"] = password

        try:
            net_connect = ConnectHandler(**device)
            output = net_connect.send_command(command)
            print(f"\n--- {device['host']} OUTPUT ---")
            print(output)
            net_connect.disconnect()

            log_to_db(device["host"], username, script_name, "SUCCESS")

        except Exception as e:
            print(f"Error on {device['host']}: {e}")
            log_to_db(device["host"], username, script_name, "FAILED")

# ---- EXECUTION ----
run_command("show ip interface brief", "show_ip_int_br")
run_command("show version", "show_version")

cursor.close()
conn.close()
```

Save & exit.

---

# 7️⃣ Run the Script

```bash
python netmiko_runner.py
```

You will be prompted for:

* Device username
* Device password
* PostgreSQL username
* PostgreSQL password

---

# 8️⃣ Verify Database Logs

```bash
sudo -i -u postgres
psql netops_db
```

```sql
SELECT * FROM script_execution_log;
```

Expected Output:

```
 id |      host      | username |  script_name   |       execution_time       | exec_status
----+----------------+----------+----------------+----------------------------+-------------
  1 | 192.168.100.31 | admin    | show_ip_int_br | 2025-12-17 15:14:01.902217 | SUCCESS
  2 | 192.168.100.32 | admin    | show_ip_int_br | 2025-12-17 15:14:03.476673 | SUCCESS
  3 | 192.168.100.31 | admin    | show_version   | 2025-12-17 15:14:05.514423 | SUCCESS
  4 | 192.168.100.32 | admin    | show_version   | 2025-12-17 15:14:07.679121 | SUCCESS
  5 | 192.168.100.31 | admin    | show_ip_int_br | 2025-12-17 15:16:16.000005 | SUCCESS
  6 | 192.168.100.32 | admin    | show_ip_int_br | 2025-12-17 15:16:17.271443 | SUCCESS
  7 | 192.168.100.30 | admin    | show_ip_int_br | 2025-12-17 15:16:20.441474 | FAILED
  8 | 192.168.100.40 | admin    | show_ip_int_br | 2025-12-17 15:16:22.737876 | SUCCESS
  9 | 192.168.100.31 | admin    | show_version   | 2025-12-17 15:16:24.838381 | SUCCESS
 10 | 192.168.100.32 | admin    | show_version   | 2025-12-17 15:16:26.490539 | SUCCESS
 11 | 192.168.100.30 | admin    | show_version   | 2025-12-17 15:16:29.654275 | FAILED
 12 | 192.168.100.40 | admin    | show_version   | 2025-12-17 15:16:31.742026 | SUCCESS
```
