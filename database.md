# Schema vs DB users
  - In Oracle, a USER and a SCHEMA are essentially the same thing. A schema is just the collection of objects owned by a user.
  - What is a USER in Oracle? A user is a,
      - login account Has a password
      - Can connect to the database
  - A schema is:
    - A logical container for database objects
    - Tables, views, indexes, procedures, sequences, etc.  
    - You never explicitly create a schema in Oracle.
       - The schema is automatically created when the user is created
       - Schema name = user name
      
          So:
          User   : DEVUSER
          Schema : DEVUSER
          Key relationship (very important)
          Concept	Oracle meaning
          USER	Authentication (login)
          SCHEMA	Object ownership
          Relationship	1 user = 1 schema (same name)

  - Multiple users accessing one schema (common in real life)

    Enterprise pattern: (User	Purpose)
      - FLYWAY_USER	Runs migrations
      - APP_USER	Application runtime
      - REPORT_USER	Read-only reports

    How it works?
      - FLYWAY_USER has privileges on APP_SCHEMA
      - APP_USER owns the schema

# Ways to execute SQL files 

  - Direct
    
      su - oracle -c "sqlplus / as sysdba @/home/oracle/SqlFileName.sql"
  - From Shell Script
    
      su - oracle -c '/home/oracle/ShellScript.sh'
      
        sqlplus -silent "/ as sysdba" <<EOF
           @/home/oracle/SqlFileName.sql
        EOF

Can we execute using database user like devuser?
       No, we have to provide the password. If we execute manually, it is fine. But, **it is a risk** if we store password and execute as part of CI/CD.

  
# Flyway - Database migration/version control tool
  - Think of it like Git for your database schema — you track changes, apply them in order, and make sure every environment (dev, test, prod) is consistent
      You have scripts:
        V1__init.sql
        V2__add_user_table.sql
        V3__add_email_column.sql

        First run (new DB)
        Flyway runs:
        V1 → V2 → V3

        Later you add:
        V4__add_index.sql

        Next deployment
        Flyway runs:
        ONLY V4
    - Flyway creates one history table per schema, not per user.
    - Benefits
        - Version Control for Databases
        - Consistent environments (Not like work on one environment and not on other)
        - Supports multiple database engines (Oracle, Postgres, MySQL, SQL Server, etc.)
        - Flyway can undo failed migrations (if designed) or mark scripts as applied/ignored. Helps recover from partial deployments.
        - Flyway fits well into automated pipelines, e.g., you can run migrations as part of Jenkins, GitHub Actions, or Kubernetes deployments.
    - How it works?
        - You write migration scripts in SQL or Java.
        - Name them with a version pattern, e.g., V1__init.sql, V2__add_table.sql.
        - Flyway connects to the database and checks the schema history table (flyway_schema_history) to see which migrations have run.
        - It applies any new migrations in order.
        - Updates the history table after successful execution.
    
