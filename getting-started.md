## Введение

* Обзор
     * Jetty для HTTP
     * Jersey для REST
     * Jackson для JSON
     * Metrics для метрик
     * И прочие библиотеки
* Установка с помощью maven
* Конфигурационные классы
* Класс приложения
* Класс представления
* Класс ресурса
     * Регистрация ресурса в приложении
* Создание Health Check класса
     * Регистрация health check класса в приложении
* Сбор проекта в JAR
     * Версионнирование ваших JAR
* Запуск Вашего приложения

## Обзор

Обзор проведет вас через процесс создания просто HelloWorld приложения на DropWizard.
По ходу мы рассмотрим лежащие в основе dropwizard библиотеки, важные концепции
и рассмотрим некоторые архитектурные возможности которые помогут Вам при росте
вашего приложения.

DropWizard колеблется между библиотекой и фреймворком. Целью DropWizard является обеспечение
высокопроизводительную, надежную реализацию приложения готового к продакшену и высоким нагрузкам.

#### Jetty для HTTP

WEB приложение не может существовать без HTTP по-этому DropWizard использует библиотеку Jetty для
HTTP, чтобы встроить оттюненый сервер непосредственно в ваш проект.

Вместо того чтобы серверить приложение на Application серверах, DropWizard использует метод
`main` как точку входа для того, чтобы запустить HTTP сервер.

Запуск сервера внутри приложения устраняет ряд неприятных проблем.

- Нет проблем PermGen
- Никакой конфигурации и обслуживания Application серверов
- Никаких разверточных средств
- Никаких настроек GC для работы с несколькими экземплярами приложений.

и пользоваляет использовать все существующие инструменты управления процессом в Unix.

#### Jersey для REST

Мы обнаружили что для построения RestFul приложений ничто не сравнится с Jersey (эталонная реализация JAX-RS) с
точки зрения характеристик и производительности. Это позволяет писать чистые, тестируемые
классы которые изящно отображают HTTP запросы к простым Java объектам. Jersey поддерживает
потоковый response, матричные URI параметры, условные GET запросы и многое - многое другое.

#### Jackson для JSON

С точки зрения форматов данных JSON стал очень популярным и Jackson является королем
JSON'а на виртуальной машине Java. В добавлении к тому что Jackson быстр, он так же
имеет сложный объект для сопоставления JSON структуры с POJO, это значит что вы можете получать свои POJO объекты
прямо из запроса.

#### Metrics для метрик

[Metrics](http://metrics.dropwizard.io/) библиотека которая позволяет вести метрики вашего кода и предоставляет полную
информацию о его выполнении.

#### Прочие библиотеки

В дополнении к Jetty, Jersey и Jackson DropWizard включет в себя ряд дополнительных библиотек которые положительно
влияют на перфоманс вашего приложения.

- [Guava](https://github.com/google/guava) это утилитарная библиотека которая привносит новые структуры данных в Java
- [Logback](http://logback.qos.ch/) и [slf4j](http://www.slf4j.org/) быстрое и гибкое логирование
- [Hibernate Validator](http://www.hibernate.org/subprojects/validator.html) эталонная реализация JSR-349 обеспечивает простой декларативный
интерфейс для валидации данных которые вводит пользователь
- [Apache HTTP Client](http://hc.apache.org/httpcomponents-client-ga/index.html) и [Jersey](http://jersey.java.net/) клиенты которые
помогают писать простой интерфейс к другим Web сервисам.
- [JDBI](http://www.jdbi.org/) это самый простой способ использовать РСУБД в Java.
- [Liquibase](http://www.liquibase.org/) Библиотека для написания миграций.
- [Freemarker](http://freemarker.sourceforge.net/) и [Mustache](http://mustache.github.io/) простые библиотеки для работы с вьюхами.
- [Joda Time](http://joda-time.sourceforge.net/) библиотека с большими возможностями для работы с датой и временем в Java.

## Установка с помощью maven

Мы рекомендуем использовать [Maven](http://maven.apache.org/), если вы используете 
[Ant](http://ant.apache.org/)/[Ivy](http://ant.apache.org/ivy/), [Buildr](http://buildr.apache.org/),
[Gradle](http://www.gradle.org/), [SBT](https://github.com/harrah/xsbt/wiki), [Leiningen](https://github.com/technomancy/leiningen)
или [Gant](https://github.com/Gant/Gant), да, это круто. Но теперь будете использовать maven :)

У вас есть три варианта развертки первого проекта

- Создать проект используя [dropwizard-archetype](https://github.com/dropwizard/dropwizard/tree/master/dropwizard-archetypes)
```bash
mvn archetype:generate -DarchetypeGroupId=io.dropwizard.archetypes -DarchetypeArtifactId=java-simple -DarchetypeVersion=0.9.1
```
- Посмотреть и склонировать [dropwizard-example](https://github.com/dropwizard/dropwizard/tree/master/dropwizard-example)
- Пройти следующий туториал который описан далее

#### Установка DropWizard с помощью Maven

Сначала добавьте `dropwizard.version` в ваш `POM.xml` (сейчас актуальная версия 1.0.6)

```xml
<properties>
    <dropwizard.version>1.0.6</dropwizard.version>
</properties>
```

А теперь добавьте `dropwizard-core` как зависимость вашего проекта

```xml
<dependencies>
    <dependency>
        <groupId>io.dropwizard</groupId>
        <artifactId>dropwizard-core</artifactId>
        <version>${dropwizard.version}</version>
    </dependency>
</dependencies>
```