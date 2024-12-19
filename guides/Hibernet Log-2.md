To achieve this objective, we will create a Java application that uses Hibernate to connect to a MySQL or Oracle database and retrieve the currently running SQL queries. Here are the steps and the complete code example:

### Steps:
1. **Setup Hibernate Configuration:**
   - Create a Hibernate configuration file (`hibernate.cfg.xml`) for MySQL or Oracle.
   
2. **Define Entity Classes:**
   - Create entity classes if needed (not required for this specific task).

3. **Create a Utility Class for Hibernate Session Management:**

4. **Create a Method to Retrieve Running SQL Queries:**
   - Use native SQL queries to get the currently running queries from the database.

### 1. Hibernate Configuration

Create `hibernate.cfg.xml` in your `src/main/resources` directory for MySQL:

```xml
<!-- MySQL Configuration -->
<!DOCTYPE hibernate-configuration PUBLIC "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
"http://hibernate.sourceforge.net/hibernate-configuration-3.0.dtd">
<hibernate-configuration>
    <session-factory>
        <property name="hibernate.dialect">org.hibernate.dialect.MySQLDialect</property>
        <property name="hibernate.connection.driver_class">com.mysql.cj.jdbc.Driver</property>
        <property name="hibernate.connection.url">jdbc:mysql://localhost:3306/yourdatabase</property>
        <property name="hibernate.connection.username">yourusername</property>
        <property name="hibernate.connection.password">yourpassword</property>
        <property name="hibernate.hbm2ddl.auto">update</property>
        <property name="hibernate.show_sql">true</property>
        <property name="hibernate.format_sql">true</property>
    </session-factory>
</hibernate-configuration>
```

Or for Oracle:

```xml
<!-- Oracle Configuration -->
<!DOCTYPE hibernate-configuration PUBLIC "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
"http://hibernate.sourceforge.net/hibernate-configuration-3.0.dtd">
<hibernate-configuration>
    <session-factory>
        <property name="hibernate.dialect">org.hibernate.dialect.OracleDialect</property>
        <property name="hibernate.connection.driver_class">oracle.jdbc.driver.OracleDriver</property>
        <property name="hibernate.connection.url">jdbc:oracle:thin:@localhost:1521:xe</property>
        <property name="hibernate.connection.username">yourusername</property>
        <property name="hibernate.connection.password">yourpassword</property>
        <property name="hibernate.hbm2ddl.auto">update</property>
        <property name="hibernate.show_sql">true</property>
        <property name="hibernate.format_sql">true</property>
    </session-factory>
</hibernate-configuration>
```

### 2. Create a Utility Class for Hibernate Session Management

Create a class `HibernateUtil` in your `src/main/java` directory:

```java
import org.hibernate.SessionFactory;
import org.hibernate.cfg.Configuration;

public class HibernateUtil {
    private static final SessionFactory sessionFactory = buildSessionFactory();

    private static SessionFactory buildSessionFactory() {
        try {
            return new Configuration().configure().buildSessionFactory();
        } catch (Throwable ex) {
            throw new ExceptionInInitializerError(ex);
        }
    }

    public static SessionFactory getSessionFactory() {
        return sessionFactory;
    }

    public static void shutdown() {
        getSessionFactory().close();
    }
}
```

### 3. Create a Method to Retrieve Running SQL Queries

Create a class `RunningQueriesRetriever` in your `src/main/java` directory:

```java
import org.hibernate.Session;
import org.hibernate.Transaction;
import org.hibernate.query.NativeQuery;
import java.util.List;

public class RunningQueriesRetriever {

    public static List<Object[]> getRunningQueriesMySQL() {
        Session session = HibernateUtil.getSessionFactory().openSession();
        Transaction transaction = null;
        List<Object[]> runningQueries = null;

        try {
            transaction = session.beginTransaction();
            String sql = "SHOW FULL PROCESSLIST";
            NativeQuery query = session.createNativeQuery(sql);
            runningQueries = query.list();
            transaction.commit();
        } catch (Exception e) {
            if (transaction != null) {
                transaction.rollback();
            }
            e.printStackTrace();
        } finally {
            session.close();
        }

        return runningQueries;
    }

    public static List<Object[]> getRunningQueriesOracle() {
        Session session = HibernateUtil.getSessionFactory().openSession();
        Transaction transaction = null;
        List<Object[]> runningQueries = null;

        try {
            transaction = session.beginTransaction();
            String sql = "SELECT sql_id, sql_text FROM v$sql WHERE command_type = 3";
            NativeQuery query = session.createNativeQuery(sql);
            runningQueries = query.list();
            transaction.commit();
        } catch (Exception e) {
            if (transaction != null) {
                transaction.rollback();
            }
            e.printStackTrace();
        } finally {
            session.close();
        }

        return runningQueries;
    }

    public static void main(String[] args) {
        // MySQL Example
        List<Object[]> runningQueriesMySQL = getRunningQueriesMySQL();
        for (Object[] row : runningQueriesMySQL) {
            System.out.println("User: " + row[0] + ", Host: " + row[1] + ", DB: " + row[2] + ", Command: " + row[3] + ", Time: " + row[4] + ", State: " + row[5] + ", Info: " + row[6]);
        }

        // Oracle Example
        List<Object[]> runningQueriesOracle = getRunningQueriesOracle();
        for (Object[] row : runningQueriesOracle) {
            System.out.println("SQL ID: " + row[0] + ", SQL Text: " + row[1]);
        }
    }
}
```

### Important Notes:
- Ensure you have the appropriate JDBC drivers for MySQL or Oracle in your classpath.
- The MySQL query uses `SHOW FULL PROCESSLIST` to fetch the running queries.
- The Oracle query uses `v$sql` to fetch the running queries.
- You may need to adjust the SQL queries based on your specific database schema and requirements.

This code provides a complete example of how to connect to a MySQL or Oracle database using Hibernate and retrieve the currently running SQL queries. You can run the `RunningQueriesRetriever` class to see the output.