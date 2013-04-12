# Welcome to Tomcat7 on CloudBees

This is a "ClickStart" that gets you going with a Maven - Tomcat 7 "seed" project starting point. You can launch it here:

<a href="https://grandcentral.cloudbees.com/?CB_clickstart=https://raw.github.com/CloudBees-community/tomcat7-maven-clickstart/master/clickstart.json"><img src="https://d3ko533tu1ozfq.cloudfront.net/clickstart/deployInstantly.png"/></a>

This will setup a continuous deployment pipeline - a CloudBees Git repository, a Jenkins build compiling and running the test suite (on each commit).
Should the build succeed, this seed app is deployed on a Tomcat 7 container.

# CloudBees Tomcat 7 container

Tomcat 7 container is available on CloudBees thanks to the [tomcat7-clickstack](https://github.com/CloudBees-community/tomcat7-clickstack). Documentation is available [here](https://developer.cloudbees.com/bin/view/RUN/Tomcat7).

# How to deploy a web application on a Tomcat7 ClickStack

You can deploy your web application on the tomcat7 clickstack using the [CloudBees SDK](https://developer.cloudbees.com/bin/view/RUN/BeesSDK) "`app:deploy`" command.

```
bees app:deploy -a my-account/tomcat7-maven-clickstart -t tomcat7 -Rjava_version=1.7 ./target/tomcat7-maven-clickstart-1.0-SNAPSHOT.war
```

* "`-a my-account/tomcat7-maven-clickstart`": name of the CloudBees account and of the application. The application will be accessible on the URL http://tomcat7-maven-clickstart.cyrille-leclerc.cloudbees.net/
* "`-t tomcat7`": identifier of the tomcat7 clickstack
* "`-Rjava_version=1.7`": optional parameter to use the version 7 of the Java runtime (JVM). Tomcat 7 supports both JVM 6 and 7.
* "`./target/tomcat7-maven-clickstart-1.0-SNAPSHOT.war`": path to the war file.
You only need to set the "`-R`", "`-t`" and "`-D`" settings once - they will be remembered for subsequent deploys.

# How to bind a CloudBees MySql database to an application on a Tomcat7 ClickStack

## Create database if needed
```
db:create --username my-username --password alpha-beta tomcat7-maven-clickstart-db
```

## Bind application to database

```
bees app:bind -a  tomcat7-maven-clickstart -db tomcat7-maven-clickstart-db -as tomcat7_maven_clickstart_db
```
* "`-a  tomcat7-maven-clickstart`": the name of your application
* "`-db tomcat7-maven-clickstart-db`": the name of your CloudBees MySQL Database
* "`-as tomcat7_maven_clickstart_db`": the name of the binding which is used to identify the binding and to compose the name of the environment variables used to describe this binding (always prefer '_' to '-' for bindings because '-' is not supported in linux environment variable names).

This binding will create the following System Properties:

* `DATABASE_URL_TOMCAT7_MAVEN_CLICKSTART_DB`: url of the database starting with "mysql:" (e.g. "mysql://ec2-1.2.3.4.compute-1.amazonaws.com:3306/tomcat7-maven-clickstart-db"). **Please note** that this URL is **not** prefixed by "jdbc:".
* `DATABASE_USERNAME_TOMCAT7_MAVEN_CLICKSTART_DB`: login of the database
* `DATABASE_PASSWORD_BEES_TOMCAT7_MAVEN_CLICKSTART_DB`: password of the database

Details on bindings are available in [Binding services (resources) to applications](https://developer.cloudbees.com/bin/view/RUN/Resource+Management).

## Declare Tomcat JNDI Datasource

Then, in your war application, declare a standard [Tomcat JNDI DataSource](http://tomcat.apache.org/tomcat-7.0-doc/jndi-datasource-examples-howto.html) a "META-INF/context.xml" file using [Tomcat variable substitution](http://tomcat.apache.org/tomcat-7.0-doc/config/index.html) syntax `${propname}` to inject your binding parameters.

```
<Context>
    <Resource
            name="jdbc/my-db"
            auth="Container"
            type="javax.sql.DataSource"

            url="jdbc:${DATABASE_URL_TOMCAT7_MAVEN_CLICKSTART_DB}"
            username="${DATABASE_USERNAME_TOMCAT7_MAVEN_CLICKSTART_DB}"
            password="${DATABASE_PASSWORD_BEES_TOMCAT7_MAVEN_CLICKSTART_DB}"

            driverClassName="com.mysql.jdbc.Driver"

            maxActive="20"
            maxIdle="1"
            maxWait="10000"
            removeAbandoned="true"
            removeAbandonedTimeout="60"
            logAbandoned="true"

            validationQuery="SELECT 1"
            testOnBorrow="true"
            />
</Context>
```
See https://github.com/CloudBees-community/tomcat7-maven-clickstart/blob/master/src/main/webapp/META-INF/context.xml

### Use the DataSource in you application

You can now use your "`java:comp/env/jdbc/my-db`" JNDI DataSource in your application.
Code sample

```
Context ctx = new InitialContext();
DataSource ds = (DataSource) ctx.lookup("java:comp/env/jdbc/my-db");
Connection conn = ds.getConnection();
ResultSet rst = stmt.executeQuery("select 1");
while (rst.next()) {
    out.print("resultset result: " + rst.getString(1));
}
rst.close();
stmt.close();
conn.close();
```




 




