<p align="center">
  <a href="http://mariadb.org/">
    <img src="https://mariadb.com/themes/custom/mariadb/logo.svg">
  </a>
</p>

# MariaDB java connector Build Guide

For general introduction for MariaDB java connector, please check on  https://github.com/MariaDB/mariadb-connector-j .
The source code here is in the latest version same with the master branch on https://github.com/MariaDB/mariadb-connector-j with redirection feature support. 
Following is a brief guide of how to build and test the driver. 

## MariaDB version
* The source code here is in the latest version same with the master branch on https://github.com/MariaDB/mariadb-connector-j with redirection feature support.

## Tools prerequisite to build the drivers
* Java 1.8+
* Maven

## Step to build
* Assuming the code directory is  mariadb-connector-j
* Git clone https://github.com/Microsoft/mariadb-connector-j
* cd mariadb-connector-j
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
			"&enableRedirect=true";

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