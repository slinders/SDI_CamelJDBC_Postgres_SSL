# SDI CamelJDBC Postgres SSL
Instructions to install SDI CamelJDBC adapter for Postgres with SSL support

[SAP Help page on Apache Camel JDBC](https://help.sap.com/viewer/7952ef28a6914997abc01745fef1b607/2.0_SPS05/en-US/598cdd48941a41128751892fe68393f4.html)

## Postgres database configuration
This example was tested on AWS RDS Postgres version 9.5.2, instance class db.t2.medium. 

To enforce SSL on the server, a new *parameter group* was created and assigned with parameter rds.force_ssl=1, then rebooted

## Enable Camel JDBC Adapter on Data Provisioning Agent
- Open <DPAgent_root>/camel/adapters.xml and uncomment the configuration of Camel JDBC Adapter.
- Download the Postgres JDBC driver, and copy it to the <DPAgent_root>/camel/lib directory. In this example, 
we downloaded the latest version available for JDBC 4.2 (42.2.18) at https://jdbc.postgresql.org/download.html
- Allow the JAR file to be executed: chmod u=rx postgresql-42.2.18.jar
- Deploy the CamelJDBC adapter 
-	<DPAgent_root>/bin/agentcli.sh --configAgent
-	Register Adapter > CamelJdbcAdapter

## Create Remote Source on HANA
- Replace <AGENT_NAME>, <HOST_NAME>, <PORT>, <DATABASE_NAME>, <USER_NAME>, <PASS_WORD>
- In the URL tag, the SSL setting is set to sslmode=require. Change this to another mode if needed. The different modes are described [here](https://jdbc.postgresql.org/documentation/head/ssl-client.html)

```
--DROP REMOTE SOURCE "RS_POSTGRES_CAMEL";
CREATE REMOTE SOURCE "RS_POSTGRES_CAMEL" 
	ADAPTER "CamelJdbcAdapter" AT LOCATION AGENT "<AGENT_NAME>"
		CONFIGURATION '<?xml version="1.0" encoding="UTF-8"?>
<ConnectionProperties name="configuration">
  <PropertyEntry name="dbtype">other</PropertyEntry>
  <PropertyEntry name="filepath"></PropertyEntry>
  <PropertyEntry name="filename"></PropertyEntry>
  <PropertyEntry name="host"></PropertyEntry>
  <PropertyEntry name="port"></PropertyEntry>
  <PropertyEntry name="dbname"></PropertyEntry>
  <PropertyEntry name="servername"></PropertyEntry>
  <PropertyEntry name="delimident">false</PropertyEntry>
  <PropertyEntry name="driverClass">org.postgresql.Driver</PropertyEntry>
  <PropertyEntry name="url">jdbc:postgresql://<HOST_NAME>:<PORT>/<DATABASE_NAME>?sslmode=require</PropertyEntry>
  <PropertyGroup name="security">
      <PropertyEntry name="pds_use_agent_stored_credential">false</PropertyEntry>
  </PropertyGroup>
</ConnectionProperties>
'
WITH CREDENTIAL TYPE 'PASSWORD'
USING '<CredentialEntry name="db_credential"><user><USER_NAME></user>
<password><PASS_WORD></password></CredentialEntry>';
```

## Test replication
Create a table in Postgres
```
--DROP TABLE public.t1;
CREATE TABLE public.t1 (
	id int4 NULL,
	"name" varchar(64) NULL
);
INSERT INTO public.t1 VALUES (0, 'This is a test');
```

Create a virtual table in HANA and query it
```
--DROP TABLE "DBADMIN"."V_T1";
CREATE VIRTUAL TABLE "DBADMIN"."V_T1" at "RS_POSTGRES_CAMEL"."<NULL>"."<NULL>"."public.t1";
SELECT * FROM "DBADMIN"."V_T1";
```
