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
        - It provide Reports
    - How it works?
        - You write migration scripts in SQL or Java.
        - Name them with a version pattern, e.g., V1__init.sql, V2__add_table.sql.
        - Flyway connects to the database and checks the schema history table (flyway_schema_history) to see which migrations have run.
        - It applies any new migrations in order.
        - Updates the history table after successful execution.
    
# My application pod is running on server A. I want to execute DB scripts on Server B.
  - Your pod does NOT execute scripts “on Server B”. It connects to the database listener running on Server B over the network and executes SQL remotely.
  - In databases, scripts always run where the DB is, not where the client runs.
  - [POD on Server A]
        |
        |  (TCP 1521 / JDBC)
        v
   [Oracle DB on Server B]
 - Methods
     - Flyway inside the pod - BEST PRACTICE
         - Separate POD
         - initcontainer inside application POD
     - sqlplus from pod (traditional)
     - Kubernetes Job / InitContainer (production-grade)
  
   Preserve vs Unpreserve data
   Uncouple from application POD

   v1.0.0 (app version) -- it has v1 file
   v1.0.1 (app version) -- it has v1 +v2 files 

   flyway clean and repair

   If fails with DB scripts it would go to previous version. Can we fail the initcontainer? I dont want to run backend container with failed Db scripts
   Yes — fail the initContainer
  ✅ Backend container will not start
  ✅ This is the correct Kubernetes + Flyway pattern
  ❌ Flyway does NOT rollback automatically
  ❌ Don’t allow app to start on failed DB schema

  isolated -- not integrated with cicd.

  dweaver vs SQL developer

  # DB scripts execution using CI/CD
    **CONN**
    job1 - v1 change
    job2 - v2 (update/new) - Delta
    job3 - v3 (update/new) - Delta
    For already deployed machine, just deploy the delta job.
    For new machine - Need golden job- v1+v2+v3  (Include all jobs - job1/2/3)

    oTHER WAY:
    job1 - v1 change
    job2 - v2 (update/new + v1) - Delta + Base changes
    job3 - v3 (update/new + v1 + V2) - Delta + Base changes
    For already deployed or new machines --> Always the latest job. 
    

  

  
  
   
