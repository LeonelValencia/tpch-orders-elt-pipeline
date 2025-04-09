# 🧊 Snowflake ELT Pipeline with dbt & Airflow

This project demonstrates how to build and orchestrate an ELT pipeline using **Snowflake**, **dbt**, and **Airflow**, leveraging sample TPCH data for transformations and testing.

It showcases modern data engineering practices including:
- Dimensional data modeling (fact tables, data marts)
- Use of dbt macros, tests, and documentation
- Snowflake RBAC (Role-Based Access Control)
- Pipeline orchestration with Airflow

---

![airflow](/assets/airflow_dbt_success.jpg)

## 🚀 Overview

- Source: `snowflake_sample_data.tpch_sf1.orders`, `lineitem`
- Tools: dbt for transformations, Airflow for orchestration
- Models: Fact tables and aggregated marts
- Tests: Generic (dbt built-in) and custom (singular)
- Snowflake setup: Warehouse, roles, RBAC
- Airflow integration via Astronomer Cosmos

---


## 🛠️ Technologies Used

- **Snowflake**: Cloud data warehouse and data source
- **dbt**: SQL-based transformation tool
- **Airflow**: Pipeline orchestration
- **Git & GitHub**: Version control
- **Jinja / dbt Macros**: Dynamic SQL templating

---

## 🔐 Snowflake RBAC Setup

- Created roles for each environment (dev, staging, prod)
- Assigned privileges for schemas and warehouses
- Used role-based dbt targets for secure access

---

## 🧪 Step-by-Step Breakdown

### 📌 1. Setup Snowflake Environment

- Create target database, schema, and user roles
- Grant permissions appropriately using RBAC principles
- Load TPCH sample data if not available

Use `ACCOUNTADMIN` role to set up necessary components:

```sql
use role accountadmin;

create warehouse dbt_wh with warehouse_size='x-small';
create database if not exists dbt_db;
create role if not exists dbt_role;

grant role dbt_role to user jayzern;
grant usage on warehouse dbt_wh to role dbt_role;
grant all on database dbt_db to role dbt_role;

use role dbt_role;
create schema if not exists dbt_db.dbt_schema;
```

To clean up:

```sql
use role accountadmin;

drop warehouse if exists dbt_wh;
drop database if exists dbt_db;
drop role if exists dbt_role;
```

### 2. Setup dbt Environment

```bash
pip install dbt-snowflake
pip install dbt-core
```

### 3. Configure dbt Profile

```bash
dbt init
cd data_pipeline
```
Edit `~/.dbt/profiles.yml`:

```yaml
tpch_project:
  target: dev
  outputs:
    dev:
      type: snowflake
      account: <account>
      user: <user>
      password: <password>
      role: <role>
      database: DBT_DB
      warehouse: <warehouse>
      schema: DBT_SCHEMA
      threads: 1
```

### 4. Configure `dbt_project.yml` and Install Packages

```yaml
models:
  snowflake_workshop:
    staging:
      materialized: view
      snowflake_warehouse: dbt_wh
    marts:
      materialized: table
      snowflake_warehouse: dbt_wh
```

---

### 5. Create Source and Staging Models

- Staging views in `/models/staging/`

### 6. Create Intermediate and Fact Models

- Final tables in `/models/marts/`

### ✅ 7. Add Macros and Tests

- Generic and singular tests in `/tests/`
- Macros in `/macros/`


**Generic tests — `models/marts/generic_tests.yml`**
```yaml
models:
  - name: fct_orders
    columns:
      - name: order_key
        tests: [unique, not_null]
      - name: status_code
        tests:
          - accepted_values:
              values: ['P', 'O', 'F']
          - relationships:
              to: ref('stg_tpch_orders')
              field: order_key
              severity: warn
```

**Singular test — `tests/fct_orders_discount.sql`**
```sql
select *
from {{ ref('fct_orders') }}
where item_discount_amount > 0
```

**Singular test — `tests/fct_orders_date_valid.sql`**
```sql
select *
from {{ ref('fct_orders') }}
where date(order_date) > CURRENT_DATE()
   or date(order_date) < date('1990-01-01')
```

---

### ☁️ 8. Run and Deploy with Airflow

Install astro

Your Astro project contains the following files and folders:

- dags: This folder contains the Python files for your Airflow DAGs.
- Dockerfile: This file contains a versioned Astro Runtime Docker image that provides a differentiated Airflow experience. If you want to execute other commands or overrides at runtime, specify them here.
- include: This folder contains any additional files that you want to include as part of your project. It is empty by default.
- packages.txt: Install OS-level packages needed for your project by adding them to this file. It is empty by default.
- requirements.txt: Install Python packages needed for your project by adding them to this file. It is empty by default.
- plugins: Add custom or community plugins for your project to this file. It is empty by default.
- airflow_settings.yaml: Use this local-only file to specify Airflow Connections, Variables, and Pools instead of entering them in the Airflow UI as you develop DAGs in this project.

**Dockerfile**
```dockerfile
RUN python -m venv dbt_venv && source dbt_venv/bin/activate && \
    pip install --no-cache-dir dbt-snowflake && deactivate
```

**requirements.txt**
```
astronomer-cosmos
apache-airflow-providers-snowflake
```
Deploy Your Project Locally
===========================

1. Start Airflow on your local machine by running 'astro dev start'.

This command will spin up 4 Docker containers on your machine, each for a different Airflow component:

- Postgres: Airflow's Metadata Database
- Webserver: The Airflow component responsible for rendering the Airflow UI
- Scheduler: The Airflow component responsible for monitoring and triggering tasks
- Triggerer: The Airflow component responsible for triggering deferred tasks

2. Verify that all 4 Docker containers were created by running 'docker ps'.

Note: Running 'astro dev start' will start your project with the Airflow Webserver exposed at port 8080 and Postgres exposed at port 5432. If you already have either of those ports allocated, you can either [stop your existing Docker containers or change the port](https://www.astronomer.io/docs/astro/cli/troubleshoot-locally#ports-are-not-available-for-my-local-airflow-webserver).

3. Access the Airflow UI for your local Airflow project. To do so, go to http://localhost:8080/ and log in with 'admin' for both your Username and Password.

You should also be able to access your Postgres Database at 'localhost:5432/postgres'.

**Create Snowflake Connection in UI (`Conn ID: snowflake_conn`)**
```json
{
  "account": "<account_locator>-<account_name>",
  "warehouse": "dbt_wh",
  "database": "dbt_db",
  "role": "dbt_role",
  "insecure_mode": false
}
```

- Define Airflow DAG to trigger dbt commands:
  - `dbt run`
  - `dbt test`
  - `dbt docs generate`
- Schedule and monitor using Airflow UI

---

## 📌 Key Concepts Demonstrated

✅ ELT pipeline orchestration using Airflow  
✅ Data transformation and modeling using dbt  
✅ Fact table creation and aggregation  
✅ Use of dbt macros and testing framework  
✅ Snowflake RBAC configuration for secure access

---

## 📎 Project Structure

```
├── models/
│   ├── staging/
│   ├── marts/
│   └── tpch_sources.yml
├── macros/
├── tests/
├── dbt_project.yml
├── profiles.yml
└── dags/
    └── dbt_dag.py
```

---

## 📚 Resources

- [dbt Documentation](https://docs.getdbt.com/)
- [Snowflake Documentation](https://docs.snowflake.com/)
- [Apache Airflow](https://airflow.apache.org/)

---

## 🚀 Future Improvements

- Add dimension tables (e.g., part, customer)
- Implement CI/CD with dbt Cloud or GitHub Actions
- Expand test coverage with dbt-utils
- Add dbt documentation site deployment

---

## 👨‍💻 Author

Built with 💙 by [Leonel Valencia](https://github.com/LeonelValencia)
