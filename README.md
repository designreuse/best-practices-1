# Best Practices - (PostgreSQL / Java / Spring / Hibernate)

Some best practices that are useful in this stack.

Contents:

* [PostgreSQL](#postgresql)
* [Java](#java)
* [Spring / Hibernate](#spring--hibernate)
* [Security](#security)
* [REST](#rest)
* [Git](#git)
* [Other](#other)
* [Stack](#stack)

## PostgreSQL

* "To address these difficulties, we recommend using date/time types that contain both date and time when using time zones. We do not recommend using the type time with time zone (though it is supported by PostgreSQL for legacy applications and for compliance with the SQL standard)".

  - https://www.postgresql.org/docs/current/static/datatype-datetime.html#DATATYPE-TIMEZONES

  Further reading for date/time types:

  - http://blog.untrod.com/2016/08/actually-understanding-timezones-in-postgresql.html
  - http://phili.pe/posts/timestamps-and-time-zones-in-postgresql/
  - http://justatheory.com/computers/databases/postgresql/use-timestamptz.html => (Good summary for best practices)
  
* Always use timezone names like "Europe/Istanbul".  Never ever use offsets! Timezone names do take care of daylight savings etc.

	- http://blog.untrod.com/2016/08/actually-understanding-timezones-in-postgresql.html
	
* Always use `timestamptz` when possible. Use `timestamp` when there is no other way. 

	- http://justatheory.com/computers/databases/postgresql/use-timestamptz.html
	
* A possible way of inserting into `timestamptz` when you don't know the offset at a given time and timezone. But this should be your last resort. Note that `+0000` value is not being taken into account since the type is `timestamp`. You add that `+0000` offset value to make it compatible with Java's `OffsetDateTime`. 

	`INSERT INTO test2 (tested_at) VALUES (TIMEZONE('Europe/Istanbul', '2018-03-05 17:35:24+0000'::timestamp));`
	
* You get the offset manually by the below link:

	- https://www.timeanddate.com/worldclock/converter.html
	
* You can still use `AT TIME ZONE` (or `TIMEZONE()`) when querying. It's useful for conversions between timezones.

	- http://justatheory.com/computers/databases/postgresql/use-timestamptz.html
	- http://phili.pe/posts/timestamps-and-time-zones-in-postgresql/#converting-between-timezones
	
- A great section to understand the difference between `timestamp` and `timestamptz`. It also descirebes the logic behind the data type changes when doing time zone conversions. Basically you don't just convert timezones, you also convert an absolute point in time to a specific local time in a given timezone.

	- http://phili.pe/posts/timestamps-and-time-zones-in-postgresql/#converting-between-timezones
	- https://stackoverflow.com/questions/31962786/why-does-postgres-discard-timezone-information-with-at-time-zone (Similar question, no answer)
	
* "The function timezone(zone, timestamp) is equivalent to the SQL-conforming construct timestamp AT TIME ZONE zone."

	- https://www.postgresql.org/docs/current/static/functions-datetime.html#FUNCTIONS-DATETIME-ZONECONVERT
	
* `timestamptz` is always displayed at according to the `TIMEZONE` setting of the db session such as `TIMEZONE='UTC'`. You can set the default value through `postgresql.conf`. The rest is up to the application which may be overridden (via anything like Spring, Hibernate, SQL client etc).

	- http://justatheory.com/computers/databases/postgresql/use-timestamptz.html
	
* You can get the client timezone and offset from ios, Android and javascript. Even if none of these help, you can still use `TimeZone` class in Java to get the offset by the date and timezone. So the above conversion from `timestamp` to `timestamptz` in PostgreSQL is not necessary.

	- https://stackoverflow.com/questions/1091372/getting-the-clients-timezone-in-javascript
	- https://stackoverflow.com/questions/19186666/get-timezone-country-from-iphone
	- https://stackoverflow.com/questions/7672597/how-to-get-timezone-from-android-mobile
	- https://docs.oracle.com/javase/9/docs/api/java/util/TimeZone.html

* Two great sql style guides (or naming conventions) are below. First one seems derived by PostgreSQL:

  - https://launchbylunch.com/posts/2014/Feb/16/sql-naming-conventions/
  - http://www.sqlstyle.guide/
  - https://dba.stackexchange.com/questions/68264/official-postgresql-capitalization-conventions
  - https://www.postgresql.org/docs/current/static/ddl-basics.html
  - https://www.postgresql.org/docs/current/static/ddl-schemas.html

* You can also use JPQL instead of SQL. (You can also use none of the querying languages since Spring Data JPA automatically creates repository implementtions by method name conventions.) A JPQL intro:

	- https://www.thoughts-on-java.org/jpql/
	
* It seems that adding "created_at" and "updated_at" columns are still common in database design:

  - https://softwareengineering.stackexchange.com/questions/118489/does-it-make-sense-to-standardize-including-a-created-date-and-last-updated-date
  - https://stackoverflow.com/questions/38916063/add-timestamp-column-with-default-now-for-new-rows-only
  - https://x-team.com/blog/automatic-timestamps-with-postgresql/
  
* As soon as there are examples of simple codes (or codenames) as primary keys, I'm gonna go for it. Saves a lot of work/time in development and much more elegant. One of the links suggest that the terms is called abbreviated encoding or Celko's encoding. But there is one more thing. It should be kept as small as possible, 3 characters at most (for performance reasons.)

  - https://www.red-gate.com/simple-talk/sql/database-administration/five-simple-database-design-errors-you-should-avoid/
  - https://stackoverflow.com/questions/337503/whats-the-best-practice-for-primary-keys-in-tables
  - https://www.techrepublic.com/blog/10-things/10-tips-for-choosing-between-a-surrogate-and-natural-primary-key/
  - http://www.itprotoday.com/business-intelligence/surrogate-key-vs-natural-key
  - https://dba.stackexchange.com/questions/142825/what-are-the-best-practices-regarding-lookup-tables-in-relational-databases/142841#142841
  - https://stackoverflow.com/questions/4824024/how-important-are-lookup-tables/4824718#4824718
  - https://dba.stackexchange.com/questions/6108/should-every-table-have-a-single-field-surrogate-artificial-primary-key
  
* The choice between UUID vs Auto Increment really depends on the application. Some people defend UUID, dismiss auto-increment and some people do the opposite. There is no consensus on this topic. Some argumentation here:

  - https://dba.stackexchange.com/questions/91669/primary-key-auto-increment-best-practices
  - https://softwareengineering.stackexchange.com/questions/328458/is-it-good-practice-to-always-have-an-autoincrement-integer-primary-key
  - https://stackoverflow.com/questions/404040/how-do-you-like-your-primary-keys
  - https://www.experts-exchange.com/questions/21308273/auto-increment-columns-and-best-practice.html
  - https://medium.com/@Mareks_082/auto-increment-keys-vs-uuid-a74d81f7476a
  - https://www.clever-cloud.com/blog/engineering/2015/05/20/why-auto-increment-is-a-terrible-idea/
  
* In my opinion, the choice between soft delete vs hard delete really depends on the column. If it is a user table, then yes. Soft delete makes a lot of sense. But if it is like a "record" column in "jogtracker" application then hard delete should be sufficient. There is no reason to store incorrect rows inserted by the user itself. There is a lot of debate as usual on this topic.

  - https://stackoverflow.com/questions/378331/physical-vs-logical-soft-delete-of-database-record (Second answer is great.)
  - http://abstraction.blog/2015/06/28/soft-vs-hard-delete (Nice comparison)
  - http://udidahan.com/2009/09/01/dont-delete-just-dont/ (Advocating soft delete)
  - http://jameshalsall.co.uk/posts/why-soft-deletes-are-evil-and-what-to-do-instead (Advocating hard delete)
  - https://www.thoughts-on-java.org/implement-soft-delete-hibernate/ (Soft delete Hibernate implementation - might be useful though)
  
* The recommended timestamp format is "yyyy-MM-dd HH:mm:SS+tz" (ISO 8601). Ex:

  - https://www.postgresql.org/docs/current/static/datatype-datetime.html#DATATYPE-DATETIME-INPUT

```
  2004-10-19 10:23:54+02
```

* Db schema name conventions are similar to the tables.

	- https://www.postgresql.org/docs/current/static/ddl-schemas.html
	- https://dba.stackexchange.com/questions/45589/what-are-the-valid-formats-of-a-postgresql-schema-name
	- https://www.postgresql.org/docs/current/static/sql-syntax-lexical.html
	
* If the table name is a reserved word in Postgresql just try to find a different name. This is what many people do to avoid clashes.

	- https://stackoverflow.com/questions/22256124/cannot-create-a-database-table-named-user-in-postgresql
	- https://dba.stackexchange.com/questions/198907/how-should-we-name-tables-that-resembles-with-reserved-words

* Max char length for email is 320. Therefore, an email field in a db should be varchar(320)
	
	- https://tools.ietf.org/html/rfc3696

* The max output length for bcrypt is 60 characters. Therefore, password field should be varchar(60)

	- https://stackoverflow.com/questions/5881169/what-column-type-length-should-i-use-for-storing-a-bcrypt-hashed-password-in-a-d

## Java

* Packaging by features instead of layers is the common practice:

  - http://www.javapractices.com/topic/TopicAction.do?Id=205
  - https://hackernoon.com/package-by-features-not-layers-2d076df1964d
  - https://dzone.com/articles/package-your-classes-feature
  - http://tidyjava.com/package-feature-demanded/
  - https://stackoverflow.com/questions/11733267/is-package-by-feature-approach-good
  - https://softwareengineering.stackexchange.com/questions/351216/if-i-package-by-feature-but-have-many-homogeneous-classes-what-is-preferable
  
* This item is a note to self. Service methods shouldn't have the same name as dao methods. Ex: 
  - recordService.create
  - recordService.list
  - recordService.update
  - **recordService.getRecordById** => This is wrong
  - recordService.delete
  - **recordService.getReport** => This method can also be changed as "report"
  
  As you can see above "get" would be much more elegant instead of "getRecordById". Standard REST service endpoints should have standard method names. Same goes with "getReport".
  
* DAO classes should be named as xxxRepository. I found this convention by coincidence while checking JHipster:

	- http://gist.asciidoctor.org/?github-mraible/jhipster4-demo//README.adoc
	
* All the classes should be package-private except the beans. Beans should be public otherwise it may cause runtime error on spring. All the members should be private if possible. If not, then the access privilege should be defined according to the usage:

	- https://docs.oracle.com/javase/tutorial/java/javaOO/accesscontrol.html
	- https://stackoverflow.com/questions/215497/in-java-difference-between-package-private-public-protected-and-private
	
* Some naming conventions for MVC objects:

	- Entity => User
	- Data transfer object => UserDTO or UserModel. (UserDTO is much more common in Java world.)
	- Data acess object => UserDAO or UserRepository
	- Service interface => UserService
	- Service Implementation => AbstractUserService or UserServiceImpl (UserServiceImpl is much more common in Spring world.)
	- **May need references here.**

## Spring / Hibernate

* "The @Transactional annotation is metadata that specifies that an interface, class, or method must have transactional semantics". It still needs to be tested to see if I use it correctly:

  - https://docs.spring.io/spring/docs/5.0.2.RELEASE/spring-framework-reference/data-access.html#transaction-declarative-attransactional-settings 
  
* JPA is Java Persistence API and Hibernate is JPA provider. JPA has only java interfaces but not its implementations whereas Hibernate has implementations of those interfaces. Spring Data JPA is a JPA Data Access Abstraction. Spring Data offers a solution to GenericDao custom implementations. It can also generate JPA queries on your behalf through method name conventions. All the annotations in regarding the data layer on Java comes from JPA. Hibernate exists in this jar: `spring-boot-starter-data-jpa`

	- https://stackoverflow.com/questions/23862994/what-is-the-difference-between-hibernate-and-spring-data-jpa
	- https://www.concretepage.com/forum/thread?qid=441
	- https://coderanch.com/t/685335/certification/JPA-Spring-JPA-Spring-Data
	- https://dzone.com/articles/easier-jpa-spring-data-jpa
	
* If import.sql is found in the classpath, Hibernate will load it automatically.

	- https://www.mkyong.com/spring-boot/spring-boot-spring-data-jpa-oracle-example/
	- https://docs.spring.io/spring-boot/docs/current/reference/html/howto-database-initialization.html
	
* Typically, your repository interface will extend Repository, CrudRepository, JPARepository or PagingAndSortingRepository in Spring Data JPA.

	- https://www.mkyong.com/spring-boot/spring-boot-spring-data-jpa-oracle-example/
	- https://docs.spring.io/spring-data/data-commons/docs/current/reference/html/#repositories
	
* Below links detail the differences between `JPARepository`, `PagingAndSortingRepository`, `CrudRepository`. Basically, `JPARepository` > `PagingAndSortingRepository` > `CrudRepository` > `Repository` and they all extend each other respectively. That being said `JPARepository` is the biggest one containing all the methods.

	- https://stackoverflow.com/questions/14014086/what-is-difference-between-crudrepository-and-jparepository-interfaces-in-spring
	- http://www.baeldung.com/spring-data-repositories
	
* There are three ways to implement a complete where clause on a dynamic level:
	
	1. As in jogtracker that you just replace the operator and direclty send the whole clause to sql query. But this may be open to sql injection attacks and you must be careful on that.
	
	2. Using RQSL, but it does not seem like it's a standart yet and you should use a third party library which is not widely adopted by the community. Here are the links:
	
		- http://www.baeldung.com/rest-api-search-language-rsql-fiql
		- http://putracode.com/implementation-dynamic-specification-using-rsql-parser-example/
		- https://github.com/jirutka/rsql-parser
		
	3. The third and most appropriate way is to use Spring Data JPA Specifications. Related article is below:
	
		- http://www.baeldung.com/rest-api-query-search-or-operation
		
* Some discussion on JPA entities.

	- https://stackoverflow.com/questions/6033905/create-the-perfect-jpa-entity
	
* Exception handling should be done via ControllerAdvice annotation in Spring cause it provides a more centralized approach which leads to a much cleaner code. Jogtracker also uses ControllerAdvice annotations. One of the below links also provide a great example for a json response output of a rest api error:

	- http://www.baeldung.com/exception-handling-for-rest-with-spring
	- https://lankydanblog.com/2017/09/12/global-exception-handling-with-controlleradvice/
	- http://www.baeldung.com/global-error-handler-in-a-spring-rest-api
	
* Rest apis can be documented either by using external tools like swagger (which is one of the most popular) or Spring Rest Docs. Since Spring is our framework for doing most of the work, we will rely on this for now. Below is further reading and Spring Rest Docs resources.  Additionally you can also create documentation by Postman now. Postman looks like it's progressed a lot by the time:

	- https://opencredo.com/rest-api-tooling-review/
	- https://swagger.io/
	- https://spring.io/guides/gs/testing-restdocs/
	- http://www.baeldung.com/spring-rest-docs
	- https://www.getpostman.com/
	
* Spring REST Docs is chosen over swagger in general. Below are the links. But what about Postman?

	- https://dzone.com/articles/a-qa-with-andy-wilkinson-on-spring-rest-docs
	- https://stackoverflow.com/questions/34457729/what-is-the-benefit-of-using-spring-rest-docs-comparing-to-swagger
	- https://dzone.com/articles/introducing-spring-auto-rest-docs
	
* I think below article is the best for rest api versioning that also includes spring fw. There is nowhere else from here to go. It includes everything. Depending on your needs you can choose which way you want from the list below:

	- http://www.springboottutorial.com/spring-boot-versioning-for-rest-services
	
* "We generally recommend that you locate your main application class in a root package above other classes. The @EnableAutoConfiguration annotation is often placed on your main class, and it implicitly defines a base “search package” for certain items. For example, if you are writing a JPA application, the package of the @EnableAutoConfiguration annotated class will be used to search for @Entity items. Using a root package also allows the @ComponentScan annotation to be used without needing to specify a basePackage attribute. You can also use the @SpringBootApplication annotation if your main class is in the root package."

	- https://docs.spring.io/spring-boot/docs/current/reference/html/using-boot-structuring-your-code.html
	
* It is possible to generate JPA entity classes from tables by using Eclipse (WTP), Hibernate, Oracle Enterprise Plugin for Eclipse etc. But it is not necessary now, since there a just a few classes and tables.

* If you are using a sequential primary key with JPA (and Hibernate), you should define this sequence both in the code and db. Data type `serial` is enough for defining such a sequence in postgres. The Java code should look like this:

```
  @Id
  @SequenceGenerator(name="sequance_name", sequenceName="sequance_name", allocationSize=1)
  @GeneratedValue(strategy=GenerationType.SEQUENCE, generator="sequance_name")
  private short id;
```
  
All the annotations above are important for the definition of the sequence otherwise it won't work, including the allocationSize. If you don't define the allocationSize the id value might go negative or collisions might happen. 

* When a not null field is defined in the db, normally you should be using both `@NotNull` and `@Column(nullable = false)` annotations. `@NotNull` annotation just checks POJO and `@Column(nullable = false)` annotation checks the db for null instances but Hibernate applies the bean validation constraints to DDL by default so @NotNull is enough in this case, thanks to the Hibernate.

	- https://auth0.com/blog/integrating-spring-data-jpa-postgresql-liquibase/
	- https://stackoverflow.com/questions/7439504/confusion-notnull-vs-columnnullable-false
	- https://stackoverflow.com/questions/2899073/basicoptional-false-vs-columnnullable-false-in-jpa
	
* Java Bean Validation API is the interface and the Hibernate Validator is the implementation. Since we use Spring and Hibernate is the default provider, importing `@Email` annotation from `javax.validation.constraints.Email` should be sufficient.
	
	- http://www.baeldung.com/javax-validation
	- https://stackoverflow.com/questions/35050936/jpa-validation-of-email-string-collection
	- https://stackoverflow.com/questions/4459474/hibernate-validator-email-accepts-askstackoverflow-as-valid
	
* Below is a list of fundamental Hibernate annotations and lets you start from the get go.

	- http://www.techferry.com/articles/hibernate-jpa-annotations.html
	- https://auth0.com/blog/integrating-spring-data-jpa-postgresql-liquibase/
  
* Core Spring annotations for your taste:

	- http://www.techferry.com/articles/spring-annotations.html#ModelAttribute
	
* The difference between `@ModelAttribute` and `@RequestBody` is defined below. `@ModelAttribute` is used for request params whereas `@Requestbody` is used for JSON/XML data in the request for REST services. Therefore `@RequestBody` is what we need.

	- https://stackoverflow.com/questions/21824012/spring-modelattribute-vs-requestbody
	- https://stackoverflow.com/questions/43716767/spring-mvc-requestbody-vs-modelattribute
	
* Some examples of Spring bean scopes with the `@Scope` annotation. Default and most used one is singleton.

	- https://www.mkyong.com/spring/spring-bean-scopes-examples/
	- http://www.baeldung.com/spring-bean-scopes

* Default value for a column can be handled by the db so you don't need an additional annotation or code in the JPA layer. But if you really need to handle that in the code you can just set an initial value for the variable and that's it.

	- https://stackoverflow.com/questions/197045/setting-default-values-for-columns-in-jpa
	
* Soft delete can easily be achieved by a single `@Where` annotation of Hibernate. This annotation should be placed right in the beginning of the entity.

	- https://stackoverflow.com/questions/19323557/handling-soft-deletes-with-spring-jpa
	- https://dzone.com/articles/hibernate-where-clause
	
* `OffsetDateTime` should be used to map PostgreSQL's `timestamptz`. That's the default data type to be used in Java 8 JDBC Driver. Timezone offset is also necessity in `ZonedDateTime` so it's meaningless to use it. `OffsetDateTime` is the exact match of `timestamptz`

	- https://jdbc.postgresql.org/documentation/head/java8-date-time.html
	- https://docs.oracle.com/javase/9/docs/api/java/time/ZonedDateTime.html

* Change the default timezone settings by the below links. They include all the possible answers including JVM, Spring, Hibernate, Jackson etc.

	- https://stackoverflow.com/questions/508019/how-to-store-date-time-and-timestamps-in-utc-time-zone-with-jpa-and-hibernate/40438746#40438746
	- https://moelholm.com/2016/11/09/spring-boot-controlling-timezones-with-hibernate/
	- https://stackoverflow.com/questions/46151633/how-to-make-default-time-zone-apply-in-spring-boot-jackson-date-serialization
	
* If you use `RepositoryRestResource` together with `RestController` then `RestController` overrides the methods.

	- https://stackoverflow.com/questions/22824840/when-to-use-restcontroller-vs-repositoryrestresource (check comments.)
	
* To return 404 http status for that do not exist, just throw `ResourceNotFoundException`:

	- https://docs.spring.io/spring-data/rest/docs/current/api/org/springframework/data/rest/webmvc/ResourceNotFoundException.html
	
* Use `ResourceNotFoundException` together with `Optional` in Java 8. `Repository` methods return `Optional` values so `orElseThrow` method of `Optional` will work. `accountRepository.findById(id).orElseThrow(ResourceNotFoundException::new);`

	- http://www.baeldung.com/java-optional
	
* You can simply compile and run spring boot with one single gradle command and that is `bootRun` however you can't stop the application gracefully and kill the daemon. You can build the application without testing by `.\gradlew build -x test` command. Additionally, you can use `spring-boot-devtools` which automatically deploys the server when the classpath is changed. Finally you can write a single .bat file that contains both `.\gradlew clean build -x test` and `.\java -jar .\build\libs\app.jar` so that you can create the jar and start the app with one single command.

	- https://stackoverflow.com/questions/37338407/what-is-the-difference-between-gradle-bootrun-and-gradle-run-to-run-a-spring
	- https://stackoverflow.com/questions/39123416/ctrlc-w-spring-boot-gradle-kills-gradle-daemon
	- https://docs.spring.io/spring-boot/docs/current/reference/html/using-boot-devtools.html

## Security

* Argon2 won the password hashing competitions, but due to Spring implementations we should stick to bcrypt for now. Argon2 might still need some time. Links below:

  - https://hynek.me/articles/storing-passwords/
  - https://security.stackexchange.com/questions/107337/reviews-of-argon2
  - https://docs.spring.io/spring-security/site/docs/5.0.0.RELEASE/api/org/springframework/security/crypto/bcrypt/BCryptPasswordEncoder.html
  - https://docs.spring.io/spring-security/site/docs/5.0.0.RELEASE/api/org/springframework/security/crypto/scrypt/SCryptPasswordEncoder.html
  
* I did some research on authentication methods in REST and a lot of developers seem to misunderstand the concepts such as OAuth and JWT. The only approach that has been agreed upon is OpenID Connect on top of OAuth 2.0. But I still think that it might be an architectural overkill because either I have to use a third-party identity provider for authentication or implement it by myself. HMAC seems a reasonable approach for machine-machine communicationin REST with API keys and API secrets.

* With a detailed research, now I understand why oauth should be used for any kind of API authentication and security. Here are the use cases that are applicable in our scenarios:

	1. You can use OpenID Connect to consume a third pary API. For example, you can use Google OpenId Connect to consume Google APIs and reach Google Servers:
	
		- https://developers.google.com/identity/protocols/OAuth2
	
	2. You can use Open Id Connect to use authentication as a service by third party identity providers. You can use Google Accounts of your users to authenticate to your APIs. Note: Auth0 is great for this way of authentication. Includes lots of identity providers.
	
		- https://connect2id.com/learn/openid-connect
		- https://openidconnect.net/
		- https://auth0.com/
	
	3. You can entirely build an oauth server by yourself. If you own Resource Server, Authorization Server and Identity Provider, then this is the authentication method that you should use. Examples are Toptal entrance application and Lefolio. It's OK to use Resource Owner Password Credentials Grant in this case. Client Credentials Grant should be used for machine to machine communication.
	
		- http://andyfiedler.com/2014/09/how-secure-is-the-oauth2-resource-owner-password-credential-flow-for-single-page-apps
		- https://www.scottbrady91.com/OAuth/Why-the-Resource-Owner-Password-Credentials-Grant-Type-is-not-Authentication-nor-Suitable-for-Modern-Applications
		- https://www.ory.sh/run-oauth2-server-open-source-api-security
	
	4. For the details of ouath flows and grants check links below:
	
		- https://auth0.com/docs/api-auth/which-oauth-flow-to-use
		- https://www.digitalocean.com/community/tutorials/an-introduction-to-oauth-2

* In Spring Security with OAuth 2.0, an application can contain both Authorization Server and Resource Server. "A Resource Server (can be the same as the Authorization Server or a separate application) serves resources that are protected by the OAuth2 token." 

	- http://projects.spring.io/spring-security-oauth/docs/oauth2.html
	
* These three articles are used to enable ssl in spring security:

	- https://memorynotfound.com/spring-boot-configure-tomcat-ssl-https/
	- https://www.thomasvitale.com/https-spring-boot-ssl-certificate/
	- https://stackoverflow.com/questions/47700115/tomcatembeddedservletcontainerfactory-is-missing-in-spring-boot-2
	
* To implement oauth on spring security there is an amazing article on Medium. The others are supplementary:

	- https://medium.com/@nydiarra/secure-a-spring-boot-rest-api-with-json-web-token-reference-to-angular-integration-e57a25806c50
	- https://stackoverflow.com/questions/46999940/spring-boot-passwordencoder-error/47150363?noredirect=1#comment85408407_47150363
	- https://projects.spring.io/spring-security-oauth/docs/oauth2.html
	
* ResourceId in OAuth and JWT is useful when there are multiple resources and multiple resource servers. It is unnecessary work to add resourceIds if there are only one resource. Below links do not explain this but some links related with resource id usage:

	- https://github.com/spring-projects/spring-security-oauth/issues/299
	- https://github.com/spring-projects/spring-security-oauth/issues/325
	- https://stackoverflow.com/questions/28703847/how-do-you-set-a-resource-id-for-a-token
	- http://jwt.io
	
* Random 64 characters of hex is enough for signing JWT tokens a.k.a. `signing-key`:

	- https://github.com/dwyl/hapi-auth-jwt2/issues/48
	- https://github.com/dwyl/learn-json-web-tokens
	
* Implementing your custom security annotation filter is as easy as below:

	- https://dreamix.eu/blog/java/implementing-custom-authorization-function-for-springs-pre-and-post-annotations
	- https://docs.spring.io/spring-security/site/docs/current/reference/html/el-access.html#el-access-web-path-variables
	
* `@PreAuthorize` is used to give access before the method is called. `@PostAuthorize` is used to give access according to the return value. `@PreFilter` is filtering the data before the method call, `@PostFilter` is filtering the data after the method is invoked.

	- https://stackoverflow.com/questions/22093018/valid-use-case-for-postauthorize-and-postfilter-annotations
	- https://docs.spring.io/spring-security/site/docs/current/reference/html/el-access.html#el-pre-post-annotations

## REST

* The most common starting points for JSON:

  - https://mvnrepository.com/artifact/org.json/json
  - https://github.com/FasterXML/jackson
   
* There is a consensus about RESTful APIs in the coomunity, except the API versioning. Here are links:

  - https://hackernoon.com/restful-api-designing-guidelines-the-best-practices-60e1d954e7c9
  - https://blog.mwaysolutions.com/2014/06/05/10-best-practices-for-better-restful-api/
  - https://www.snyxius.com/21-best-practices-designing-launching-restful-api/
  - http://www.vinaysahni.com/best-practices-for-a-pragmatic-restful-api
  - https://github.com/RestCheatSheet/api-cheat-sheet#api-design-cheat-sheet
  - https://medium.com/@schneidenbach/restful-api-best-practices-and-common-pitfalls-7a83ba3763b5 (Also technical details here!)
  
  Further reading on REST:
  
  - http://www.restapitutorial.com/resources.html
  - https://github.com/RestCheatSheet

* JSON properties should have camelCase as it is told in REST conventions. The below code sample from jogtracker is WRONG!

```
  @JsonProperty("id")
  private Integer recordId;

  @JsonProperty("user_id")
  private Integer userId;
	
  @JsonProperty("air_temp")
  private Double airTemp;
	
  @JsonProperty("rel_temp")
  private Double relTemp;	
```

## Git

* Below are different articles on git branching strategy. Will be discessed in detail later.

	- http://nvie.com/posts/a-successful-git-branching-model/
	- https://barro.github.io/2016/02/a-succesful-git-branching-model-considered-harmful/
	- http://www.nomachetejuggling.com/2017/04/09/a-different-branching-strategy/
	- https://hackernoon.com/how-the-creators-of-git-do-branches-e6fcc57270fb
	- https://news.ycombinator.com/item?id=11190310

## Other

* Connection pools should be used to improve the performance and HikariCp seems to be the most obvious choice.

	- http://www.baeldung.com/hikaricp
	- https://dzone.com/articles/database-connection-pooling-in-java-with-hikaricp 
	
* You need to mock objects in order to do unit testing because you are depending on db objects in these type of applications. JMockit and Mockito are clearly winners but Mockito has much bigger usage and community. It's either JMockit or PowerMock + Mockito (aka PowerMockito). 

	- https://www.mkyong.com/unittest/junit-spring-integration-example/
	- https://www.mkyong.com/unittest/unit-test-what-is-mocking-and-why/
	- http://www.baeldung.com/mockito-vs-easymock-vs-jmockit
	- https://www.slant.co/topics/259/~best-mock-frameworks-for-java

* Using either Guava or Apache Commons library is a good start for a Java project.

	- https://commons.apache.org/
	- https://github.com/google/guava
	- https://stackoverflow.com/questions/1444437/apache-commons-vs-guava-formerly-google-collections (Very old comparison)
	- https://stackoverflow.com/questions/4542550/what-are-the-big-improvements-between-guava-and-apache-equivalent-libraries (Old again)
	- https://stackoverflow.com/questions/14978699/is-it-a-good-idea-to-use-google-guava-library-for-android-development
	
* "Guava is a productivity multiplier for Java projects across the board: we aim to make working in the Java language more pleasant and more productive. The Guava project contains several of Google's core libraries that we rely on in our Java-based projects: collections, caching, primitives support, concurrency libraries, common annotations, string processing, I/O, and so forth." Use it when it's needed.

	- https://github.com/google/guava
	- https://github.com/google/guava/wiki/PhilosophyExplained

* It seems like jcenter surpassed mavenCentral.

	- https://stackoverflow.com/questions/24852219/android-buildscript-repositories-jcenter-vs-mavencentral
	- https://jfrog.com/knowledge-base/why-should-i-use-jcenter-over-maven-central/

* Lombok is a great tool for creating boilerplate code such as getters, setters and toString(), hashCode(), equals() methods. It is considered to be safe and supports Java 9.

	- https://stackoverflow.com/questions/3852091/is-it-safe-to-use-project-lombok/12807937
	- https://stackoverflow.com/users/18122/oliver-gierke
	- https://github.com/spring-projects/spring-data-rest/blame/897bc88d6922e755038a6a8e34ca25ace6a27689/spring-data-rest-webmvc/src/main/java/org/springframework/data/rest/webmvc/alps/RootResourceInformationToAlpsDescriptorConverter.java#L20
	- https://projectlombok.org/
	- https://github.com/franzbecker/gradle-lombok

* Liquibase is a source / version control tool for database. Flyway is also an alternative to this tool and referenced by Spring Boot / Pivotal. No need for the jogtracker though, but may be necessary for bigger projects and production environments.

	- https://stackoverflow.com/questions/29760629/why-and-when-liquibase
	- https://www.liquibase.org/
	- https://flywaydb.org/

# Stack

	* Windows 10
	* PostgreSQL 10.2
	* Java 9
	* Spring Boot 2.0.0
	* Gradle 4.6
	* Eclipse Oxygen.2
	* DBeaver
	* Git 2.16
