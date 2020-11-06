# SDI CamelJDBC Postgres SSL
Instructions to install SDI CamelJDBC adapter for Postgres with SSL support

[SAP Help page on Apache Camel JDBC](https://help.sap.com/viewer/7952ef28a6914997abc01745fef1b607/2.0_SPS05/en-US/598cdd48941a41128751892fe68393f4.html)

## Postgres database configuration
This example was tested on AWS RDS Postgres version 9.5.2, instance class db.t2.medium. 

To enforce SSL on the server, a new *parameter group* was created and assigned with parameter rds.force_ssl=1, then rebooted

## Enable Camel JDBC Adapter on Data Provisioning Agent
### Step 1
- Open <DPAgent_root>/camel/adapters.xml and add a new configuration of Camel JDBC Adapter by pasting the following code at the end of the file, but before the closing tag </Adapters>

```
<Adapter type="CamelJdbcPostgresAdapter" displayName="Camel Jdbc Postgres Adapter">
        <RemoteSourceDescription>
            <PropertyGroup name="configuration" displayName="Configuration">
                <PropertyEntry name="dbtype" displayName="Database Type" description="Database Type" defaultValue="other" isRequired="true">
                    <Choices>
                        <Choice name="postgres" displayName="Postgres"/>
                    </Choices>
                </PropertyEntry>
                <PropertyEntry name="driverClass" displayName="JDBC Driver Class" description="JDBC Driver Class" isRequired="false"/>
                <PropertyEntry name="url" displayName="JDBC URL" description="JDBC URL" isRequired="false"/>
                <PropertyGroup name="security" displayName="Security">
                    <PropertyEntry name="pds_use_agent_stored_credential" displayName="Use Agent Stored Credential" description="Use Agent Stored Credential" defaultValue="false" isRequired="false" allowAlterWhenSuspended="false">
                        <Choices>
                            <Choice name="true" displayName="true"/>
                            <Choice name="false" displayName="false"/>
                        </Choices>
                    </PropertyEntry>
                </PropertyGroup>
            </PropertyGroup>
            <CredentialProperties>
                <CredentialEntry name="db_credential" displayName="Credential" userDisplayName="user" passwordDisplayName="password">
                    <Dependencies>
                        <Dependency name="pds_use_agent_stored_credential" value="false"/>
                    </Dependencies>
                </CredentialEntry>
            </CredentialProperties>
        </RemoteSourceDescription>
        <Capabilities>
            CAP_AND_DIFFERENT_COLUMNS,
            CAP_TRUNCATE_TABLE,
            CAP_LIKE,
            CAP_IN,
            CAP_AND,
            CAP_OR,
            CAP_DISTINCT,
            CAP_HAVING,
            CAP_ORDERBY,
            CAP_ORDERBY_EXPRESSIONS,
            CAP_GROUPBY,
            CAP_SELECT,
            CAP_INSERT,
            CAP_UPDATE,
            CAP_DELETE,
            CAP_EXCEPT,
			CAP_INTERSECT,
            CAP_AGGREGATES,
            CAP_AGGREGATE_COLNAME,
            CAP_DIST_AGGREGATES,
            CAP_INSERT_SELECT,
            CAP_JOINS,
            CAP_JOINS_OUTER,
            CAP_NESTED_JOIN,
            CAP_BI_SUBSTRING,
            CAP_BI_NOW,
            CAP_BI_UPPER,
            CAP_BI_LOWER,
            CAP_BI_LCASE,
            CAP_BI_UCASE,
            CAP_BI_CONCAT,
            CAP_BI_LTRIM,
            CAP_BI_RTRIM,
            CAP_BI_TRIM,
            CAP_WHERE,
            CAP_SIMPLE_EXPR_IN_PROJ,
            CAP_EXPR_IN_PROJ,
            CAP_NESTED_FUNC_IN_PROJ,
            CAP_SIMPLE_EXPR_IN_WHERE,
            CAP_EXPR_IN_WHERE,
            CAP_NESTED_FUNC_IN_WHERE,
            CAP_SIMPLE_EXPR_IN_INNER_JOIN,
            CAP_EXPR_IN_INNER_JOIN,
            CAP_NESTED_FUNC_IN_INNER_JOIN,
            CAP_SIMPLE_EXPR_IN_LEFT_OUTER_JOIN,
            CAP_EXPR_IN_LEFT_OUTER_JOIN,
            CAP_NESTED_FUNC_IN_LEFT_OUTER_JOIN,
            CAP_SIMPLE_EXPR_IN_ORDERBY,
            CAP_EXPR_IN_ORDERBY,
            CAP_NESTED_FUNC_IN_ORDERBY,
            CAP_NONEQUAL_COMPARISON,
            CAP_OR_DIFFERENT_COLUMNS,
            CAP_PROJECT,
            CAP_BI_SECOND,
            CAP_BI_MINUTE,
            CAP_BI_HOUR,
			CAP_BI_MONTH,
            CAP_BI_YEAR,
            CAP_BI_COT,
            CAP_BI_ABS,
            CAP_BI_ACOS,
            CAP_BI_ASIN,
            CAP_BI_ATAN,
            CAP_BI_ATAN2,
            CAP_BI_CEILING,
            CAP_BI_COS,
            CAP_BI_EXP,
            CAP_BI_FLOOR,
            CAP_BI_LN,
            CAP_BI_CEIL,
            CAP_BI_LOG,
            CAP_BI_MOD,
            CAP_BI_POWER,
            CAP_BI_SIGN,
            CAP_BI_SIN,
            CAP_BI_SQRT,
            CAP_BI_TAN,
            CAP_BI_ROUND,
            CAP_BI_ASCII,
            CAP_BI_RIGHT,
            CAP_BI_LEFT,
            CAP_BI_TO_BIGINT,
            CAP_BI_TO_DECIMAL,
            CAP_BI_TO_DOUBLE,
            CAP_BI_TO_REAL,
            CAP_BI_TO_SMALLINT,
            CAP_BI_TO_INT,
            CAP_BI_TO_INTEGER
            CAP_BI_COALESCE,
            CAP_BI_IFNULL,
            CAP_BI_NULLIF,
            CAP_BIGINT_BIND
        </Capabilities>
        <RouteTemplate>jdbc-general.xml</RouteTemplate>
    </Adapter>
```

### Step 2
- Download the Postgres JDBC driver, and copy it to the <DPAgent_root>/camel/lib directory. In this example, 
we downloaded the latest version available for JDBC 4.2 (42.2.18) at https://jdbc.postgresql.org/download.html
- Allow the JAR file to be executed: chmod u=rx postgresql-42.2.18.jar
- Reboot the DP agent for the adapters.xml file from step 1 to be registered
- Deploy the CamelJDBC adapter: 
-	<DPAgent_root>/bin/agentcli.sh --configAgent
-	Register Adapter > CamelJdbcAdapter

## Register the adapter
Register the adapter either through the DP Agent client, or using the following statement in HANA:
CREATE ADAPTER "CamelJdbcPostgresAdapter" AT LOCATION AGENT "<AGENT_NAME>";

## Create Remote Source on HANA
- Use the remote source UI (but then make sure to add the SSL entry to the JDBC string) or the below SQL statement
- Replace <AGENT_NAME>, <HOST_NAME>, <PORT>, <DATABASE_NAME>, <USER_NAME>, <PASS_WORD>
- In the URL tag, the SSL setting is set to sslmode=require. Change this to another mode if needed. The different modes are described [here](https://jdbc.postgresql.org/documentation/head/ssl-client.html)

```
--DROP REMOTE SOURCE "RS_POSTGRES_CAMEL";
CREATE REMOTE SOURCE "RS_POSTGRES_CAMEL" 
	ADAPTER "CamelJdbcPostgresAdapter" AT LOCATION AGENT "<AGENT_NAME>"
		CONFIGURATION '<?xml version="1.0" encoding="UTF-8"?>
<ConnectionProperties name="configuration">
  <PropertyEntry name="dbtype">postgres</PropertyEntry>
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
	"name" varchar(64) NULL,
	primary key (id)
);
INSERT INTO public.t1 VALUES (0, 'This is a test');
```

Create a virtual table in HANA and query it
```
--DROP TABLE "DBADMIN"."V_T1";
CREATE VIRTUAL TABLE "DBADMIN"."V_T1" at "RS_POSTGRES_CAMEL"."<NULL>"."<NULL>"."public.t1";
SELECT * FROM "DBADMIN"."V_T1";
```
