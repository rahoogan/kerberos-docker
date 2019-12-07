# Kerberos on Docker

WARNING: This project is not production ready, so use at your own risk!

This project is an experiment to run an fully integrated on-demand kerberos cluster using docker. The components include:
  - MIT Kerberos KDC (Key Distribution Center)
  - MIT Kerberos Administration Server
  - 389ds LDAP Directory Server
  - BIND DNS Server

Possible use cases might be:
  - Testing configuration changes
  - Testing performance/upgrades
  - Running applications which require kerberos authentication (Definitely not production!)

## Run

### Step 1. Specify Environment Variables
The docker-compose config expects the following environment variables to be defined - either in a `.env` file, or individual environment variables:

```bash
# Directory on host to use as volume to store configs
KRB_SRC_CONF_LOC=

# Mount location in container for kerberos configs
KRB_CONF_MNT=

# Directory on host to use as volume to store keytabs
KRB_SRC_KEYTAB_LOC=

# Mount location in container for keytabs
KRB_KEYTAB_MNT=

# Directory on host to store kerberos state information
KRB_SRC_STATE_LOC=

# Mount location within container for kerberos state info
KRB_SRC_STATE_MNT=

# Directory on host to store logs 
KRB_SRC_LOG=

# Mount location within container to store logs
KRB_LOGS_MNT=

# Master Password
KRB_MASTER_PASS=

# Default domain to use
KRB_DOMAIN=

# Default Realm KDC will be managing
KRB_REALM=

# Kerberos Server Hostname
KRB_SERVER_HOSTNAME=

# Domain name of the server hosting the admin server
KRB_ADMIN_SERVER=

# Domain name of the server hosting the KDC server
KRB_KDC_SERVER=
```

### Step 2: Run
```bash
docker-compose up
```

## License
Licensed under the [MIT License](https://github.com/rahoogan/kerberos-docker/blob/master/LICENSE)
