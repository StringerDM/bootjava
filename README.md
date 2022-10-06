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
    -   Подключаем зависимости:
    -   Lombock
    -   Spring Web
    -   H2 database
    -   Spring Data JPA

    По умолчанию приложение открывается по адресу localhost:8080

    Ссылки: 
    Spring Initializrs: https://start.spring.io/

    Commit: https://github.com/StringerDM/bootjava/commit/35a21d499357b464ebb5b571cb97ac0bc5e57f01

    1.2	Spring Boot maven plugin. Конвертация в WAR
  
    Ссылки:
    Конвертация JAR приложения в WAR http://spring-projects.ru/guides/convert-jar-to-war-maven/
  
    1.3	Настройка проекта
    Готовый проект с патчами находится в ветке patched:   git clone --branch patched https://github.com/JavaOPs/bootjava.git

    1.4 Проект Lombok
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

    Commit: https://github.com/StringerDM/bootjava/commit/ef6cdb5d5fb182bf1387e77206ddf174ce4ed005 

</details>

<details>
  <summary>2. Работа с DB (H2, Spring Data JPA)</summary>
    2.1	Spring Data JPA. ApplicationRunner
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
    
    C spring boot v2.3 убрали валидацию по умолчанию, поэтому добавили в pom.xml:
    
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>
    
    Далее определяем интерфейс userRepository extends JpaRepository<User, Integer>. Имплементация по умолчанию JpaRepository это класс SimpleJpaRepository, сбда можно брейк поинты ставить для дебага.
    
    В aplication.property сделаем одну настройку (Common application Data properties - https://docs.spring.io/spring-boot/docs/current/reference/html/appendix-application-properties.html#data-properties все настройки spring boot и по ключевому слов JPA мы можем найти все конфигурационный классы и что можно объявлять):
    
    spring.jpa.show-sql=true - для отображения запросов в базу.
    
    Запускаем приложение и смотрим как наша таблица создается. По умолчанию для embedded БД таблицы сначало дропаются, затем создается общий для всех hibernate siquence и создаются таблицы.
    
    Зделаем сначало заполнение таблиц програмно. В spring boot есть 2 интерфейса ApplicationRunner and CommandLineRunner которые позволяют выполнять произвольный код после старта приложения. Разница между ними в том что ApplicationRunner мы принимае массив аргументов обернутый в класс который позовляет нам выполнять какието удобные вещи например getOptional value. Реализовывать интерфейсы можно в любом из бинов spring, мы реализуем его в главном RestaurantVotingApplication:
    
    //реализуем интерфейс ApplicationRunner
    @SpringBootApplication
    @AllArgsConstructor
    public class RestaurantVotingApplication implements ApplicationRunner {
    
    //инжектим userRepository через аннотацию @AllArgsConstructor
    private final UserRepository userRepository;
    
    Ссылки: :
    

    Commit: https://github.com/StringerDM/bootjava/commit/530474b5f8ac9f85dd89284476fcb42685cb7aba
    
</details>
<details>
  <summary>1 Основы Spring Boot</summary>

Ссылки: 

Commit: 
</details>
