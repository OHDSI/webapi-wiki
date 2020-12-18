# Basic Security Configuration

This tutorial will demonstrate how to configure the OHDSI WebAPI and ATLAS using the OHDSI WebAPI's built in SHIRO security configuration. In this setup, we will establish a separate database to hold the user credentials for authentication which includes the user name and an **encrypted password**. While the password is encrypted, storing user credentials in a database is a potential security risk and so we only recommend this for demonstration environments. 

## Assumptions
- This tutorial assumes that you already have a working version of the OHDSI WebAPI configured and running in your environment but with security disabled.
- This tutorial assumes that you already have a working version of [ATLAS](https://github.com/OHDSI/Atlas) configured and running in your environment but with security disabled.

## settings.xml
The settings.xml file is used to configure your build of the OHDSI WebAPI in your development environment by allowing you to override the settings to the values in the settings.xml file.  You will need to make the following changes / additions to your settings.xml file in the profile you wish to use in this demonstration environment.

```xml
<security.provider>AtlasRegularSecurity</security.provider>
<security.origin>*</security.origin>
<security.ssl.enabled>true</security.ssl.enabled>
<security.maxLoginAttempts>3</security.maxLoginAttempts>
<security.duration.initial>10</security.duration.initial>
<security.duration.increment>10</security.duration.increment>
<security.db.datasource.url>jdbc:postgresql://localhost:5432/ohdsi</security.db.datasource.url>
<security.db.datasource.driverClassName>org.postgresql.Driver</security.db.datasource.driverClassName>
<security.db.datasource.schema>ohdsi</security.db.datasource.schema>
<security.db.datasource.username>ohdsi</security.db.datasource.username>
<security.db.datasource.password>ohdsi</security.db.datasource.password>
<security.db.datasource.authenticationQuery>select password,firstName,middleName,lastName from atlas_security.demo_security where email = ?</security.db.datasource.authenticationQuery>
```

## security.maxLoginAttempts

This is the maximum number of login attempts allowed before the account is locked out of the system.

## security.duration

This represents the `initial` length of lockout and the `incremental` length of lockout in seconds. So, if there are more than `security.maxLoginAttempts`, the initial lockout time will start and for every subsequent failed login, the incremental value will be added to the total lockout time.

## database 
Once you have completed the configuration of the profile for your OHDSI WebAPI you will need to create the database, schema and table that will contain our sample login information.  To start, create database (eg. `Security`), and a schema within it as `atlas_security`. 

The script to create a minimal sample table in a postgresql environment is as follows:

```sql
CREATE TABLE atlas_security.demo_security
(
    username character varying(255) COLLATE pg_catalog."default",
    password character varying(255) COLLATE pg_catalog."default",
    firstname character varying(255) COLLATE pg_catalog."default",
    middlename character varying(255) COLLATE pg_catalog."default",
    lastname character varying(255) COLLATE pg_catalog."default"
)
WITH (
    OIDS = FALSE
)
TABLESPACE pg_default;

ALTER TABLE atlas_security.demo_security
OWNER to ohdsi_app_user;

GRANT ALL ON TABLE atlas_security.demo_security TO ohdsi_app_user WITH GRANT OPTION;
```

Next you will need to insert a sample record that will contain our demonstration username and password.  The password is encrypted using BCrypt.  You can create your own username and password or use the sample insert statement provided below where we have already encrypted the password 'ohdsi' for the user named 'ohdsi'.  To create a different password hash using BCrypt you can use the following web site:

https://www.dailycred.com/article/bcrypt-calculator

And then put that password hash into the statement below.

```sql
insert into ohdsi.demo_security (email,password) 
values ('ohdsi', '$2a$04$Fg8TEiD2u/xnDzaUQFyiP.uoDu4Do/tsYkTUCWNV0zTCW3HgnbJjO')
```

If you are still facing issues after following the steps in this guide, please refer to this [WebAPI Issue #1341](https://github.com/OHDSI/WebAPI/issues/1341), [WebAPI Issue #1099](https://github.com/OHDSI/WebAPI/issues/1099) or post an issue to the [WebAPI Issue Tracker](https://github.com/OHDSI/WebAPI/issues). 

## Configuring ATLAS

Now that we have the OHDSI WebAPI configured, table created and populated we can now setup ATLAS to expect a secure OHDSI WebAPI. Please see the [[Atlas Security]] section for details.
