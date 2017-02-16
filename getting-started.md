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

Сначала добавьте `dropwizard.version` в ваш `pom.xml` (сейчас актуальная версия 1.0.6)

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

Этого достаточно чтобы установить `dropwizard`. Теперь давайте приступим
к написанию реального кода.

## Создание конфигурационного файла

Каждое DropWizard приложение должно иметь конфигурационный класс который является суб классом от
`Configuration` класса, который хранит специфические параметры вашего приложения. Эти параметры описываются
в YAML конфигурационном файле который десериализуется в инстанс объекта вашего конфигурационного файла и валидируется.
 
Приложение которое мы будем строить это высокопроизводительный сервис HelloWorld и одним из наших требований
является то, что нам нужно иметь возможность указать имя по-умолчанию и формат сообщений которые мы будем отдавать
клиенту.

Вот пример класса конфигурации, полный пример [здесь](https://github.com/dropwizard/dropwizard/blob/master/dropwizard-example/src/main/java/com/example/helloworld/HelloWorldConfiguration.java)

```java
package com.example.helloworld;

import io.dropwizard.Configuration;
import com.fasterxml.jackson.annotation.JsonProperty;
import org.hibernate.validator.constraints.NotEmpty;

public class HelloWorldConfiguration extends Configuration {
    @NotEmpty
    private String template;

    @NotEmpty
    private String defaultName;

    @JsonProperty
    public String getTemplate() {
        return template;
    }

    @JsonProperty
    public void setTemplate(String template) {
        this.template = template;
    }

    @JsonProperty
    public String getDefaultName() {
        return defaultName;
    }

    @JsonProperty
    public void setDefaultName(String name) {
        this.defaultName = name;
    }
}
```

здесь очень много происходит, давайте разберемся по порядку.

Когда этот класс десериализуется из `YAML` файла конфигурации он будет вытягивать
как минимум два поля `template` - это шаблон для нашего hello world и `defaultName` - это
имя по-умолчанию которое будет возвращаться клиенту когда имя не передано как query parameter.

`template` и `defaultName` помечены аннотацией `@NotEmpty` так что если конфигурационный файл
`YAML` не содержит этих полей или они имеют пустые значения, то будет сгенерировано исключение при запуске
приложения и приложение не будет запущено.

И методы геттеры и методы сеттеры для `template` и `defaultName` помечаются аннотацией `@JsonProperty`, это делается для того, чтобы
файл конфигурации можно было не только десерриализовать из `yaml` файла, но и сериализовать обратно.

> **Заметка:**
> Отображение `yaml` файла конфигурации в ваш объект конфигурации делает jackson, это значит что вы можете использовать
> все аннотации jackson в своих классах конфигурации для десерриализации. А аннотацию `@NotEmpty` обрабатывает
> Hibernate Validator

Наш `yaml` файл конфигурации будет выглядеть следующим образом (полный пример можете
посмотреть [здесь](https://github.com/dropwizard/dropwizard/blob/master/dropwizard-example/example.yml))

```yaml
template: Hello, %s!
defaultName: Stranger
```

DropWizard имеет множество параметров конфигурации, больше чем приведено выше, но все они имеют
значения по-умолчанию, это значит что вы не обязаны определять их в своих файлах конфигурации и 
оставить свои `yaml` файлы конфигурации простыми и чистыми.

Хорошо, сохраните теперь этот файл конфигурации с именем `hello-world.yaml` в корне проекта, мы будем работать с
этим файлом довольно быстро. Далее мы создадим класс приложения.

## Создание класса приложения

Ваше DropWizard приложение состоит минимум из двух классов. Это класс конфигурации который
расширяет `Configuration` класс и класс приложения который расширяет класс `Application<T extends Configuration>`.

Класс приложения обеспечивает связку всех компонентов вашего приложения (но об этом позже) наш `HelloWorldApplication`
класс будет выглядеть следующим образом.

```java
package com.example.helloworld;

import io.dropwizard.Application;
import io.dropwizard.setup.Bootstrap;
import io.dropwizard.setup.Environment;
import com.example.helloworld.resources.HelloWorldResource;
import com.example.helloworld.health.TemplateHealthCheck;

public class HelloWorldApplication extends Application<HelloWorldConfiguration> {
    public static void main(String[] args) throws Exception {
        new HelloWorldApplication().run(args);
    }

    @Override
    public String getName() {
        return "hello-world";
    }

    @Override
    public void initialize(Bootstrap<HelloWorldConfiguration> bootstrap) {
        // nothing to do yet
    }

    @Override
    public void run(HelloWorldConfiguration configuration,
                    Environment environment) {
        // nothing to do yet
    }

}
```

> **Замечание от переводчика**: 
> Из обязательных методов здесь является только метод run(), остальные переопределяются
> по желанию.

Как вы могли заметить класс приложения параметризирован классом конфигурации. Метод `initialize()`
используется для настройки аспектов приложения которые настраиваются перед запуском приложения, например
таких как: бандлы, поставщики конфигураций, комманды, таски и так далее.

Так же мы добавили статический метод `main(String[] args)` который будет точкой входа вашего
приложения. На данный момент наше приложение не несет никакой функциональности, по-этому метод `run` у нас пустой
и скучный. Давайте исправим это!

## Создание класса представления

Прежде чем наше приложение попадет в использование нам нужно разработать его `api`. К счастью наше приложение соответствует 
стандарту [RFC 1149](http://www.ietf.org/rfc/rfc1149.txt), который определяет следующий JSON формат.

```json
{
  "id": 1,
  "content": "Hi!"
}
```

Поле `id` является уникальным идентификатором и `content` это текстовое представление высказывания.

Давайте теперь смоделируем наше представление.

```java
package com.example.helloworld.api;

import com.fasterxml.jackson.annotation.JsonProperty;
import org.hibernate.validator.constraints.Length;

public class Saying {
    private long id;

    @Length(max = 3)
    private String content;

    public Saying() {
        // Jackson deserialization
    }

    public Saying(long id, String content) {
        this.id = id;
        this.content = content;
    }

    @JsonProperty
    public long getId() {
        return id;
    }

    @JsonProperty
    public String getContent() {
        return content;
    }
}
```

Это довольно простой POJO, но стоит рассмотреть несколько особенностей.

Во-первых он иммутабелен, это очень легко и безопасно использовать в многопоточных и однопоточных системах.
Во-вторых здесь используется стандарт Java Beans для свойств `id` и `content`. Это позволяет
сериализовать этот объект в JSON так как нам нужно. Jackson заполнит поле `id` в json вызвав геттер `#getId()` для
получения значения id, то же самое правило распространяется для `content`. Ну и бин используется
аннотацию `@Length` для ограничения длины строки `content`, это Hibernate Validation аннотация.

> **Заметка**:
> Для сериализации в JSON здесь используется Jackson, которые поддерживает более широкие возможности
> чем сериализация обычных Java Bean объектов. В дополнении к специальным аннотациям вы можете даже
> написать свои сериализаторы и десерриализаторы.

Теперь когда у нас есть класс представления, то теперь пора сделать класс ресурса для этого класса.

## Создание класса ресурса

Jersey ресурсы являются центром DropWizard приложения. Каждый класс ресурса
ассоциируется с определенным URI шаблоном. В нашем приложении нам нужен ресурс
который возвращает экземпляр класса `Saying` по url `/hello-world` так будет выглядеть
наш resource класс.

```java
package com.example.helloworld.resources;

import com.example.helloworld.api.Saying;
import com.codahale.metrics.annotation.Timed;

import javax.ws.rs.GET;
import javax.ws.rs.Path;
import javax.ws.rs.Produces;
import javax.ws.rs.QueryParam;
import javax.ws.rs.core.MediaType;
import java.util.concurrent.atomic.AtomicLong;
import java.util.Optional;

@Path("/hello-world")
@Produces(MediaType.APPLICATION_JSON)
public class HelloWorldResource {
    private final String template;
    private final String defaultName;
    private final AtomicLong counter;

    public HelloWorldResource(String template, String defaultName) {
        this.template = template;
        this.defaultName = defaultName;
        this.counter = new AtomicLong();
    }

    @GET
    @Timed
    public Saying sayHello(@QueryParam("name") Optional<String> name) {
        final String value = String.format(template, name.orElse(defaultName));
        return new Saying(counter.incrementAndGet(), value);
    }
}
```

`HelloWorldResource` имеет две анотации. `@Path` и `@Produces`.
`@Path("/hello-world")` сообщает Jersey что этот ресурс должен быть
доступен по URI `/hello-world` и `@Produces(MediaType.APPLICATION_JSON)`
сообщает Jersey что наш ресурс отдает ответ в виде JSON документов.

`HelloWorldResource` получает два параметра в конструктор: `template` этот шаблон
используется для конструирования фразы и `defaultName` используется когда
пользователь отказывается сообщать нам свое имя. А
`AtomicLong` дает нам дешевый потокобезопасный способ получать уникальные
id для наших записей.

> **Предупреждение**:
> Классы ресурсов исполняются в многопоточной среде. В общем мы рекомендуем
> делать классы ресурсов без состояния или иммутабельными. Но очень важно сохранять
> контекст