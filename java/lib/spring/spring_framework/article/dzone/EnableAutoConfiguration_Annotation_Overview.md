# [@EnableAutoConfiguration Annotation Overview](https://dzone.com/articles/how-springboot-autoconfiguration-magic-works)

- [@EnableAutoConfiguration Annotation Overview](#enableautoconfiguration-annotation-overview)
  - [Exploring the Power of @Conditional](#exploring-the-power-of-conditional)
  - [Spring Boot AutoConfiguration](#spring-boot-autoconfiguration)

## Exploring the Power of @Conditional 

While developing Spring based applications we may come across a need to **register beans conditionally**.

For example, you may want to register a `DataSource` bean pointing to the dev database while running application locally and point to a different production database while running in production.

You can externalize the database connection parameters into the properties file and use the file appropriate for the environment, but you need to change the configuration whenever you need to point to a different environment and build the application.

To address this problem, Spring 3.1 introduced the concept of **Profiles**. You can register multiple beans of the same type and associate them with one or more profiles. When you run the application you can activate the desired profiles and beans associated with the activated profiles, and only those profiles will be registered.

    @Configuration
    public class AppConfig
    {
        @Bean
        @Profile("DEV")
        public DataSource devDataSource() {
            ...
        }
    ​
        @Bean
        @Profile("PROD")
        public DataSource prodDataSource() {
            ...
        }
    }

Then you can specify the active profile using System Property `-Dspring.profiles.active=DEV`.

This approach works for simple cases like enable or disable bean registrations based on activated profiles. But if you want to register beans based on some **conditional logic** then the profiles approach itself is not sufficient.

To provide more flexibility for registering Spring beans conditionally, Spring 4 introduced the concept of `@Conditional`. By using the `@Conditional` approach you can register a bean conditionally based on any arbitrary condition.

For example, you may want to register a bean when:

- A specific class is present in classpath
- A Spring bean of certain type doesn’t already registered in ApplicationContext
- A specific file exists on a location
- A specific property value is configured in a configuration file
- A specific system property is present/absent

These are just a few examples only and you can have any condition you want.

Let us take a look at how Spring’s @Conditional works.

Suppose we have a `UserDAO` interface with methods to get data from a data store. We have two implements of `UserDAO` interface namely `JdbcUserDAO` which talks to `MySQL` database and `MongoUserDAO` which talks to `MongoDB`.

We may want to enable only one interface of `JdbcUserDAO` and `MongoUserDAO` based on a System Property, say `dbType`.

If the application is started using `java -jar myapp.jar -DdbType=MySQL`, then we want to enable `JdbcUserDAO`. Otherwise, if the application is started using `java -jar myapp.jar -DdbType=MONGO`, we want to enable `MongoUserDAO`.

Suppose we have a `UserDAO` bean and a `JdbcUserDAO` bean. The `MongoUserDAO` implementation is as follows:

    public interface UserDAO
    {
        List<String> getAllUserNames();
    }
    ​
    public class JdbcUserDAO implements UserDAO
    {
        @Override
        public List<String> getAllUserNames()
        {
            System.out.println("**** Getting usernames from RDBMS *****");
            return Arrays.asList("Siva","Prasad","Reddy");
        }
    }
    ​
    public class MongoUserDAO implements UserDAO
    {
        @Override
        public List<String> getAllUserNames()
        {
            System.out.println("**** Getting usernames from MongoDB *****");
            return Arrays.asList("Bond","James","Bond");
        }
    }

We can implement the Condition `MySQLDatabaseTypeCondition` to check whether the System Property `dbType` is "`MYSQL`" as follows:

public class MySQLDatabaseTypeCondition implements Condition
{
    @Override
    public boolean matches(ConditionContext conditionContext, AnnotatedTypeMetadata metadata)
    {
        String enabledDBType = System.getProperty("dbType");
        return (enabledDBType != null && enabledDBType.equalsIgnoreCase("MYSQL"));
    }
}

We can implement the Condition `MongoDBDatabaseTypeCondition` to check whether the System Property `dbType` is "`MONGODB`" as follows:

    public class MongoDBDatabaseTypeCondition implements Condition
    {
        @Override
        public boolean matches(ConditionContext conditionContext, AnnotatedTypeMetadata metadata)
        {
            String enabledDBType = System.getProperty("dbType");
            return (enabledDBType != null && enabledDBType.equalsIgnoreCase("MONGODB"));
        }
    }

Now we can configure both `JdbcUserDAO` and `MongoUserDAO` beans conditionally using `@Conditional` as follows:

    @Configuration
    public class AppConfig
    {
        @Bean
        @Conditional(MySQLDatabaseTypeCondition.class)
        public UserDAO jdbcUserDAO(){
            return new JdbcUserDAO();
        }
    ​
        @Bean
        @Conditional(MongoDBDatabaseTypeCondition.class)
        public UserDAO mongoUserDAO(){
            return new MongoUserDAO();
        }
    }

If we run the application like `java -jar myapp.jar -DdbType=MYSQL` then only the `JdbcUserDAO` bean will be registered. But if you set the System property like `-DdbType=MONGODB` then only the `MongoUserDAO` bean will be registered.

Now that we have seen how to conditionally register a bean based on System Property.

Suppose we want to register `MongoUserDAO` bean only when `MongoDB` java driver class "`com.mongodb.Server`" is available on classpath, if not we want to register `JdbcUserDAO` bean.

To accomplish that we can create Conditions to check the presence or absence of MongoDB driver class "`com.mongodb.Server`" as follows:

    public class MongoDriverPresentsCondition implements Condition
    {
        @Override
        public boolean matches(ConditionContext conditionContext,AnnotatedTypeMetadata metadata)
        {
            try {
                Class.forName("com.mongodb.Server");
                return true;
            } catch (ClassNotFoundException e) {
                return false;
            }
        }
    }
    ​
    public class MongoDriverNotPresentsCondition implements Condition
    {
        @Override
        public boolean matches(ConditionContext conditionContext, AnnotatedTypeMetadata metadata)
        {
            try {
                Class.forName("com.mongodb.Server");
                return false;
            } catch (ClassNotFoundException e) {
                return true;
            }
        }
    }

We just have seen how to register beans conditionally based on the presence or absence of a class in classpath.

What if we want to register the `MongoUserDAO` bean only if no other Spring bean of the type `UserDAO` is already registered.

We can create a Condition to check if there is any existing bean of a certain type as follows:

    public class UserDAOBeanNotPresentsCondition implements Condition
    {
        @Override
        public boolean matches(ConditionContext conditionContext, AnnotatedTypeMetadata metadata)
        {
            UserDAO userDAO = conditionContext.getBeanFactory().getBean(UserDAO.class);
            return (userDAO == null);
        }
    }

What if we want to register `MongoUserDAO` bean only if property `app.dbType=MONGO` is set in properties placeholder configuration file?

We can implement that Condition as follows:

    public class MongoDbTypePropertyCondition implements Condition
    {
        @Override
        public boolean matches(ConditionContext conditionContext,
        AnnotatedTypeMetadata metadata)
        {
            String dbType = conditionContext.getEnvironment().getProperty("app.dbType");
            return "MONGO".equalsIgnoreCase(dbType);
        }
    }

We have just seen how to implement various types of Conditions. But there is even more elegant way to implement Conditions using Annotations. Instead of creating a Condition implementation for both `MYSQL` and `MongoDB`, we can create a `DatabaseType` annotation as follows:

    @Target({ ElementType.TYPE, ElementType.METHOD })
    @Retention(RetentionPolicy.RUNTIME)
    @Conditional(DatabaseTypeCondition.class)
    public @interface DatabaseType
    {
        String value();
    }

Then we can implement `DatabaseTypeCondition` to use the `DatabaseType` value to determine whether to enable or disable bean registration as follows:

public class DatabaseTypeCondition implements Condition
{
    @Override
    public boolean matches(ConditionContext conditionContext, AnnotatedTypeMetadata metadata)
    {
        Map<String, Object> attributes = metadata.getAnnotationAttributes(DatabaseType.class.getName());
        String type = (String) attributes.get("value");
        String enabledDBType = System.getProperty("dbType","MYSQL");
        return (enabledDBType != null && type != null && enabledDBType.equalsIgnoreCase(type));
    }
}

Now we can use the `@DatabaseType` annotation on our bean definitions as follows:

    @Configuration
    @ComponentScan
    public class AppConfig
    {
        @DatabaseType("MYSQL")
        public UserDAO jdbcUserDAO(){
            return new JdbcUserDAO();
        }
    ​
        @Bean
        @DatabaseType("MONGO")
        public UserDAO mongoUserDAO(){
            return new MongoUserDAO();
        }
    }

Here we are getting the metadata from `DatabaseType` annotation and checking against the System Property `dbType` value to determine whether to enable or disable the bean registration.

We have seen good number of examples to understand how we can register beans conditionally using `@Conditional` annotation.

Spring Boot extensively uses `@Conditional` feature to register beans conditionally based on various criteria.

You can find various Condition implementations that SpringBoot uses in `org.springframework.boot.autoconfigure` package of `spring-boot-autoconfigure-{version}.jar`.

Now that we've come to know about how Spring Boot uses the `@Conditional` feature to conditionally check whether to register a bean or not, but what exactly triggers the auto-configuration mechanism?

This is what we are going to look at in the next section.

## Spring Boot AutoConfiguration 

The key to the Spring Boot’s auto-configuration magic is `@EnableAutoConfiguration` annotation. Typically we annotate our Application entry point class with either `@SpringBootApplication` or if we want to customize the defaults we can use the following annotations:

    @Configuration
    @EnableAutoConfiguration
    @ComponentScan
    public class Application
    {
    ​
    }

The `@EnableAutoConfiguration` annotation enables the auto-configuration of Spring `ApplicationContext` by scanning the classpath components and registers the beans that are matching various Conditions.

SpringBoot provides various `AutoConfiguration` classes in `spring-boot-autoconfigure-{version}.jar`, which are **responsible for registering various components**.

Typically `AutoConfiguration` classes are annotated with `@Configuration` to mark it as a **Spring configuration class** and annotated with `@EnableConfigurationProperties` to bind the **customization properties** and one or more Conditional bean registration methods.

For example, consider the `org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration` class.

    @Configuration
    @ConditionalOnClass({ DataSource.class, EmbeddedDatabaseType.class })
    @EnableConfigurationProperties(DataSourceProperties.class)
    @Import({ Registrar.class, DataSourcePoolMetadataProvidersConfiguration.class })
    public class DataSourceAutoConfiguration 
    {
        ...
        ...
        @Conditional(DataSourceAutoConfiguration.EmbeddedDataSourceCondition.class)
        @ConditionalOnMissingBean({ DataSource.class, XADataSource.class })
        @Import(EmbeddedDataSourceConfiguration.class)
        protected static class EmbeddedConfiguration {
    ​
        }
    ​
        @Configuration
        @ConditionalOnMissingBean(DataSourceInitializer.class)
        protected static class DataSourceInitializerConfiguration {
            @Bean
            public DataSourceInitializer dataSourceInitializer() {
                return new DataSourceInitializer();
            }
        }
    ​
        @Conditional(DataSourceAutoConfiguration.NonEmbeddedDataSourceCondition.class)
        @ConditionalOnMissingBean({ DataSource.class, XADataSource.class })
        protected static class NonEmbeddedConfiguration {
            @Autowired
            private DataSourceProperties properties;
        ​
            @Bean
            @ConfigurationProperties(prefix = DataSourceProperties.PREFIX)
            public DataSource dataSource() {
                DataSourceBuilder factory = DataSourceBuilder.create(this.properties.getClassLoader()).driverClassName(this.properties.getDriverClassName()).url(this.properties.getUrl()).username(this.properties.getUsername()).password(this.properties.getPassword());
                if (this.properties.getType() != null) {
                    factory.type(this.properties.getType());
                }
                return factory.build();
            }
        }
        ...
        ...
        @Configuration
        @ConditionalOnProperty(prefix = "spring.datasource", name = "jmx-enabled")
        @ConditionalOnClass(name = "org.apache.tomcat.jdbc.pool.DataSourceProxy")
        @Conditional(DataSourceAutoConfiguration.DataSourceAvailableCondition.class)
        @ConditionalOnMissingBean(name = "dataSourceMBean")
        protected static class TomcatDataSourceJmxConfiguration {
            @Bean
            public Object dataSourceMBean(DataSource dataSource) {
                ....
                ....
            }
        }
        ...
        ...
    }

Here, `DataSourceAutoConfiguration` is annotated with `@ConditionalOnClass({ DataSource.class,EmbeddedDatabaseType.class })` which means that the `AutoConfiguration` of beans within `DataSourceAutoConfiguration` will be considered only if the `DataSource.class` and `EmbeddedDatabaseType.class` classes are available on classpath.

The class is also annotated with `@EnableConfigurationProperties(DataSourceProperties.class)` which enables binding the properties in `application.properties` to the properties of `DataSourceProperties` class automatically.

    @ConfigurationProperties(prefix = DataSourceProperties.PREFIX)
    public class DataSourceProperties implements BeanClassLoaderAware, EnvironmentAware, InitializingBean {
    ​
        public static final String PREFIX = "spring.datasource";
        ...
        ...
        private String driverClassName;
        private String url;
        private String username;
        private String password;
        ...
        //setters and getters
    }

With this configuration all the properties that starts with `spring.datasource.*` will be automatically binds to `DataSourceProperties` object.

    spring.datasource.url=jdbc:mysql://localhost:3306/test
    spring.datasource.username=root
    spring.datasource.password=secret
    spring.datasource.driver-class-name=com.mysql.jdbc.Driver

You can also see some inner classes and bean definition methods that are annotated with SpringBoot’s Conditional annotations such as `@ConditionalOnMissingBean`, `@ConditionalOnClass` and `@ConditionalOnProperty` etc.

These bean definitions will be registered in `ApplicationContext` only if those conditions are matched.

You can also explore many other `AutoConfiguration` classes in `spring-boot-autoconfigure-{version}.jar` such as:

- `org.springframework.boot.autoconfigure.web.DispatcherServletAutoConfiguration`
- `org.springframework.boot.autoconfigure.orm.jpa.HibernateJpaAutoConfiguration`
- `org.springframework.boot.autoconfigure.data.jpa.JpaRepositoriesAutoConfiguration`
- `org.springframework.boot.autoconfigure.jackson.JacksonAutoConfigurationetc etc.`

I hope now you have an understanding of how Spring Boot auto-configuration works by using various `AutoConfiration` classes along with `@Conditional` features.
