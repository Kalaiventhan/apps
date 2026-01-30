# Ways to execute SQL files 

  - Direct
      su - oracle -c "sqlplus / as sysdba @/home/oracle/SqlFileName.sql"
  - From Shell Script
      su - oracle -c '/home/oracle/ShellScript.sh'
      #!/bin/sh
      # This script helps to execute DB script file.
      #
      ######################################################################
      sqlplus -silent "/ as sysdba" <<EOF
         @/home/oracle/SqlFileName.sql
      EOF
  
