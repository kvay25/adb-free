# Oracle Autonomous Database Free Container Image Documentation

Oracle Autonomous Database Free Container image comes with 2 prebuilt Databases - `MY_ATP` and `MY_ADW`. These are similar to Transaction Processing and Data Warehouse workload type databases in Autonomous Database Serverless Cloud service.

Following key features are supported:

- Oracle Rest Data Services (ORDS)
- APEX
- Database Actions
- Mongo API enabled (by default routed to `MY_ATP`)

The storage size is limited to 20 GB for each Database

## Using this image

### Container CPU/memory requirements

Oracle Autonomous Database Free container needs 4 CPUs and 8 GiB memory

### Starting an ADB Free container

To start an Oracle Autonomous Database Free container run the following command 

```bash
podman run -d \
-p 1521:1522 \
-p 1522:1522 \
-p 8443:8443 \
-p 27017:27017 \
--hostname localhost \
--cap-add SYS_ADMIN \
--device /dev/fuse \
--name adb_container \
ghcr.io/oracle/adb-free:latest
```

#### Note:
- For OFS mount, container should start with `SYS_ADMIN` capability. Also, virtual device `/dev/fuse` should be accessible
- `--hostname` is the Fully Qualified Domain Name (FQDN) of your host.

Note the following ports which are forwarded to the container process

| Port | Description                                     |
|------|-------------------------------------------------|
| 1521 | TLS                                             |
| 1522 | mTLS                                            |
| 8443 | HTTPS port for ORDS / APEX and Database Actions |
| 27017 | Mongo API ( MY_ATP )                            |

If you are behind a corporate proxy, pass the proxy environment variables as shown below:

```bash
podman run -d \
-p 1521:1522 \
-p 1522:1522 \
-p 8443:8443 \
-p 27017:27017 \
--hostname localhost \
-e http_proxy=http://my-corp-proxy.com:80/ \
-e https_proxy=http://my-corp-proxy.com:80/ \
-e no_proxy=localhost,127.0.0.1 \
-e HTTP_PROXY=http://my-corp-proxy.com:80/  \
-e HTTPS_PROXY=http://my-corp-proxy.com:80/  \
-e NO_PROXY=localhost,127.0.0.1 \
--cap-add SYS_ADMIN \
--device /dev/fuse \
--name adb_container \
ghcr.io/oracle/adb-free:latest
```

### Connecting to Oracle Autonomous Database Free container

#### Change expired password

You are forced to change Admin user passwords during first login

| Database      | Expired ADMIN password |
|--------|------------------------|
| MY_ATP | Welcome_MY_ATP_1234    |
| MY_ADW | Welcome_MY_ADW_1234    |


Use the container utility script /u01/scripts/change_expired_password.sh to change expired password

```bash
podman exec <container_id> /u01/scripts/change_expired_password.sh <MY_Database> <user> <old_password> <new_password>
```

Example:
```bash
podman exec adb_container /u01/scripts/change_expired_password.sh MY_ATP admin Welcome_MY_ATP_1234 ******

```

```text
SQL*Plus: Release 19.0.0.0.0 - Production on Thu Aug 17 04:00:02 2023
Version 19.20.0.1.0
 
Copyright (c) 1982, 2023, Oracle.  All rights reserved.
 
ERROR:
ORA-28001: the password has expired
 
 
Changing password for admin
New password:
Retype new password:
Password changed
 
Connected to:
Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production
Version 19.20.0.1.0
```

#### ORDS/APEX/Database Actions

During container initialization `--hostname` argument is used to generate self-signed SSL certs to serve HTTPS traffic on port 8443.

For example, if you run the ADB free container on a host whose FQDN is `my.host.com` you can pass `--hostname my.host.com`

APEX and Database Actions can be accessed using the passed hostname (or simply localhost)


| Application      | MY_ATP                                          | MY_ADW                                                              | Example                                            |
|------------------|-------------------------------------------------|---------------------------------------------------------------------|----------------------------------------------------|
| APEX             | https://localhost:8443/ords/my_atp/             | https://localhost:8443/ords/my_adw/                                 | https://my.host.com:8443/ords/my_atp               |
| Database Actions | https://localhost:8443/ords/my_atp/sql-developer | https://localhost:8443/ords/my_adw/sql-developer | https://my.host.com:8443/ords/my_adw/sql-developer |

#### Wallet Setup

In the container, TLS wallet is generated at location `/u01/app/oracle/wallets/tls_wallet`

Copy wallet to your host. 

```bash
podman cp adb_container:/u01/app/oracle/wallets/tls_wallet /scratch/tls_wallet
```
In this example, wallet is copied to `/scratch/tls_wallet` folder

Point `TNS_ADMIN` environment variable to the wallet directory
```bash
export TNS_ADMIN=/scratch/tls_wallet
```

If you want to connect to a remote host where the ADB free container is running, replace `localhost` in `$TNS_ADMIN/tnsnames.ora` with the remote host FQDN

```bash
sed -i 's/localhost/my.host.com/g' $TNS_ADMIN/tnsnames.ora
```

#### Available TNS aliases

Similar to Autonomous Database Serverless Cloud service, use any one of the following aliases to connect to ADB free container.

##### MY_ATP TNS aliases

For mTLS use the following 
- my_atp_medium
- my_atp_high
- my_atp_low
- my_atp_tp
- my_atp_tpurgent

For TLS use the following

- my_atp_medium_tls
- my_atp_high_tls
- my_atp_low_tls
- my_atp_tp_tls
- my_atp_tpurgent_tls

##### MY_ADW TNS aliases

For mTLS use the following 
- my_adw_medium
- my_adw_high
- my_adw_low

For TLS use the following
- my_adw_medium_tls
- my_adw_high_tls
- my_adw_low_tls

TNS alias mappings to their connect string can be found in` $TNS_ADMIN/tnsnames.ora` file.

#### SQL*Plus

In this example, we connect using the alias `my_atp_low`
```text
sqlplus admin/<my_atp_admin_password>@my_atp_low

SQL*Plus: Release 21.0.0.0.0 - Production on Wed Jul 26 22:38:27 2023
Version 21.9.0.0.0

Copyright (c) 1982, 2022, Oracle.  All rights reserved.

Last Successful login time: Wed Jul 26 2023 16:36:16 +00:00

Connected to:
Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production
Version 19.20.0.1.0

SQL> 
```

#### Python thin driver

Install thin driver in your Python 3 environment
```bash
pip install oracledb
```

```python
import oracledb
dsn = "admin/<my_adw_admin_password>@my_adw_medium"
conn = oracledb.connect(dsn=dsn, wallet_location="/scratch/tls_wallet")
cr = conn.cursor()
r = cr.execute("SELECT 1 FROM DUAL")
print(r.fetchall())

>> [(1,)]
```


### Create an app user

Connect as Admin
```bash
sqlplus admin/<my_atp_admin_password>@my_atp_medium
```
Create user as shown below:
```sql
CREATE USER APP_USER IDENTIFIED BY "<my_app_user_password>" QUOTA UNLIMITED ON DATA;
 
-- ADD ROLES
GRANT CONNECT TO APP_USER;
GRANT CONSOLE_DEVELOPER TO APP_USER;
GRANT DWROLE TO APP_USER;
GRANT GRAPH_DEVELOPER TO APP_USER;
GRANT RESOURCE TO APP_USER;  
 
 
-- ENABLE REST
BEGIN
    ORDS.ENABLE_SCHEMA(
        p_enabled => TRUE,
        p_schema => 'APP_USER',
        p_url_mapping_type => 'BASE_PATH',
        p_url_mapping_pattern => 'app_user',
        p_auto_rest_auth=> TRUE
    );
    commit;
END;
/
 
-- QUOTA
ALTER USER APP_USER QUOTA UNLIMITED ON DATA;

```

## F.A.Q

### How can I run Oracle Autonomous Database Free container on ARM64 arch i.e. machines with M1/M2 chips ?
Use colima + docker to emulate x86_64 arch. Replace podman with docker in all commands. This is only until we have a native ARM 64 image.

### How can I install colima and docker on machines with M1/M2 chips ?
```bash
brew install docker
brew install docker-compose
brew install colima
brew reinstall qemu
```

### How can I start Colima x86_64 Virtual Machine with minimum memory/cpu requirements ?
```bash
colima start --cpu 4 --memory 8 --arch x86_64
```

### How can I start podman VM on x86_64 Mac with minimum memory/cpu requirements ?
```bash
podman machine init
podman machine set --cpus 4 --memory 8192
podman machine start
```