# Data-Loading-CSV-to-SQL-Table
Create a Python script that automatically extracts a CSV file from the inbound folder and loads it into the SQL table daily by setting up a task in the task scheduler, thereby eliminating manual activity.


## PROJECT REQUIREMENTS
✅ Read all .csv files from the Inbound folder </br>
✅ Upload them into the SQL Server table </br>
✅ Move successfully uploaded files to the Processed folder </br>
✅ Log all activities (success and errors) to the Logs folder </br>

### Source Folder Structure </br>
├── Inbound\      ➤  New incoming CSV files (source) </br>
├── Processed\    ➤  Processed files moved here after upload </br>
└── Logs\         ➤  Logs saved as .txt with details </br>

## 1. SQL Server Setup: 
Create SQL Table (Run in SSMS or Python)
   - Create a table where the CSV structure must match the SQL table schema.

## 2. Coding: Write a Python script
   - Prepare a Python code (here, the platform is Jupyter). 

```
import os
import shutil
import pandas as pd
from datetime import datetime
from sqlalchemy import create_engine

#--Folder paths (update only if needed)<br>
inbound_path = r"D:\EXCEL_Python_to_SQL\Inbound"
processed_path = r"D:\EXCEL_Python_to_SQL\Processed"
logs_path = r"D:\EXCEL_Python_to_SQL\Logs"

#Database configuration
server = r"LAPTOP-0PF0BEHQ\MSSQLSERVER01"
database = "Python_to_SQL"
table_name = "HR_DATA_Py_to_SQL"

#-------------------------
#SQLAlchemy connection string for Windows Authentication
conn_str = (
    f"mssql+pyodbc://@{server}/{database}"
    "?driver=ODBC+Driver+17+for+SQL+Server"
    "&Trusted_Connection=yes"
)
engine = create_engine(conn_str)

#-------------------------
#Function to write logs
def write_log(message):
    log_file = os.path.join(logs_path, f"log_{datetime.today().strftime('%Y-%m-%d')}.txt")
    with open(log_file, 'a') as f:
        f.write(f"{datetime.now()} - {message}\n")
        
#-------------------------
#Main file processing loop
for file in os.listdir(inbound_path):
    if file.endswith(".csv"):
        file_path = os.path.join(inbound_path, file)
        try:
            #Read CSV into DataFrame
            df = pd.read_csv(file_path)

            #Upload to SQL Server table
            df.to_sql(table_name, con=engine, if_exists='append', index=False)
            write_log(f"SUCCESS: Loaded file '{file}' into table '{table_name}'.")

            #Move file to Processed folder 
            shutil.move(file_path, os.path.join(processed_path, file))
            write_log(f"MOVED: '{file}' to Processed folder.")

        except Exception as e:
            write_log(f"ERROR processing file '{file}': {e}")
```

Then convert the saved script(.ipynb) to .py
```
 !jupyter nbconvert --to script csv_to_SQL.ipynb
```


## 3. Automation: Windows: 
**Task Scheduler**
   - → Run script e.g. weekly/daily/hourly

### Task Schedular Setup:
Automatically run csv_to_SQL.py daily using Windows Task Scheduler.

**Pre-Requisite**</br>
✅ Python must be installed </br>
✅ Python must be added to the system PATH (or we’ll use the full path to python.exe) </br>


➤  **STEP 1: Open Task Scheduler**
- Press Win + S, type Task Scheduler, and open it
- Click "Create Basic Task

➤ **STEP 2: Fill in Task Details**
- Name: Automate CSV to SQL Upload
- Trigger: Then choose the start time (e.g., 09:00 AM) </br>
- Action: Start a program
  
➤ **STEP 3: Configure Script Execution**
- Program/script field:
- Add arguments field:
- Start in (Optional but safe to add):

➤ **STEP 4: Finalize Task**
- Click Next
- Review details
- Click Finish

➤ **STEP 5 (RECOMMENDED): Run with Highest Privileges**
- In Task Scheduler, find your task
- Right-click → Properties
- On the General tab:
✅ Check: “Run with highest privileges”
✅ Select: “Run whether user is logged on or not”

➤ **STEP 6: Test It Now**
- Right-click the task → Click Run
- Open:
   1. Your SQL table → check new rows
   2. Logs folder → check today's log file





