FROM library/centos
MAINTAINER raghavanu@gmail.com

# Mandatory Argument: Master Password for KDC 
ARG KRB_MASTER_PASS

# Mandatory Argument: Kerberos Realm Name managed by KDC
ARG KRB_REALM

# Mandatory Argument: Domain name of Kerberos Admin Server
ARG KRB_ADMIN_SERVER

# Mandatory Argument: Domain name of Kerberos KDC Server
ARG KRB_KDC_SERVER

# Mandatory Argument: Default Kerberos Domain Name (Should be same as Realm, but lowercase)
ARG KRB_DOMAIN

# Mandatory Arguments: Source conf file locations
ARG KRB_SOURCE_CONF
ARG KRB_KDC_SOURCE_CONF

# Mandatory Arguments: Mount points for docker volumes
ARG KRB_LOGS_MNT
ARG KRB_SRC_STATE_MNT
ARG KRB_KEYTAB_MNT

# Optional Argument: Container location of conf files
ARG KRB_CONF_LOC=/etc/krb5
ARG KRB_KEYTAB_LOC=/etc/security
ARG KRB_STATE_LOC=/var/kerberos

# Optional Argument: Log Directory for KDC Logs
ARG KRB_LOG_DIR=/var/log/krb5kdc
ARG KRB_KDC_LOG=/var/log/krb5kdc/kdc.log
ARG KRB_KADMIN_LOG=/var/log/krb5kdc/kadmin.log

# Optional Argument: Kerberos Server Yum Package Names & Versions
ARG KRB_PKG=krb5-server-1.15.1-19.el7
ARG KRB_LDAP_PKG=krb5-server-ldap-1.15.1-19.el7

# Optional Argument: Kerberos Default Ticket Renew Lifetime
ARG KRB_TICKET_RENEW_LIFETIME="6h"

# Optional Argument: Kerberos Default Ticket Lifetime
ARG KRB_TICKET_LIFETIME="1d"

# Optional Argument: Entrypoint Script
ARG KRB_ENTRYPOINT=entrypoint.sh

# Set Environment Variables
ENV KRB_REALM=$KRB_REALM
ENV KRB_TICKET_RENEW_LIFETIME=$KRB_TICKET_RENEW_LIFETIME
ENV KRB_TICKET_LIFETIME=$KRB_TICKET_LIFETIME
ENV KRB_ADMIN_SERVER=$KRB_ADMIN_SERVER
ENV KRB_KDC_SERVER=$KRB_KDC_SERVER
ENV KRB_DOMAIN=$KRB_DOMAIN
ENV KRB_ADMIN_SERVER=$KRB_ADMIN_SERVER
ENV KRB5_KDC_PROFILE=/var/kerberos/krb5kdc
ENV KRB_KDC_LOG=$KRB_KDC_LOG
ENV KRB_KADMIN_LOG=$KRB_KADMIN_LOG
ENV KRB_SOURCE_CONF=$KRB_SOURCE_CONF
ENV KRB_CONF_LOC=$KRB_CONF_LOC
ENV KRB_MASTER_PASS=$KRB_MASTER_PASS
ENV KRB_LOG_DIR=$KRB_LOG_DIR

# Volumes
## Create Conf Locations
RUN mkdir -p $KRB_CONF_LOC

## Create Mount Points
RUN mkdir -p $KRB_LOGS_MNT && mkdir -p $KRB_SRC_STATE_MNT && mkdir -p $KRB_KEYTAB_MNT

## Create symlinks to docker volumes
RUN ln -s $KRB_LOGS_MNT /var/log
RUN ln -s $KRB_KEYTAB_MNT $KRB_KEYTAB_LOC
RUN ln -s $KRB_SRC_STATE_MNT $KRB_STATE_LOC

# Expose Ports
## KDC Server Port
EXPOSE 88
## Kadmin Server Port 
EXPOSE 749

# Update and Install Packages
RUN yum update -y && \
    yum install -y $KRB_PKG $KRB_LDAP_PKG expect chrony

# Setup/Copy config files

## Remove default config
RUN rm /etc/krb5.conf

## Copy config files from source/host
COPY $KRB_SOURCE_CONF $KRB_CONF_LOC/krb5.conf
COPY $KRB_KDC_SOURCE_CONF $KRB_CONF_LOC/kdc.conf

## Create Expect Script for Kerberos DB Install
RUN echo $'\
#!/usr/bin/env expect\n\
set timeout -1\n\
spawn kdb5_util create -r $KRB_REALM -s\n\
expect "Enter KDC database master key: "\n\
send -- "$KRB_MASTER_PASS\\n"\n\
expect "Re-enter KDC database master key to verify: "\n\
send -- "$KRB_MASTER_PASS\\n"\n\
expect eof\n\
' > /krb-install.expect
RUN sed -i -e "s/\$KRB_REALM/$KRB_REALM/" \
           -e "s/\$KRB_MASTER_PASS/$(printf '%s\n' "$KRB_MASTER_PASS" | sed 's/[[\.*^$/]/\\&/g')/" \
           krb-install.expect

## Replace Environment Variables
## TODO: Replacing variables is always messy - especially when dealing with escape characters. Here we use sed, and assume no special chars are going to be used, except for variables specifyig filepaths
RUN sed -i -e "s/\$KRB_REALM/$KRB_REALM/" \
           -e "s/\$KRB_DOMAIN/$KRB_DOMAIN/" \
           -e "s/\$KRB_KDC_SERVER/$KRB_KDC_SERVER/" \
           -e "s/\$KRB_ADMIN_SERVER/$KRB_ADMIN_SERVER/" \
           -e "s/\$KRB_TICKET_LIFETIME/$KRB_TICKET_LIFETIME/" \
           -e "s/\$KRB_TICKET_RENEW_LIFETIME/$KRB_TICKET_RENEW_LIFETIME/" \
           -e "s/\$KRB_KDC_LOG/$(printf '%s\n' "$KRB_KDC_LOG" | sed 's/[[\.*^$/]/\\&/g')/" \
           -e "s/\$KRB_KADMIN_LOG/$(printf '%s\n' "$KRB_KADMIN_LOG" | sed 's/[[\.*^$/]/\\&/g')/" \
           /etc/krb5/krb5.conf

## Create symlink to location where kdc expects conf file
RUN ln -s $KRB_CONF_LOC/krb5.conf /etc/krb5.conf

# Create Kerberos Database
RUN expect krb-install.expect

# Clean up installation files and variables
RUN rm /krb-install.expect
ENV KRB_MASTER_PASS=""

# Run KDC and Kadmin Servers
ENTRYPOINT krb5kdc && kadmind && tail -f ${KRB_LOG_DIR}/*
