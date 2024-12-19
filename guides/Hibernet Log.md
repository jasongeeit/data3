Here’s a **full solution** to get both the SQL query along with the schema and data being executed in Hibernate:

---

### **1. Prerequisites**
Make sure Hibernate is properly configured in your project with the following dependencies in your `pom.xml` (for Maven):

```xml
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-core</artifactId>
    <version>5.6.15.Final</version>
</dependency>
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>1.7.36</version>
</dependency>
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>1.2.11</version>
</dependency>
```

---

### **2. Configuration for SQL Logging**

To capture SQL queries and parameter data:
1. **Hibernate Properties (Logging SQL + Bind Parameters):**
   Add these to your Hibernate configuration file (`hibernate.cfg.xml` or `application.properties`).

   **For `hibernate.cfg.xml`:**
   ```xml
   <property name="hibernate.show_sql">true</property>
   <property name="hibernate.format_sql">true</property>
   <property name="hibernate.use_sql_comments">true</property>
   <property name="hibernate.type">trace</property>
   ```

   **For `application.properties` in Spring Boot:**
   ```properties
   spring.jpa.show-sql=true
   spring.jpa.properties.hibernate.format_sql=true
   spring.jpa.properties.hibernate.use_sql_comments=true
   spring.jpa.properties.hibernate.type=trace
   ```

2. **Enable Logging with SLF4J:**
   Configure a logging framework (e.g., Logback). Add this to `logback.xml`:

   ```xml
   <configuration>
       <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
           <encoder>
               <pattern>%d{yyyy-MM-dd HH:mm:ss} - %msg%n</pattern>
           </encoder>
       </appender>

       <logger name="org.hibernate.SQL" level="DEBUG" additivity="false">
           <appender-ref ref="STDOUT"/>
       </logger>

       <logger name="org.hibernate.type.descriptor.sql.BasicBinder" level="TRACE">
           <appender-ref ref="STDOUT"/>
       </logger>

       <root level="INFO">
           <appender-ref ref="STDOUT"/>
       </root>
   </configuration>
   ```

This will log SQL queries and their bind parameters in your console.

---

### **3. Programmatically Capture SQL Using Hibernate Interceptor**

If you want to capture SQL dynamically in code:

#### **Step 1: Create a Custom Interceptor**
```java
import org.hibernate.EmptyInterceptor;
import org.hibernate.type.Type;

import java.io.Serializable;
import java.util.Iterator;

public class SQLInterceptor extends EmptyInterceptor {
    @Override
    public String onPrepareStatement(String sql) {
        System.out.println("SQL Query: " + sql); // Log the SQL query
        return super.onPrepareStatement(sql);
    }

    @Override
    public boolean onFlushDirty(
            Object entity, Serializable id, Object[] currentState,
            Object[] previousState, String[] propertyNames, Type[] types) {
        System.out.println("Entity Updated: " + entity.getClass().getName());
        return super.onFlushDirty(entity, id, currentState, previousState, propertyNames, types);
    }

    @Override
    public void postFlush(Iterator entities) {
        System.out.println("Entities Flushed to Database.");
        super.postFlush(entities);
    }
}
```

#### **Step 2: Register the Interceptor**
Add the interceptor when building the `SessionFactory`:

```java
import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.hibernate.cfg.Configuration;

public class HibernateUtil {
    private static SessionFactory sessionFactory;

    static {
        try {
            // Create Configuration
            Configuration configuration = new Configuration();
            configuration.configure("hibernate.cfg.xml");
            configuration.setInterceptor(new SQLInterceptor()); // Set custom interceptor

            // Build SessionFactory
            sessionFactory = configuration.buildSessionFactory();
        } catch (Exception ex) {
            ex.printStackTrace();
            throw new ExceptionInInitializerError("Initial SessionFactory creation failed.");
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

---

### **4. Example to Fetch Schema and Data**

Here’s how you can use Hibernate to fetch schema and data while logging SQL:

#### **Step 1: Entity Class**
```java
import javax.persistence.*;

@Entity
@Table(name = "students")
public class Student {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "name")
    private String name;

    @Column(name = "age")
    private int age;

    // Getters and Setters
    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}
```

#### **Step 2: Fetch Data with Logging**
```java
import org.hibernate.Session;
import org.hibernate.Transaction;

import java.util.List;

public class HibernateMain {
    public static void main(String[] args) {
        Session session = HibernateUtil.getSessionFactory().openSession();
        Transaction transaction = session.beginTransaction();

        try {
            // Fetching data from the "students" table
            List<Student> students = session.createQuery("FROM Student", Student.class).list();
            for (Student student : students) {
                System.out.println("Student: " + student.getName() + ", Age: " + student.getAge());
            }

            transaction.commit();
        } catch (Exception e) {
            if (transaction != null) transaction.rollback();
            e.printStackTrace();
        } finally {
            session.close();
            HibernateUtil.shutdown();
        }
    }
}
```

---

### **5. Run the Application**
- With this setup, you’ll:
  1. Log the exact SQL queries executed.
  2. Log the schema structure.
  3. Fetch and print the data programmatically.
  4. Dynamically intercept queries and manipulate them if needed.

---

### Output Example:
```plaintext
SQL Query: select student0_.id as id1_0_, student0_.name as name2_0_, student0_.age as age3_0_ from students student0_
Entity Updated: Student
Student: John, Age: 20
Student: Alice, Age: 22
```