# Apache Airflow:

**Apache Airflow** is an open-source platform used to programmatically author, schedule, and monitor data workflows (or data pipelines).

Instead of configuring jobs via static configuration files or simple cron jobs, Airflow allows you to define workflows as **code in Python**.

---

### Key Concepts

* **DAG (Directed Acyclic Graph):** The core building block in Airflow. A DAG represents a collection of all the tasks you want to run, organized in a way that reflects their relationships and dependencies. Because it is acyclic, execution flows in one direction without infinite loops.
* **Operators:** The templates for individual tasks. They define what actually gets done (e.g., executing a Python function, running a SQL query, triggering a Kubernetes pod, or running a Bash command).
* **Tasks:** Instances of operators assigned to a specific node inside a DAG.
* **Scheduler:** The background process that monitors DAG definitions, evaluates dependency conditions, and triggers task instances when their requirements are met.
* **Airflow UI:** A web interface to view running pipelines, inspect logs, monitor task statuses, manually trigger jobs, and visualize task dependencies.

---

### Traditional Scripts/Cron vs. Apache Airflow

| Feature | Traditional Way (Cron + Scripts) | Apache Airflow |
| --- | --- | --- |
| **Workflow Definition** | Cron syntax linking to separate scripts. | Expressed in clean, version-controlled Python code. |
| **Dependencies** | Hard to coordinate complex multi-step pipelines. | Easily managed with explicit DAG dependencies (`task_a >> task_b`). |
| **Monitoring** | Manual log checking or custom email alerts. | Rich web UI with live execution graphs, status indicators, and logs. |
| **Error Handling** | Requires custom retry and alert logic in scripts. | Built-in parameters for retries (`retries`, `retry_delay`), SLAs, and alerts. |
| **Scalability** | Limited to the host running the cron job. | Distributed execution across clusters (e.g., via Celery or Kubernetes executors). |

---

Here is an example of an **ETL (Extract, Transform, Load) pipeline** using Apache Airflow.

### Scenario

1. **Extract:** Fetch raw user data from a mock REST API.
2. **Transform:** Filter out inactive users and format the creation dates.
3. **Load:** Save the cleaned dataset into a database (or a local CSV file for simplicity).

---

### Airflow DAG Example (`etl_pipeline.py`)

```python
from datetime import datetime, timedelta
import json
import pandas as pd
from airflow import DAG
from airflow.operators.python import PythonOperator

# Default arguments applied to all tasks
default_args = {
    "owner": "data_team",
    "depends_on_past": False,
    "email_on_failure": False,
    "retries": 1,
    "retry_delay": timedelta(minutes=5),
}

# 1. EXTRACT TASK
def extract_data(**kwargs):
    # Simulating data extracted from an API or source system
    raw_data = [
        {"id": 1, "name": "Alice", "status": "active", "joined": "2024-01-15T10:00:00"},
        {"id": 2, "name": "Bob", "status": "inactive", "joined": "2024-02-20T11:30:00"},
        {"id": 3, "name": "Charlie", "status": "active", "joined": "2024-03-05T09:15:00"},
    ]
    # Pass extracted data to the next step via Airflow's XCom
    kwargs["ti"].xcom_push(key="raw_users", value=raw_data)
    print("Successfully extracted user data.")

# 2. TRANSFORM TASK
def transform_data(**kwargs):
    ti = kwargs["ti"]
    raw_data = ti.xcom_pull(key="raw_users", task_ids="extract_task")
    
    # Transform using pandas
    df = pd.DataFrame(raw_data)
    
    # Filter for active users only
    df_active = df[df["status"] == "active"].copy()
    
    # Reformat date column
    df_active["joined_date"] = pd.to_datetime(df_active["joined"]).dt.strftime("%Y-%m-%d")
    df_transformed = df_active[["id", "name", "joined_date"]]

    # Output transformed data as JSON string to pass forward
    transformed_json = df_transformed.to_json(orient="records")
    ti.xcom_push(key="cleaned_users", value=transformed_json)
    print(f"Transformed {len(df_transformed)} active records.")

# 3. LOAD TASK
def load_data(**kwargs):
    ti = kwargs["ti"]
    cleaned_json = ti.xcom_pull(key="cleaned_users", task_ids="transform_task")
    data = json.loads(cleaned_json)
    
    # Simulating loading into a database or destination store
    print("--- Loading Clean Data to Destination ---")
    for record in data:
        print(f"Loaded User ID {record['id']}: {record['name']} (Joined: {record['joined_date']})")

# Define the DAG
with DAG(
    dag_id="simple_etl_pipeline",
    default_args=default_args,
    description="An end-to-end ETL pipeline example",
    schedule_interval="@daily",
    start_date=datetime(2024, 1, 1),
    catchup=False,
) as dag:

    # Define tasks using PythonOperator
    task_extract = PythonOperator(
        task_id="extract_task",
        python_callable=extract_data,
    )

    task_transform = PythonOperator(
        task_id="transform_task",
        python_callable=transform_data,
    )

    task_load = PythonOperator(
        task_id="load_task",
        python_callable=load_data,
    )

    # Set explicit dependencies (Extract -> Transform -> Load)
    task_extract >> task_transform >> task_load

```

---

### How Execution Works

1. **`task_extract`** runs first, fetching raw JSON data and pushing it into Airflow's **XCom** key-value store.
2. **`task_transform`** waits for `task_extract` to finish, pulls the raw JSON, uses **pandas** to filter out inactive users, and cleans up dates.
3. **`task_load`** receives the final dataset from the transformation task and loads it into the target destination.
 
    * 
