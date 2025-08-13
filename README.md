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

#---Folder paths
inbound_path = r"D:\EXCEL_Python_to_SQL\Inbound"
processed_path = r"D:\EXCEL_Python_to_SQL\Processed"
logs_path = r"D:\EXCEL_Python_to_SQL\Logs"

#---Database configuration
server = r"LAPTOP-0PF0BEHQ\MSSQLSERVER01"
database = "Python_to_SQL"
table_name = "HR_DATA_Py_to_SQL"

#---SQLAlchemy connection string for Windows Authentication
conn_str = (
    f"mssql+pyodbc://@{server}/{database}"
    "?driver=ODBC+Driver+17+for+SQL+Server"
    "&Trusted_Connection=yes"
)
engine = create_engine(conn_str)

#---Function to write logs per file
def write_log(file_name, message):
    #create a unique log file for each CSV file (timestamp to avoid overwriting)
    log_file_name = f"log_{os.path.splitext(file_name)[0]}_{datetime.now().strftime('%Y%m%d_%H%M%S')}.txt"
    log_file_path = os.path.join(logs_path, log_file_name)
    with open(log_file_path, 'a') as f:
        f.write(f"{datetime.now()} - {message}\n")
    return log_file_path  #return path if needed

#---Main file processing loop
for file in os.listdir(inbound_path):
    if file.endswith(".csv"):
        file_path = os.path.join(inbound_path, file)
        try:
            #Read CSV into DataFrame
            df = pd.read_csv(file_path)

            #Upload to SQL Server table
            df.to_sql(table_name, con=engine, if_exists='append', index=False)
            write_log(file, f"SUCCESS: Loaded file '{file}' into table '{table_name}'.")

            #Move file to Processed folder
            shutil.move(file_path, os.path.join(processed_path, file))
            write_log(file, f"MOVED: '{file}' to Processed folder.")

        except Exception as e:
            write_log(file, f"ERROR processing file '{file}': {e}")
```

Then convert the saved script(.ipynb) to .py
```
 !jupyter nbconvert --to script csv_to_SQL.ipynb
```

## 3. Batch File (csv_to_sql_loader_run.bat)
Created in C:\Users\tausif shaikh\ with this content:
```
bat
Copy
Edit
@echo off
REM Change to the script directory
cd /d "C:\Users\tausif shaikh"

REM Run the Python script
py "C:\Users\tausif shaikh\Excel_to_SQL.py"

REM Exit cleanly
exit /b 0
```
Save that as a ' .bat' file.

Test:
```
"C:\Users\tausif shaikh\csv_to_sql_loader_run.bat"
```
If your inbound folder has CSVs, you should see logs and processed files update.

## 4. Task Schedular Setup:
**Pre-Requisite**</br>
✅ Python must be installed </br>
Automatically run csv_to_SQL.py daily using Windows Task Scheduler.

1. **Open Task Scheduler** → Create Task.
2. **General Tab:**
   - Name: CSV to SQL Loader
   - Select Run whether user is logged on or not
   - Check Run with highest privileges
3. **Triggers Tab:**
   - New Trigger → Choose schedule (Daily, Hourly, etc.)
4. **Actions Tab:**
   - **New** → Start a program
   - **Program/script:** "C:\Users\tausif shaikh\csv_to_sql_loader_run.bat"
     (Quotes are required because of the space in the path)
5. **Conditions Tab:** Uncheck “Start only if on AC power” if needed.
6. **Settings Tab:** Enable “Allow task to be run on demand”.
7. **Save** → Enter Windows password.

#### ➤ **Test It Now**
1. Right-click the task → Click Run
   - Task Scheduler runs the .bat → which runs Python script → which processes files and      logs results.
   - Task finishes immediately after processing — no hanging.
2. Your SQL table → check new rows appended





