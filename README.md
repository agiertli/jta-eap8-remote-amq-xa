# jta-crash-rec: Example of JTA Crash Recovery

Mike Musgrove

>
> The `jta-crash-rec` quickstart uses JTA and Byteman to show how to code distributed (XA) transactions in order to preserve ACID properties on server crash.
>

## What is it?

The `jta-crash-rec` quickstart demonstrates how to code distributed or XA (eXtended Architecture) transactions so that the ACID properties are preserved across participating resources deployed to Red Hat JBoss Enterprise Application Platform after a server crash. An XA transaction is one in which multiple resources, such as MDBs and databases, participate within the same transaction. It ensures all operations are performed as a single entity of work. ACID is a set of 4 properties that guarantee the resources are processed in the following manner:

* Atomic - if any part of the transaction fails, all resources remain unchanged.

* Consistent - the state will be consistent across resources after a commit

* Isolated - the execution of the transaction for each resource is isolated from each others

* Durable - the data will persist after the transaction is committed

This quickstart shows how to atomically update multiple resources within one transaction. It updates a relational database table using JPA and sends a message using JMS. This type of paired updates to two different resources are called XA transactions and are defined by the Jakarta EE JTA specification JSR-907.

The relational database table in this example contains two columns that represent a `key` / `value` pair. The application presents an HTML form containing two input text boxes and allows you to create, update, delete or list these pairs. When you add or update a `key` / `value` pair, the quickstart starts a transaction, updates the database table, produces a JMS message containing the update, and then commits the transaction. If all goes well, eventually the consumer gets the message and generates a database update, setting the `value` corresponding to the `key` to something that indicates it was changed by the message consumer.

In this example, you halt the JBoss EAP server in the middle of an XA transaction after the database modification has been committed, but before the JMS producer is committed. You can verify that the transaction was started, then restart the JBoss EAP server to complete the transaction. You then verify that everything is in a consistent state.

JBoss EAP ships with H2, an in-memory database written in Java. In this example, we use H2 for the database. Although H2 XA support is not recommended for production systems, the example does illustrate the general steps you need to perform for any datasource vendor. This example provides its own H2 XA datasource configuration. It is defined in the `jta-crash-rec-ds.xml` file in the WEB-INF folder of the WAR archive.

## Considerations for Use in a Production Environment

H2 Database

This quickstart uses the H2 database included with Red Hat JBoss Enterprise Application Platform 7.4. It is a lightweight, relational example datasource that is used for examples only. It is not robust or scalable, is not supported, and should NOT be used in a production environment.

Datasource Configuration File

This quickstart uses a `*-ds.xml` datasource configuration file for convenience and ease of database configuration. These files are deprecated in JBoss EAP and should not be used in a production environment. Instead, you should configure the datasource using the Management CLI or Management Console. Datasource configuration is documented in the [*Configuration Guide*](https://access.redhat.com/documentation/en-us/red_hat_jboss_enterprise_application_platform/7.4/html-single/configuration_guide/).

## System Requirements

The application this project produces is designed to be run on Red Hat JBoss Enterprise Application Platform 7.4 or later.

All you need to build this project is Java 8.0 (Java SDK 1.8) or later and Maven 3.3.1 or later. See [Configure Maven to Build and Deploy the Quickstarts](https://github.com/jboss-developer/jboss-developer-shared-resources/blob/master/guides/CONFIGURE_MAVEN_JBOSS_EAP.adoc#configure_maven_to_build_and_deploy_the_quickstarts) to make sure you are configured correctly for testing the quickstarts.

## Use of the EAP\_HOME and QUICKSTART\_HOME Variables

In the following instructions, replace `*EAP\_HOME*` with the actual path to your JBoss EAP installation. The installation path is described in detail here: [Use of *EAP\_HOME* and *JBOSS\_HOME* Variables](https://github.com/jboss-developer/jboss-developer-shared-resources/blob/master/guides/USE_OF_EAP_HOME.adoc#use_of_product_home_and_jboss_home_variables).

When you see the replaceable variable *QUICKSTART\_HOME*, replace it with the path to the root directory of all of the quickstarts.

## Download and Configure Byteman

This quickstart uses *Byteman* to help demonstrate crash recovery. You can find more information about *Byteman* here: [Configure Byteman for Use with the Quickstarts](https://github.com/jboss-developer/jboss-developer-shared-resources/blob/master/guides/CONFIGURE_BYTEMAN.adoc#configure_byteman_for_use_with_the_quickstarts)

Follow the instructions here to download and configure *Byteman*: [Download and Configure Byteman]({configureBytemanDownloadDocUrl})

## Configure the Server

*NOTE*: The *Byteman* scripts only work in JTA mode. They do not work in JTS mode. If you have configured the server for a quickstart that uses JTS, you must follow the quickstart instructions to remove the JTS configuration from the JBoss EAP server before making the following changes. Otherwise *Byteman* will not halt the server.

## Start the JBoss EAP Standalone Server

1. Open a terminal and navigate to the root of the JBoss EAP directory.

2. Start the JBoss EAP server with the full profile by typing the following command.

```
$ *EAP\_HOME*/bin/standalone.sh -c standalone-full.xml
```

Configure the JBoss EAP with the remote Artemis Broker

```
$ *EAP\_HOME*/jboss-cli.sh --connect --file=configure-remote-broker.cli

```

|  |  |
| --- | --- |
| Note |
For Windows, use the `*EAP\_HOME*\bin\standalone.bat` script.
 |

## Start the Activemq Artemis Server

1. Open a terminal.

2. Start the Activemq Artemis Server by typing the following command.

```
$ podman run --rm --name artemis -p 61616:61616 -p 8161:8161 -e ANONYMOUS_LOGIN=true apache/activemq-artemis:latest-alpine
```

|  |  |
| --- | --- |
| Note |
 With docker, use `docker run --rm --name artemis -p 61616:61616 -p 8161:8161 -e ANONYMOUS_LOGIN=true apache/activemq-artemis:latest-alpine`.
  |

## Build and Deploy the Quickstart

1. Make sure you [start the JBoss EAP server](#start_the_eap_standalone_server) as described above.

2. Open a terminal and navigate to the root directory of this quickstart.

3. Type the following command to build the artifacts.

```
$ mvn clean package wildfly:deploy
```

This deploys the `jta-crash-rec/target/jta-crash-rec.war` to the running instance of the server.

You should see a message in the server log indicating that the archive deployed successfully.

## Access the Application

The application will be running at the following URL: <http://localhost:8080/jta-crash-rec/XA>.

## Test the Application

1. When you access the application, you will find a web page containing two html input boxes for adding `key` / `value` pairs to a database. Instructions for using the application are shown at the top of the application web page.

2. When you add a new `key` / `value` pair, the change is committed to the database and a JMS message sent. The message consumer then updates the newly inserted row by appending the text `updated via JMS` to the value. Since the consumer updates the row asynchronously, you may need to click *Refresh Table* to see the text added to the `key` / `value` pair you previously entered.

3. When an *XA transaction* is committed, the application server completes the transaction in two phases.

	* In phase 1 each of the resources, in this example the database and the JMS message producer, are asked to prepare to commit any changes made during the transaction.

	* If all resources vote to commit then the application server starts phase 2 in which it tells each resource to commit those changes.

	* The added complexity is to cope with failures, especially failures that occur during phase 2. Some failure modes require cooperation between the application server and the resources in order to guarantee that any pending changes are recovered.

4. To demonstrate XA recovery, you must enable the Byteman tool to terminate the application server while *phase 2* is running as follows:

	* Stop the JBoss EAP server.

	* Follow the instructions here to clear the transaction objectstore remaining from any previous tests: [Clear the Transaction ObjectStore](https://github.com/jboss-developer/jboss-developer-shared-resources/blob/master/guides/CONFIGURE_BYTEMAN.adoc#clear_the_transaction_object_store)

	* The following line of text must be appended to the server configuration file using the instructions located here: [[Use Byteman to Halt the Application](https://github.com/jboss-developer/jboss-developer-shared-resources/blob/master/guides/CONFIGURE_BYTEMAN.adoc#use_byteman_to_halt_the_application)

	```
	JAVA_OPTS="-javaagent:/*BYTEMAN\_HOME*/lib/byteman.jar=script:/*QUICKSTART\_HOME*/jta-crash-rec/src/main/scripts/xa.btm ${JAVA_OPTS}"
	```

	|  |  |
	| --- | --- |
	| Note |
	For Windows, append the following line.

	```
	`JAVA_OPTS=%JAVA_OPTS% -javaagent:C:*BYTEMAN\_HOME*\lib\byteman.jar=script:C:__QUICKSTART_HOME__\jta-crash-rec\src\main\scripts\xa.btm %JAVA_OPTS%``
	```

	 |

	* [Start the JBoss EAP server](#start_the_eap_standalone_server) with the standalone full profile as described above.

5. Once you complete step 4, you are ready to create a *recovery record*. Go to the application URL <http://localhost:8080/jta-crash-rec/XA> and insert another row into the database. At this point, Byteman halts the application server.

6. If you want to verify the database insert was committed but that message delivery is still pending, you can use an SQL client such as the H2 database console tool. Issue a query to show that the value is present but does not contain the message added by the consumer (`updated via JMS`). Here is how you can do it using H2:

	* Start the H2 console by typing:

	```
	$ java -cp *EAP\_HOME*/modules/system/layers/base/com/h2database/h2/main/h2*.jar org.h2.tools.Console
	```

	* Log in:

	```
	Database URL: jdbc:h2:file:~/jta-crash-rec-quickstart
	User name:    sa
	Password:     sa
	```

	* The console is available at the url <http://localhost:8082>. If you receive an error such as `Exception opening port "8082"` it is most likely because some other application has that port open. You will need to find which application is using the port and close it.

	* Once you are logged in enter the following query to see that the pair you entered is present but does not contain *"updated via JMS"*.

	```
	select * from kvpair
	```

	* Log out of the H2 console and make sure you close the terminal. H2 is limited to one connection and the application will need it from this point forward.

	* If you are using the default file based transaction logging store, there will be a record in the file system corresponding to the pending transaction.

		+ Open a terminal and navigate to the `*EAP\_HOME*` directory

		+ List the contents of the following directory:

		```
		$ ls *EAP\_HOME*/standalone/data/tx-object-store/ShadowNoFileLockStore/defaultStore/StateManager/BasicAction/TwoPhaseCoordinator/AtomicAction/
		```

		+ An example of a logging record file name is:

		```
		0_ffff7f000001_-7f1cf331_4f0b0ad4_15
		```

		+ After recovery, log records are normally deleted automatically. However, logs may remain in the case where the Transaction Manager ™ commit request was received and acted upon by a resource, but the TM crashed before it had time to clean up the logs of that resource.

7. To observe XA recovery

	* Stop the H2 console and exit the terminal to close the database connections. Otherwise, you may see messages like the following when you start your server:

	```
	Database may be already in use: "Locked by another process"
	```

	* [Disable the Byteman script](https://github.com/jboss-developer/jboss-developer-shared-resources/blob/master/guides/CONFIGURE_BYTEMAN.adoc#disable_the_byteman_script) by restoring the backup server configuration file.

	* [Start the JBoss EAP server](#start_the_eap_standalone_server) with the standalone full profile as described above.

	* Load the web interface to the application

	* By the time the JBoss EAP server is ready, the transaction should have recovered.

	* A message is printed on the JBoss EAP server console when the consumer has completed the update. Look for a line that reads:

	```
	JTA Crash Record Quickstart: key value pair updated via JMS
	```

	* Check that the row you inserted in step 4 now contains the text `updated via JMS`, showing that the JMS message was recovered successfully. Use the application URL to perform this check.

	* You will most likely see the following messages in the server log.

	```
	WARN  [com.arjuna.ats.jta] (Periodic Recovery) ARJUNA016037: Could not find new XAResource to use for recovering non-serializable XAResource XAResourceRecord < resource:null, txid:< formatId=131077, gtrid_length=29, bqual_length=36, tx_uid=0:ffff7f000001:1040a11d:534ede43:1c, node_name=1, branch_uid=0:ffff7f000001:1040a11d:534ede43:20, subordinatenodename=null, eis_name=java:jboss/datasources/JTACrashRecQuickstartDS >, heuristic: TwoPhaseOutcome.FINISH_OK, product: H2/1.3.168-redhat-2 (2012-07-13), jndiName: java:jboss/datasources/JTACrashRecQuickstartDS com.arjuna.ats.internal.jta.resources.arjunacore.XAResourceRecord@788f0ec1 >
	WARN  [com.arjuna.ats.jta] (Periodic Recovery) ARJUNA016038: No XAResource to recover < formatId=131077, gtrid_length=29, bqual_length=36, tx_uid=0:ffff7f000001:1040a11d:534ede43:1c, node_name=1, branch_uid=0:ffff7f000001:1040a11d:534ede43:20, subordinatenodename=null, eis_name=java:jboss/datasources/JTACrashRecQuickstartDS >
	```

	This is normal. What actually happened is that the first resource, `JTACrashRecQuickstartDS`, committed before the JBoss EAP server was halted to insert the recovery record. The transaction logs are only updated/deleted after the outcome of the transaction is determined. If the transaction manager did update the log as each participant (database and JMS queue) completed then throughput would suffer. Notice you do not get a similar message for the JMS resource since that is the resource that recovered and the log record was updated to reflect this change. You need to manually remove the record for the first participant if you know which one is which or, if you are using the community version of the ${product.name} server, then you can also inspect the transaction logs using a JMX browser. For the demo it is simplest to delete the records from the file system, however, **be wary of doing this on a production system**.

8. Do NOT forget to [Disable the Byteman script](https://github.com/jboss-developer/jboss-developer-shared-resources/blob/master/guides/CONFIGURE_BYTEMAN.adoc#disable_the_byteman_script) by restoring the backup server configuration file. The Byteman rule must be removed to ensure that your application server will be able to commit 2PC transactions!

## Server Log: Expected Warnings and Errors

You will see the following warnings in the server log. You can ignore these warnings.

```
WFLYJCA0091: -ds.xml file deployments are deprecated. Support may be removed in a future version.
HHH000431: Unable to determine H2 database version, certain features may not work
```

## Undeploy the Quickstart

When you are finished testing the quickstart, follow these steps to undeploy the archive.

1. Make sure you [start the JBoss EAP server](#start_the_eap_standalone_server) as described above.

2. Open a terminal and navigate to the root directory of this quickstart.

3. Type this command to undeploy the archive:

```
$ mvn wildfly:undeploy
```

## Run the Quickstart in Red Hat CodeReady Studio or Eclipse

You can also start the server and deploy the quickstarts or run the Arquillian tests in Red Hat CodeReady Studio or from Eclipse using JBoss tools. For general information about how to import a quickstart, add a JBoss EAP server, and build and deploy a quickstart, see [Use Red Hat CodeReady Studio or Eclipse to Run the Quickstarts](https://github.com/jboss-developer/jboss-developer-shared-resources/blob/master/guides/USE_JBDS.adoc#use_red_hat_jboss_developer_studio_or_eclipse_to_run_the_quickstarts).

|  |  |
| --- | --- |
| Note |
Within Red Hat CodeReady Studio, make sure you define a server runtime environment that uses the `standalone-full.xml` configuration file.
 |

## Debug the Application

If you want to debug the source code of any library in the project, run the following command to pull the source into your local repository. The IDE should then detect it.

```
$ mvn dependency:sources
```

## JBoss EAP for OpenShift Incompatibility

This quickstart is not compatible with JBoss EAP for OpenShift or JBoss EAP for OpenShift Online templates.

Last updated 2021-06-23 15:53:26 UTC
