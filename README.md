# Project Title

Restoring the customer repository database and environment into Qlik vApp by keeping the customer environment  host name

## Getting Started

Obtain a copy of the customer's repository database backup from pg_dump  database.tar file


### Prerequisites

Once you get a copy of the customer repository database .tar file do the steps below

1. Stop all Qlik Sense Services except the repository database

2. Get a copy of the customer's repository database backup from pg_dump  database.tar file

3. Connect to PostGres repository database

4. Disconnect QSR database then rename it to QSR-OLD ( run script to rename connect other DB then swap )  The Latest PGAdmin4 has the option to disconnect

```
SELECT * FROM pg_stat_activity 
    WHERE datname = 'QSR';

SELECT  pg_terminate_backend (pid)
FROM  pg_stat_activity
WHERE datname = 'QSR';

ALTER DATABASE "QSR" RENAME TO "QSROLD";
```

5. Restore the customer database. In this case create a database called for example  DMC by running the commands below, then restore the database to DMC from the backup .tar file

```
cd C:\Program Files\Qlik\Sense\Repository\PostgreSQL\9.6\bin
createdb -h localhost -p 4432 -U postgres -T template0 DMC
pg_restore.exe -h localhost -p 4432 -U postgres -d DMC"C:\DMC-Backup\QSR_backup_20190121.tar"
```


6. Disconnect DMC database then rename it to DMCBAK

```
SELECT * FROM pg_stat_activity 
    WHERE datname = 'DMC';

SELECT  pg_terminate_backend (pid)
FROM  pg_stat_activity
WHERE datname = 'DMC';

ALTER DATABASE "DMC" RENAME TO "DMCBAK";
```

7. Do step 5 again and restore DMC then remane it to QSR - As the Qlik Sense default database

```
SELECT * FROM pg_stat_activity 
    WHERE datname = 'DMC';

SELECT  pg_terminate_backend (pid)
FROM  pg_stat_activity
WHERE datname = 'DMC';

ALTER DATABASE "DMC" RENAME TO "QSR";
```

8. Get the customer environment hostname from the table ServerNodeConfigurations, for example:
```
prodp001.testdomain.local
```

9. Open c:\windows\system32\drivers\etc\hosts 
file and add an entry with the customer hostname prodp001.testdomain.local  to set it up as localhost 127.0.0.1

10. Use Microsoft Management Console(MMC) to delete all the Qlik Sense related certificates. ( local User, trusted root )

11. Delete the certificate files in the folder: %ProgramData%\Qlik\Sense\Repository\Exported Certificates\.Local Certificates

12. Make a copy of %ProgramData%\Qlik\Sense\Host.cfg and rename the copy to Host.cfg.old
Host.cfg contains the hostname encoded in base64. Generate this string for the new hostname using a site such as base64encode. 
prodp001.testdomain.local
emlwZWFwcDAwMXNhLnByb2QuaWRjLnhlbg==

13. Open Host.cfg and replace the content with the new string for teh encrypted host name ( customer host name ) 
emlwZWFwcDAwMXNhLnByb2QuaWRjLnhlbg==


14. Open the hosts.cfg file and change it from QlikServer1.domain.local to new host name


15. Generate certificates with - bootstrap command by running repository.exe - and pass it the bootstrap and restorehostname attribute
```
cd C:\Program Files\Qlik\Sense\Repository\
Repository.exe -bootstrap -iscentral -restorehostname
```

16. Lookup the users in the repository database "Users" table.  Set a certain user "administrator"  role to RootAdmin in the users table

( To gain access and impersonate logged user, run the separete powershell script see: https://github.com/yoichiH01/LoginSense ) 

Otherwise you will be locked out and not able to login. This will be done via a token and PS script above


17. Change shared folder path via QS via util to vApp default shared folder path


18. Start all Qlik sense services 

## Deployment




## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details


