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

## Step to build
* Assuming the code directory is mariadb-connector-j
* Git clone https://github.com/Microsoft/mariadb-connector-j
* cd mariadb-connector-j and checkout to target branch
* mvn package, 
	If you want to build without running unit tests, use
    mvn -Dmaven.test.skip=true package
	Otherwise, the default test database names is testj, user root without password. You can also specify the connection string as follows:
	mvn -DdbUrl=jdbc:mariadb://localhost:3306/testj?user=root&password=xxx -DlogLevel=FINEST package

After that, you should have JDBC jar mariadb-java-client-x.y.z.jar in the 'target' subdirectory.

## Step to test
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