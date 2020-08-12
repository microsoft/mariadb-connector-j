<p align="center">
  <a href="http://mariadb.org/">
    <img src="https://mariadb.com/themes/custom/mariadb/logo.svg">
  </a>
</p>

# MariaDB java connector Build Guide

For general introduction for MariaDB java connector, please check on  https://github.com/MariaDB/mariadb-connector-j .
The source code here is in the latest version same with the related branch on https://github.com/MariaDB/mariadb-connector-j with redirection feature support. 
Following is a brief guide of how to build and test the driver. 

## MariaDB version
The source code here is in the latest status same with the realted branch on https://github.com/MariaDB/mariadb-connector-j with redirection feature support(2020-06-03).
Valid branches:
* master: based on MariaDB/mariadb-connector-j branch master with redirection support.
* 2.6.0-with-redirection: based on MariaDB/mariadb-connector-j tag 2.6.0 with redirection support.
* 2.5.1-with-redirection: based on MariaDB/mariadb-connector-j tag 2.5.1 with redirection support.

**Notice** There is an issue traced for MariaDB/mariadb-connector-j 2.5.2+ on https://jira.mariadb.org/browse/CONJ-807. The recommended branch is 2.5.1-with-redirection.

## Redirection option Usage
A new connection option is introduced for redirection and the option name is **enableRedirect**, default value: **off**.

The detailed usage of the option enableRedirect is as follows:
<table>
<tr>
<td>off(0)</td>
<td> - Redirection will not be used.</td>
</tr>
<tr>
<td>on(1)</td>
<td>
      Will enforce redirection. <br/>
      - If redirection is not supported on the server, the connection will be aborted and the following error is returned: <i>"Connection aborted because redirection is not enabled on the MySQL server or the network package doesn't meet redirection protocol."</i></br>
      - If the MySQL server supports redirection, but the redirected connection failed for any reason, also abort the first proxy connection. Return the error of the redirected connection.
</td> 
</tr>
<tr>
<td>
preferred(2)
</td>
<td>  - It will use redirection if possible.</br>
      - If the connection does not use SSL on the driver side, the server does not support redirection, or the redirected connection fails to connect for any non-fatal reason while the proxy connection is still a valid one, it will fall back to the first proxy connection.
</td> 
</tr>
</table>


## Tools prerequisite to build the drivers
* Java 1.8+
* Maven

## Step to build and install
### Build
* Assuming the code directory is mariadb-connector-j
* Git clone https://github.com/Microsoft/mariadb-connector-j .
* cd mariadb-connector-j and checkout to target branch
* Use following command to build and run default unit test:
``` 
mvn package
```
The default test database names is testj which need to be created ahead, user is root, and without password. You can also specify the connection string as follows for test:
```
mvn -DdbUrl=jdbc:mariadb://localhost:3306/testj?user=root&password=xxx -DlogLevel=FINEST package
```
Please notice that the unit test sets is not designed fully compatibitale with Azure MySQL server, so don't run this test against Azure MySQL server and expect a full pass.

If you want to build without running unit tests and document check, use:
```
mvn -Dmaven.javadoc.skip=true -Dmaven.test.skip=true package
```

### Install
After build, you should have JDBC jar mariadb-java-client-x.y.z.jar in the 'target' subdirectory, e.g. mariadb-java-client-2.5.1.jar. Replace this jar with the mariadb-java-client jar package you currently used in your environemnt. Following are two use examples in different scenario:
* Jmeter test: Put the jar or replace the jar package in Jemter's lib directory, e.g. apache-jmeter-5.2.1\lib\
* A basic JDBC Java project: You may follow http://www.ccs.neu.edu/home/kathleen/classes/cs3200/JDBCtutorial.pdf to setup a basic JDBC Java project, add the jar path in Build Path - Libraries - Add External JARS.

After install, specify the connection string setting with enableRedirect option, e.g. 
```
jdbc:mysql://xxx.mysql.database.azure.com/testj/?user=xx&password=xx&useSSL=true&serverSslCert=xx/BaltimoreCyberTrustRoot.crt.pem&enableRedirect=on"
```
**Notice:** Please notice that there is a limitation for Azure DB for MySQL where redirection is only possible when the connection is configured with SSL and only works with TLS 1.2 with a FIPS approved cipher for redirection.


## Test
* The pdf document http://www.ccs.neu.edu/home/kathleen/classes/cs3200/JDBCtutorial.pdf is a step by step guide to use JDBC with Eclipse. 
The difference here is that in step 2, you do not need to go to http://www.mysql.com/downloads/connector/j/ to download the jar, but using the one built in the ‘target’. After setup the environment,  following is a version of sample code to test "SELECT 1" performance.

```java
    // Load the JDBC driver
    Class.forName("org.mariadb.jdbc.Driver");
    System.out.println("Driver loaded");

    int count = 10;
    String query = "SELECT 1";
    int i=0;
    // Try to connect
    String url = "jdbc:mysql://xxx.mysql.database.azure.com"+
            "?verifyServerCertificate=false"+
            "&useSSL=true"+
            "&requireSSL=true" +
			"&enableRedirect=on";

    Connection connection = DriverManager.getConnection (url, "username", "password");
    double t1 = System.nanoTime();
    Statement s1 = connection.createStatement();
    for(i=0;i<count;i++)
    {
        s1.executeQuery(query);
    }
    double t2 = System.nanoTime();
    System.out.println (" time = " + (t2 - t1)/count/1000000);
    connection.close();
```