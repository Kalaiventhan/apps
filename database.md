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
    
