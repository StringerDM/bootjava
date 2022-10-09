<img src="http://javaops.ru/static/img/logo/javaops_30.png" width="223"/>

Открытый курс для всех желающих приобщиться к живой современной разработке на Java
# [Разработка Spring Boot 2.x HATEOAS приложения (BootJava)](http://javaops.ru/view/bootjava?ref=gh)
## [Программа](http://javaops.ru/view/bootjava#program)

### Java приложения на самом современном и востребованном стеке: Spring Boot 2.x, Spring Data Rest/HATEOAS, Lombok, JPA, H2, ....
Мы создадим с нуля основу любого современного REST веб-приложения: аутентификация и авторизация на основе ролей, регистрация пользователя в приложении, управление своим профилем и администрирование пользователей.

Конспект:
<details>
  <summary>1. Основы Spring Boot</summary>
    1.1	Создаем проект через Spring Initializer
    
    commit: https://github.com/StringerDM/bootjava/commit/35a21d499357b464ebb5b571cb97ac0bc5e57f01
    
    -   Подключаем зависимости:
    -   Lombock
    -   Spring Web
    -   H2 database
    -   Spring Data JPA

    По умолчанию приложение открывается по адресу localhost:8080

    Ссылки: 
    Spring Initializrs: https://start.spring.io/

    1.2	Spring Boot maven plugin. Конвертация в WAR
  
    Ссылки:
    Конвертация JAR приложения в WAR http://spring-projects.ru/guides/convert-jar-to-war-maven/
  
    1.3	Настройка проекта
    Готовый проект с патчами находится в ветке patched:   git clone --branch patched https://github.com/JavaOPs/bootjava.git

    1.4 Проект Lombok
   
    Commit: https://github.com/StringerDM/bootjava/commit/ef6cdb5d5fb182bf1387e77206ddf174ce4ed005 
    
    В Pom.xml он уже у нас есть, причем <optional> true </optional>:
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>

    Если мы посмотрим, что такое optional dependencies: http://maven.apache.org/guides/introduction/introduction-to-optional-and-excludes-dependencies.html то увидим,  что оно используется для библиотек, у которых есть много транзитивных зависимостей и подключая эти библиотеки с optional мы избавляемся от их зависимостей которые нам возможно не понадобятся. У нас совсем не библиотека, а собственный проект поэтому использование optional достаточно сомнительно.
  
    Кроме того, если мы посмотрим: Maven Scope for Lombok (Compile vs. Provided) https://stackoverflow.com/questions/29385921/548473 то увидим что в оф документации Lombok нужно подключать со скопом provided. То есть lombok на нужен только на этапе компиляции и из сборки он исключается. 
  
    И еще одна ссылка Exclude lombok in Spring Boot https://stackoverflow.com/questions/45202639/548473 где говорится что если мы делаем JAR то туда включается embedded Tomcat и все зависимости даже со скопом provided также попадают в нашу сборку. Для того чтобы исключить lombock из сборки нужно явно добавить в pom.xml в boot maven plugin явную конфигурацию <exclude>:
  
              <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
  
    Добавляем getters and setters и пустой + со всем аргументами конструктор используя аннотации Lombok.
        @Data
        @NoArgsConstructor
        @AllArgsConstructor

    Полезная аннотация которая добавляет логгер классу.
        @Log   

    Ссылки: Фичи Lombok https://urvanov.ru/2015/09/22/project-lombok/

</details>

<details>
  <summary>2. Работа с DB (H2, Spring Data JPA)</summary>
    2.1	Spring Data JPA. ApplicationRunner
    
    Commit: https://github.com/StringerDM/bootjava/commit/530474b5f8ac9f85dd89284476fcb42685cb7aba
        
    В проекте у нас уже есть подключенный spring-boot-starter-data-jpa, также подключина БД H2 и при запуске sping boot уже может сразу поднять БД с настройками по умолчанию. База embedded т.е. она работает в тойже JVM что и наше приложение и по умолчанию spring boot создает ее прямо и entites (классы отмеченные @Entity).
    
    Добавляем требуемые аннотации в модель для валидации, названия таблиц и колонок (не обязательно, по умолчанию по имени полей). См. commit.
    
    @Entity
    @Table(name = "")
    @Column(name = "")
    
    @Size(max = 128)
    @NotEmpty
    @NotNull
    @Email
    
    и т.д. 
    
    Чтобы не создавать поле Id можно унаследоваться от класса AbstractPersistable<Integer> который уже содержит поле Id с нужными аннотациями для генерации ключей в базе и методами setId, isNew, equals, heshcode, toString.
    
    Также добавим lombok аннотацию @ToString(callSuper = true, exclude = {"password"}) с параметрами "callSuper = true" для включения поля id из суперкласса и exclude = {"password"} для исключения из строки поля password.
    
    Для ролей мы не делаем отдельное entity а указываем их как @ElementCollection(fetch = FetchType.EAGER)
    
    Cо spring boot v2.3 убрали валидацию по умолчанию, поэтому добавили в pom.xml:
    
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>
    
    Далее определяем интерфейс userRepository extends JpaRepository<User, Integer>. Имплементация по умолчанию JpaRepository это класс SimpleJpaRepository, сбда можно брейк поинты ставить для дебага.
    
    В aplication.property сделаем одну настройку (Common application Data properties - https://docs.spring.io/spring-boot/docs/current/reference/html/appendix-application-properties.html#data-properties все настройки spring boot и по ключевому слов JPA мы можем найти все конфигурационный классы и что можно объявлять):
    
    spring.jpa.show-sql=true - для отображения запросов в базу. (это крайне полезно для Hibernate во время разработки).
    
    Запускаем приложение и смотрим как наша таблица создается. По умолчанию для embedded БД таблицы сначало дропаются, затем создается общий для всех hibernate siquence и создаются таблицы.
    
    Зделаем сначало заполнение таблиц програмно. В spring boot есть 2 интерфейса ApplicationRunner and CommandLineRunner которые позволяют выполнять произвольный код после старта приложения. Разница между ними в том что ApplicationRunner мы принимае массив аргументов обернутый в класс который позовляет нам выполнять какието удобные вещи например getOptional value. Реализовывать интерфейсы можно в любом из бинов spring, мы реализуем его в главном RestaurantVotingApplication:
    
    //реализуем интерфейс ApplicationRunner
    @SpringBootApplication
    @AllArgsConstructor
    public class RestaurantVotingApplication implements ApplicationRunner {
    
    //инжектим userRepository через аннотацию @AllArgsConstructor
    private final UserRepository userRepository;
    
    //вставляем в базу 2х юзеров:
    
    @Override
    public void run(ApplicationArguments args) {
        userRepository.save(new User("user@gmail.com", "User_First", "User_Last", "password", Set.of(Role.ROLE_USER)));
        userRepository.save(new User("admin@javaops.ru", "Admin_First", "Admin_Last", "admin", Set.of(Role.ROLE_USER, Role.ROLE_ADMIN)));
    }
    
    Запускаем приложение и видимо что Hibernat делает 3 запроса, 1м он достает 2х юзеров и потом на каждого юзера он достает роли. Это измвестная проблема n+1, если бы у нас было 10 тысяч юзеров то Hibernate сгенерил бы 10 001 запрос.
    Проблема N+1. Стратегии загрузки коллекций
      N+1 selects issue https://stackoverflow.com/questions/97197/548473
      в JPA             https://dou.ua/lenta/articles/jpa-fetch-types/
      в Hibernate       https://dou.ua/lenta/articles/hibernate-fetch-types/
      если ссылки выше не открываются: Runet Censorship Bypass https://chrome.google.com/webstore/detail/%D0%BE%D0%B1%D1%85%D0%BE%D0%B4-%D0%B1%D0%BB%D0%BE%D0%BA%D0%B8%D1%80%D0%BE%D0%B2%D0%BE%D0%BA-%D1%80%D1%83%D0%BD%D0%B5%D1%82%D0%B0/npgcnondjocldhldegnakemclmfkngch    
    В TopJava мы решали её тремя сопособами:
      - Через fetch Join
      - Entity Graff
      - И для ролей в Юзере мы делали @BatchSize(size = 20)
      
    В Hibernate есть настрока которая позволяет выставлять batch size глобально для всего приложения. 
      Hibernate configurations - http://docs.jboss.org/hibernate/orm/current/userguide/html_single/Hibernate_User_Guide.html#configurations - по ссылке можно найти настройку spring.jpa.properties.hibernate.default_batch_fetch_size=20 (укажем 20 по размеру колонок в таблице на странице).    
      hibernate.jdbc.fetch_size vs hibernate.jdbc.batch_size - https://stackoverflow.com/questions/21257819/548473
    
    Также добавим spring.jpa.properties.hibernate.format_sql=true - форматирование sql запросов в выводе (запросы читать легче)
    и spring.jpa.properties.hibernate.jdbc.batch_size=20 это количество в баче апрдейтов и инсертов хибернейта.
    # https://stackoverflow.com/questions/21257819/what-is-the-difference-between-hibernate-jdbc-fetch-size-and-hibernate-jdbc-batc
    
    И последняя настройка, если мы посмотрим на лог то мы увидим Warning - spring.jpa.open-in-view is enabled by default и нужно его выключить:
    spring.jpa.open-in-view=false
    Open Session In View Anti-Pattern - # https://vladmihalcea.com/the-open-session-in-view-anti-pattern/
    spring.jpa.open-in-view - # https://stackoverflow.com/a/48222934/548473
    Это антипаттерн - если в модели при преобразовании view остались какието не проинициализированный поля которые lazy proxy то открывается транзакция и делаются еще дополнительный запросы в базу чтобы проинициализировать эти поля.
    Запускаем приложение и смотрим на отработку запроса findAll и видем что теперь только 2 запроса.1й для юзеров и 1 запрос для всех ролей. Если юзеров будет много то роли будут доставаться пачками по 20 юзеров.
    
    2.2 H2. Популирование и конфигурирование
    
    Commit: https://github.com/StringerDM/bootjava/commit/2e03672e1984c941211e37256e7b07eaea5445a3
        
    Открытая СУБД написанная полностью на Java не смотря на малый размер, поддерживает много возможностей... 
    
    Первое что мы сделаем это перейдем с формата .properties на формат .yaml 
    Явно объявим то что было по дефолту 
    Встроенная база 
          hibernate:
            ddl-auto: create-drop
          datasource:
            url: jdbc:h2:mem:voting
            username: sa
            password:
          #    tcp: jdbc:h2:tcp://localhost:9092/mem:voting
          # Absolute path
          #    url: jdbc:h2:C:/projects/bootjava/restorant-voting/db/voting
          #    tcp: jdbc:h2:tcp://localhost:9092/C:/projects/bootjava/restorant-voting/db/voting
          # Relative path form current dir
          #    url: jdbc:h2:./db/voting
          # Relative path from home
          #    url: jdbc:h2:~/voting
          #    tcp: jdbc:h2:tcp://localhost:9092/~/voting
          h2.console.enabled: true

    если у вас версия spring-boot 2.5.0 и выше, добавьте в application.yaml:
    spring.jpa.defer-datasource-initialization: true

    Чтобы поднять H2 TCP сервер мы делаем конф. класс и объявляем там
          @Bean(initMethod = "start", destroyMethod = "stop")
          public Server h2Server() throws SQLException {
              log.info("Start H2 TCP server");
              return Server.createTcpServer("-tcp", "-tcpAllowOthers", "-tcpPort", "9092");
          }

    При этом в pom нам нужно убрать runtime зависимости h2 потомучто классы h2 теперь понадобились на этапе компиляции.
    
    Запускаем приложение и подключаемся к базе через idea. Если мы попробуем приконектится по url то ничего не выйдет, конект пройдет но если мы на неё посмотрим то никаких баз не увидим. База данных к которой мы приконектились поднимается в памяти в процессе JVM idea и никакой отношение к БД приложения не имеет. Поэтому мы подняли TCP сервер чтобы мы могли приконектится извне - jdbc:h2:tcp://localhost:9092/mem:voting
    
    Подключаемся к базе и делаем интеграцию с Idea выбирая в persistence/springboot -> data source – H2.
    
    H2 console также доступна по http://localhost:8080/h2-console
    
    Давайте пропопулируем нашу БД не через приложение а через скрипт как это обычно делается.
    Из applicationRunner удаляем save user и добавляем в ресурсы файл data.sql где популируем users и userRoles (у spring boot 2 файла который он автоматически исполняет data.sql и schema.sql schema нам не требуется т.к. за создание схемы базы отвечает hibernate).
    Loading Initial Data https://www.baeldung.com/spring-boot-data-sql-and-schema-sql
    Запускаем приложение и сталкиваемся с проблемой что ID у нас должно быть NotNull но оно автоматически не генерится. Смотрим на лог генерации таблицы и видимо что ID сгенерировалось как обычное поле.
    H2: NULL not allowed for column “ID”  - https://stackoverflow.com/a/54697387/548473
    Смотрим решение проблемы на stackoverflow и видим 3 варианта:
    
      1.	Поменять @GeneratedValue с авто, как у нас в наследуемом AbstractPersistable классе на
      change @GeneratedValue to strategy = GenerationType.IDENTITY

      2.	Set spring.jpa.properties.hibernate.id.new_generator_mappings=false (spring-boot alias spring.jpa.hibernate.use-new-id-generator-mappings) это означает      
      работу по старой стратегии не по sequence а по identity 

      3.	insert with nextval: INSERT INTO TABLE(ID, ...) VALUES (hibernate_sequence.nextval, ...) – вставлять в базу ID сгенерированный hibernate.
     
    Для нас самое просто использовать 2й вариант. Теперь все работает. Со старой стратегии ID генерится как identity.
    
    2.3 Рефакторинг model. Spring Data JPA @Query
    
    commit: https://github.com/StringerDM/bootjava/commit/f789d22071f65c732533c9b512015e8a05b8ede5
    Заменим стандартный AbstractPersistable собственным классом BaseEntity:
        @Access(AccessType.FIELD)
        Здесь объявляем чтобы hibernate работал с entity по полям - https://stackoverflow.com/a/6084701/548473
    Методы тип isNew() не нужно помечать что они transient.
    Методы equals и hashCode сделаны попроще. 
    И в equal эту строчку взяли из класса AbstractPersistable:
    
        if (o == null || !getClass().equals(ProxyUtils.getUserClass(o))) {
        return false;
        }

    Т.к. hibernate может проектировать классы и перед сравнением их нужно развернуть.
    Ссылка как правильно в Entity hibernate переопределять equals и hashCode (очень частая ошибка)
    https://stackoverflow.com/questions/1638723

    По правилам рекомендуется делать уникальное неизменяемое бизнес поле, а обычно такого нет и во всех проектах использовался primary key. На primary key сделали @GeneratedValue(strategy = GenerationType.IDENTITY) как у нас и генирурется на данный момент, поэтому в файле конфигурации id.new_generator_mappings: false уже не требуется.

    Все наши Entity классы будем наследовать он BaseEntity.

    interface UserRepository {
    В репозиториях в запросе @Query для именованных параметров (:email) теперь в методе можно не указывать аннотацию @Param(“email”), hibernate теперь берет имя параметра через отражение.

        @Query("SELECT u FROM User u WHERE u.email = LOWER(:email)")
        Optional<User> findByEmailIgnoreCase(String email);
    }

    Также как и в контроллерах в аннотациях @Pasthariable и @RequestParam атрибуты nameValue не требуется.

</details>
<details>
  <summary>3 Spring Data REST + HATEOAS</summary>

Ссылки: 

Commit: 
</details>

