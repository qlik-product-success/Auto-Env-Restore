# Project Title

Restoring a Qlik Sense repository database and environment into an other environment. This can be required for example to replicate a customer environment during troubleshooting.

For migration of Qlik Sense site please consult Qlik Support for more advice.
[Qlik Support KB: How To Migrate Qlik Sense](https://support.qlik.com/apex/QS_CoveoSearch#q=how%20to%20migrate%20qlik%20sense&t=Knowledge&sort=relevancy)

### Pre-Requisites

- PostgreSQL database dump (.tar file) from source site. See [Backing up Qlik Sense site](https://help.qlik.com/en-US/sense/February2019/Subsystems/PlanningQlikSenseDeployments/Content/Sense_Deployment/Backing-up-a-site.htm) in Qlik Sense Help for details
- Qlik Sense Enterprise installed. This guide assuems default single node installation as target environment. 

### Restore database

1. Stop all Qlik Sense Services except the repository database

1. Connect to Qlik Sense Repository databse with `pg_admin`

1. Disconnect QSR database then rename it to QSR-OLD ( run script to rename connect other DB then swap ) The Latest PGAdmin4 has the option to disconnect

    ```
    SELECT * FROM pg_stat_activity 
        WHERE datname = 'QSR';

    SELECT  pg_terminate_backend (pid)
    FROM  pg_stat_activity
    WHERE datname = 'QSR';

    ALTER DATABASE "QSR" RENAME TO "QSROLD";
    ```

1. Restore the source database. In this case create a database called for example DMC by running the commands below, then restore the database to DMC from the backup .tar file

    ```
    cd "$env:ProgramFiles\Qlik\Sense\Repository\PostgreSQL\9.6\bin"
    
    createdb -h localhost -p 4432 -U postgres -T template0 DMC
    
    pg_restore.exe -h localhost -p 4432 -U postgres -d DMC "C:\DMC-Backup\QSR_backup_20190121.tar"
    ```

1. Disconnect DMC database then rename it to DMCBAK

    ```
    SELECT * FROM pg_stat_activity 
        WHERE datname = 'DMC';

    SELECT  pg_terminate_backend (pid)
    FROM  pg_stat_activity
    WHERE datname = 'DMC';

    ALTER DATABASE "DMC" RENAME TO "DMCBAK";
    ```

1. Do step 5 again and restore DMC then remane it to QSR - As the Qlik Sense default database

    ```
    SELECT * FROM pg_stat_activity 
        WHERE datname = 'DMC';

    SELECT  pg_terminate_backend (pid)
    FROM  pg_stat_activity
    WHERE datname = 'DMC';

    ALTER DATABASE "DMC" RENAME TO "QSR";
    ```

### Generate certificate

1. Get the customer environment hostname from the table ServerNodeConfigurations, for example:

    ```
    prodp001.testdomain.local
    ```

1. Open Windows hosts file (c:\windows\system32\drivers\etc\hosts) and add an entry for the imported hostname with `127.0.0.1 prodp001.testdomain.local`

    ```
    notepad.exe c:\windows\system32\drivers\etc\hosts
    ```

1. Use Microsoft Management Console(MMC) to delete all the Qlik Sense related certificates. ( local User, trusted root )

1. Remove current local Qlik Sense certificates from *%ProgramData%\Qlik\Sense\Repository\Exported Certificates\.Local Certificates*

    ```
    Compress-Archive -Path "$env:ProgramData\Qlik\Sense\Repository\Exported Certificates\.Local Certificates\*.pem" -CompressionLevel Optimal -DestinationPath "$env:ProgramData\Qlik\Sense\Repository\Exported Certificates\.Local Certificates\Old-Cert.Zip"

    Remove-Item "$env:ProgramData\Qlik\Sense\Repository\Exported Certificates\.Local Certificates\*" -Include *.pem -Exclude *.zip
    ```

1. Make a copy of %ProgramData%\Qlik\Sense\Host.cfg and rename the copy to Host.cfg.old

    ```
    Copy-Item "$env:ProgramData\Qlik\Sense\Host.cfg" -Destination "$env:ProgramData\Qlik\Sense\Host.cfg.old"
    ```

1. Host.cfg contains the hostname encoded in base64. Generate this string through Powershell, for example `prodp001.testdomain.local` becomes 
`cHJvZHAwMDEudGVzdGRvbWFpbi5sb2NhbA==`
    ```
    [Convert]::ToBase64String([System.Text.Encoding]::UTF8.GetBytes("prodp001.testdomain.local"))
    ```

1. Edit Host.cfg and replace the content with the new string for the encrypted host name `cHJvZHAwMDEudGVzdGRvbWFpbi5sb2NhbA==`

1. Open the hosts.cfg file and change it from default hostname to new host name

1. Generate certificates with - bootstrap command by running repository.exe - and pass it the bootstrap and restorehostname attribute

    ```
    cd C:\Program Files\Qlik\Sense\Repository\
    Repository.exe -bootstrap -iscentral -restorehostname
    ```
### Enable Access

1. Lookup the users in the repository database "Users" table.  Set a certain user "administrator"  role to RootAdmin in the users table

    ( To gain access and impersonate logged user, run the separete powershell script see: https://github.com/yoichiH01/LoginSense ) 

    Otherwise you will be locked out and not able to login. This will be done via a token and PS script above

1. Change shared folder path via QS via util to vApp default shared folder path

1. Start all Qlik Sense services 


## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details


