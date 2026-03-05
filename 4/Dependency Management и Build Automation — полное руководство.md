## 1. Что такое Dependency Management

### Определение

**Dependency Management Tool (Инструмент управления зависимостями)** — это система, которая автоматизирует процесс загрузки, обновления и управления внешними библиотеками (зависимостями), необходимыми для работы вашего приложения.

text

```
╔═══════════════════════════════════════════════════════════════════╗
║              ЧТО ТАКОЕ ЗАВИСИМОСТИ (DEPENDENCIES)?                 ║
╠═══════════════════════════════════════════════════════════════════╣
║                                                                    ║
║  ЗАВИСИМОСТЬ — это внешняя библиотека, фреймворк или модуль,     ║
║  который использует ваше приложение.                              ║
║                                                                    ║
║  ПРИМЕРЫ:                                                          ║
║                                                                    ║
║  Java проект:                                                      ║
║  ├── Spring Boot (фреймворк)                                      ║
║  ├── Jackson (JSON парсинг)                                       ║
║  ├── PostgreSQL Driver (JDBC драйвер)                             ║
║  ├── JUnit (тестирование)                                         ║
║  └── Lombok (генерация кода)                                      ║
║                                                                    ║
║  Node.js проект:                                                   ║
║  ├── Express (веб-фреймворк)                                      ║
║  ├── React (UI библиотека)                                        ║
║  ├── Axios (HTTP клиент)                                          ║
║  ├── Jest (тестирование)                                          ║
║  └── TypeScript (язык)                                            ║
║                                                                    ║
║  Python проект:                                                    ║
║  ├── Django (фреймворк)                                           ║
║  ├── Requests (HTTP библиотека)                                   ║
║  ├── Pandas (обработка данных)                                    ║
║  ├── Pytest (тестирование)                                        ║
║  └── SQLAlchemy (ORM)                                             ║
║                                                                    ║
╠═══════════════════════════════════════════════════════════════════╣
║                                                                    ║
║  ТИПЫ ЗАВИСИМОСТЕЙ:                                                ║
║                                                                    ║
║  1️⃣  ПРЯМЫЕ (Direct)                                              ║
║     Библиотеки, которые вы явно добавили в проект                ║
║     Пример: Express в package.json                                ║
║                                                                    ║
║  2️⃣  ТРАНЗИТИВНЫЕ (Transitive)                                    ║
║     Зависимости ваших зависимостей                                ║
║     Пример: Express зависит от body-parser → body-parser          ║
║              становится транзитивной зависимостью                 ║
║                                                                    ║
║  3️⃣  ЗАВИСИМОСТИ РАЗРАБОТКИ (Dev Dependencies)                    ║
║     Нужны только при разработке, не в production                  ║
║     Пример: тесты, линтеры, инструменты сборки                    ║
║                                                                    ║
║  4️⃣  ЗАВИСИМОСТИ ВРЕМЕНИ ВЫПОЛНЕНИЯ (Runtime)                     ║
║     Нужны для работы приложения в production                      ║
║     Пример: веб-фреймворки, драйверы БД                          ║
║                                                                    ║
╚═══════════════════════════════════════════════════════════════════╝
```

### Проблемы без Dependency Management

text

```
┌─────────────────────────────────────────────────────────────────────┐
│          БЕЗ DEPENDENCY MANAGEMENT (давным-давно)                    │
└─────────────────────────────────────────────────────────────────────┘

Шаг 1: Найти библиотеку
  ├─ Поиск в Google "Java JSON parser"
  ├─ Зайти на сайт jackson-databind
  └─ Найти раздел Downloads

Шаг 2: Скачать JAR файл
  ├─ jackson-databind-2.15.0.jar
  └─ Сохранить в проект

Шаг 3: Найти зависимости библиотеки
  ├─ jackson-databind требует jackson-core
  ├─ jackson-databind требует jackson-annotations
  └─ Повторить шаги 1-2 для каждой

Шаг 4: Добавить в проект
  ├─ Скопировать все JAR в папку /lib
  └─ Добавить в classpath вручную

Шаг 5: Обновить версию
  ├─ Удалить старые JAR
  ├─ Повторить всё сначала
  └─ Проверить совместимость

ПРОБЛЕМЫ:
❌ Ручная работа
❌ Версионные конфликты (dependency hell)
❌ Сложно отследить что откуда
❌ Нет автоматического обновления
❌ Много места (дублирование JAR)
❌ Невозможно поделиться проектом (JAR в Git?)


┌─────────────────────────────────────────────────────────────────────┐
│          С DEPENDENCY MANAGEMENT (современность)                     │
└─────────────────────────────────────────────────────────────────────┘

Шаг 1: Добавить зависимость в файл конфигурации
  
  # Maven (pom.xml)
  <dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.15.0</version>
  </dependency>

  # или npm (package.json)
  "dependencies": {
    "express": "^4.18.0"
  }

Шаг 2: Запустить команду
  mvn install
  # или
  npm install

Шаг 3: Всё!
  ✅ Автоматически скачиваются все зависимости
  ✅ Автоматически разрешаются транзитивные зависимости
  ✅ Кэшируются локально (~/.m2 или ~/.npm)
  ✅ Версии фиксируются в lock-файлах
  ✅ Легко обновить: изменить версию и пересобрать
  ✅ Можно поделиться: только файл конфигурации в Git
```

### Функции Dependency Management Tools

text

```
╔═══════════════════════════════════════════════════════════════════╗
║           ФУНКЦИИ DEPENDENCY MANAGEMENT TOOLS                      ║
╠═══════════════════════════════════════════════════════════════════╣
║                                                                    ║
║  1️⃣  ЗАГРУЗКА ЗАВИСИМОСТЕЙ                                        ║
║     • Автоматическое скачивание из репозиториев                   ║
║     • Кэширование локально                                        ║
║     • Поддержка приватных репозиториев                            ║
║                                                                    ║
║  2️⃣  РАЗРЕШЕНИЕ ВЕРСИЙ                                            ║
║     • Выбор правильной версии при конфликтах                      ║
║     • Семантическое версионирование (SemVer)                      ║
║     • Транзитивные зависимости                                    ║
║                                                                    ║
║  3️⃣  ИЗОЛЯЦИЯ ОКРУЖЕНИЙ                                           ║
║     • Отдельные зависимости для dev/test/prod                     ║
║     • Виртуальные окружения                                       ║
║                                                                    ║
║  4️⃣  LOCK-ФАЙЛЫ                                                   ║
║     • Фиксация точных версий                                      ║
║     • Воспроизводимые сборки                                      ║
║     • package-lock.json, Gemfile.lock, poetry.lock                ║
║                                                                    ║
║  5️⃣  АУДИТ БЕЗОПАСНОСТИ                                           ║
║     • Проверка уязвимостей                                        ║
║     • Автоматические обновления (Dependabot)                      ║
║                                                                    ║
║  6️⃣  УПРАВЛЕНИЕ ОБНОВЛЕНИЯМИ                                      ║
║     • Проверка доступных обновлений                               ║
║     • Batch обновления                                            ║
║                                                                    ║
╚═══════════════════════════════════════════════════════════════════╝
```

---

## 2. Maven

### Что такое Maven

text

```
╔═══════════════════════════════════════════════════════════════════╗
║                           APACHE MAVEN                             ║
╠═══════════════════════════════════════════════════════════════════╣
║                                                                    ║
║  Maven — это инструмент для автоматизации сборки и управления     ║
║  зависимостями для Java проектов.                                 ║
║                                                                    ║
║  Основан на концепции POM (Project Object Model)                  ║
║                                                                    ║
║  Год создания: 2004                                                ║
║  Организация: Apache Software Foundation                           ║
║  Конфигурация: XML (pom.xml)                                      ║
║  Репозиторий: Maven Central Repository                             ║
║                                                                    ║
╠═══════════════════════════════════════════════════════════════════╣
║                                                                    ║
║  КЛЮЧЕВЫЕ КОНЦЕПЦИИ:                                               ║
║                                                                    ║
║  1️⃣  POM (Project Object Model)                                   ║
║     XML файл с описанием проекта и зависимостей                   ║
║                                                                    ║
║  2️⃣  КООРДИНАТЫ (GAV)                                             ║
║     GroupId + ArtifactId + Version = уникальная идентификация     ║
║                                                                    ║
║  3️⃣  LIFECYCLE (Жизненный цикл)                                   ║
║     Фиксированные фазы сборки (validate, compile, test, etc.)     ║
║                                                                    ║
║  4️⃣  PLUGINS (Плагины)                                            ║
║     Расширения функциональности                                   ║
║                                                                    ║
║  5️⃣  REPOSITORIES (Репозитории)                                   ║
║     Откуда скачивать зависимости                                  ║
║                                                                    ║
╚═══════════════════════════════════════════════════════════════════╝
```

### Структура pom.xml

XML

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    
    <!-- ═══════════════════════════════════════════════════════════ -->
    <!-- БАЗОВАЯ ИНФОРМАЦИЯ О ПРОЕКТЕ                                -->
    <!-- ═══════════════════════════════════════════════════════════ -->
    
    <!-- Версия формата POM -->
    <modelVersion>4.0.0</modelVersion>
    
    <!-- Координаты проекта (GAV) -->
    <!-- GroupId — обычно обратное доменное имя организации -->
    <groupId>com.example</groupId>
    
    <!-- ArtifactId — имя проекта/модуля -->
    <artifactId>my-application</artifactId>
    
    <!-- Версия проекта -->
    <version>1.0.0-SNAPSHOT</version>
    
    <!-- Тип упаковки: jar (по умолчанию), war, pom, ear, etc. -->
    <packaging>jar</packaging>
    
    <!-- Читаемое имя -->
    <name>My Application</name>
    
    <!-- Описание -->
    <description>Example Maven project</description>
    
    
    <!-- ═══════════════════════════════════════════════════════════ -->
    <!-- СВОЙСТВА (PROPERTIES)                                        -->
    <!-- ═══════════════════════════════════════════════════════════ -->
    
    <properties>
        <!-- Версия Java -->
        <java.version>17</java.version>
        
        <!-- Кодировка исходников -->
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        
        <!-- Версии зависимостей (централизованно) -->
        <spring.boot.version>3.1.0</spring.boot.version>
        <junit.version>5.9.3</junit.version>
        <lombok.version>1.18.28</lombok.version>
    </properties>
    
    
    <!-- ═══════════════════════════════════════════════════════════ -->
    <!-- РОДИТЕЛЬСКИЙ POM (опционально)                              -->
    <!-- ═══════════════════════════════════════════════════════════ -->
    
    <!-- Наследование от другого POM -->
    <!-- Например, Spring Boot предоставляет parent POM -->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.1.0</version>
        <relativePath/>
    </parent>
    
    
    <!-- ═══════════════════════════════════════════════════════════ -->
    <!-- ЗАВИСИМОСТИ (DEPENDENCIES)                                  -->
    <!-- ═══════════════════════════════════════════════════════════ -->
    
    <dependencies>
        
        <!-- Spring Boot Web Starter -->
        <dependency>
            <!-- Группа (организация) -->
            <groupId>org.springframework.boot</groupId>
            
            <!-- Артефакт (библиотека) -->
            <artifactId>spring-boot-starter-web</artifactId>
            
            <!-- Версия (можно не указывать если есть parent) -->
            <version>${spring.boot.version}</version>
            
            <!-- Scope — где используется зависимость -->
            <!-- compile (по умолчанию) — везде -->
            <!-- test — только в тестах -->
            <!-- provided — предоставляется runtime (servlet API) -->
            <!-- runtime — только при выполнении (JDBC драйверы) -->
            <scope>compile</scope>
        </dependency>
        
        <!-- PostgreSQL Driver (только runtime) -->
        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <version>42.6.0</version>
            <scope>runtime</scope>
        </dependency>
        
        <!-- Lombok (compile time only) -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>${lombok.version}</version>
            <scope>provided</scope>
        </dependency>
        
        <!-- JUnit 5 (только для тестов) -->
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter</artifactId>
            <version>${junit.version}</version>
            <scope>test</scope>
        </dependency>
        
        <!-- Исключение транзитивных зависимостей -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-logging</artifactId>
            <exclusions>
                <!-- Исключаем Logback, используем Log4j2 -->
                <exclusion>
                    <groupId>ch.qos.logback</groupId>
                    <artifactId>logback-classic</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        
    </dependencies>
    
    
    <!-- ═══════════════════════════════════════════════════════════ -->
    <!-- DEPENDENCY MANAGEMENT (управление версиями)                 -->
    <!-- ═══════════════════════════════════════════════════════════ -->
    
    <!-- Не добавляет зависимости, только управляет версиями -->
    <!-- Полезно в multi-module проектах -->
    <dependencyManagement>
        <dependencies>
            <!-- BOM (Bill of Materials) — импорт набора версий -->
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>2022.0.3</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
    
    
    <!-- ═══════════════════════════════════════════════════════════ -->
    <!-- BUILD (конфигурация сборки)                                 -->
    <!-- ═══════════════════════════════════════════════════════════ -->
    
    <build>
        <!-- Имя финального JAR файла -->
        <finalName>my-application</finalName>
        
        <!-- Плагины -->
        <plugins>
            
            <!-- Compiler Plugin — компиляция Java -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.11.0</version>
                <configuration>
                    <source>${java.version}</source>
                    <target>${java.version}</target>
                </configuration>
            </plugin>
            
            <!-- Surefire Plugin — запуск тестов -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>3.1.2</version>
            </plugin>
            
            <!-- Spring Boot Plugin — создание executable JAR -->
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <version>${spring.boot.version}</version>
                <executions>
                    <execution>
                        <goals>
                            <goal>repackage</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
            
            <!-- JAR Plugin — упаковка в JAR -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-jar-plugin</artifactId>
                <version>3.3.0</version>
                <configuration>
                    <archive>
                        <manifest>
                            <!-- Main класс для запуска -->
                            <mainClass>com.example.Application</mainClass>
                        </manifest>
                    </archive>
                </configuration>
            </plugin>
            
            <!-- JaCoCo — code coverage -->
            <plugin>
                <groupId>org.jacoco</groupId>
                <artifactId>jacoco-maven-plugin</artifactId>
                <version>0.8.10</version>
                <executions>
                    <execution>
                        <goals>
                            <goal>prepare-agent</goal>
                        </goals>
                    </execution>
                    <execution>
                        <id>report</id>
                        <phase>test</phase>
                        <goals>
                            <goal>report</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
            
        </plugins>
    </build>
    
    
    <!-- ═══════════════════════════════════════════════════════════ -->
    <!-- REPOSITORIES (откуда скачивать зависимости)                 -->
    <!-- ═══════════════════════════════════════════════════════════ -->
    
    <repositories>
        <!-- Maven Central (по умолчанию, можно не указывать) -->
        <repository>
            <id>central</id>
            <url>https://repo.maven.apache.org/maven2</url>
        </repository>
        
        <!-- Приватный репозиторий (Nexus, Artifactory) -->
        <repository>
            <id>company-releases</id>
            <url>https://nexus.company.com/repository/maven-releases/</url>
        </repository>
    </repositories>
    
    
    <!-- ═══════════════════════════════════════════════════════════ -->
    <!-- PROFILES (профили для разных окружений)                     -->
    <!-- ═══════════════════════════════════════════════════════════ -->
    
    <profiles>
        <!-- Development профиль -->
        <profile>
            <id>dev</id>
            <activation>
                <!-- Активируется по умолчанию -->
                <activeByDefault>true</activeByDefault>
            </activation>
            <properties>
                <env>dev</env>
            </properties>
        </profile>
        
        <!-- Production профиль -->
        <profile>
            <id>prod</id>
            <properties>
                <env>prod</env>
            </properties>
            <build>
                <plugins>
                    <!-- Минификация в production -->
                    <plugin>
                        <groupId>com.github.eirslett</groupId>
                        <artifactId>frontend-maven-plugin</artifactId>
                        <configuration>
                            <minify>true</minify>
                        </configuration>
                    </plugin>
                </plugins>
            </build>
        </profile>
    </profiles>

</project>
```

### Maven Lifecycle

text

```
┌─────────────────────────────────────────────────────────────────────┐
│                      MAVEN BUILD LIFECYCLE                           │
└─────────────────────────────────────────────────────────────────────┘

Maven имеет 3 встроенных lifecycle:
1. default — основная сборка
2. clean — очистка
3. site — генерация документации


DEFAULT LIFECYCLE (фазы выполняются последовательно):
────────────────────────────────────────────────────────

1. validate
   ├─ Проверка что проект корректен
   └─ Вся необходимая информация доступна

2. compile
   ├─ Компиляция исходного кода
   ├─ src/main/java → target/classes
   └─ Обработка ресурсов (src/main/resources)

3. test
   ├─ Запуск unit тестов
   ├─ src/test/java компилируется
   ├─ Тесты выполняются через Surefire Plugin
   └─ Результаты в target/surefire-reports

4. package
   ├─ Упаковка скомпилированного кода
   ├─ JAR/WAR/EAR в зависимости от <packaging>
   └─ Результат: target/my-app-1.0.0.jar

5. verify
   ├─ Проверки качества
   ├─ Интеграционные тесты
   └─ Code coverage проверки

6. install
   ├─ Установка артефакта в локальный репозиторий
   └─ ~/.m2/repository/

7. deploy
   ├─ Публикация в удалённый репозиторий
   └─ Nexus, Artifactory, etc.


CLEAN LIFECYCLE:
────────────────

pre-clean → clean → post-clean
            │
            └─ Удаляет target/ директорию


КОМАНДЫ:
────────

# Очистка
mvn clean

# Компиляция
mvn compile

# Тесты
mvn test

# Упаковка (+ все предыдущие фазы)
mvn package

# Установка в локальный repo
mvn install

# Публикация в remote repo
mvn deploy

# Комбинации (часто используемые)
mvn clean install       # Очистка + сборка + установка
mvn clean package       # Очистка + сборка
mvn clean verify        # Очистка + сборка + проверки

# С пропуском тестов (не рекомендуется!)
mvn clean package -DskipTests

# С профилем
mvn clean package -P prod
```

### Основные команды Maven

Bash

```
# ═══════════════════════════════════════════════════════════
#                    ОСНОВНЫЕ КОМАНДЫ MAVEN
# ═══════════════════════════════════════════════════════════

# Создать новый проект из архетипа
mvn archetype:generate \
  -DgroupId=com.example \
  -DartifactId=my-app \
  -DarchetypeArtifactId=maven-archetype-quickstart \
  -DinteractiveMode=false

# Очистка проекта
mvn clean

# Компиляция
mvn compile

# Запуск тестов
mvn test

# Упаковка в JAR/WAR
mvn package

# Установка в локальный репозиторий (~/.m2/repository)
mvn install

# Деплой в remote репозиторий
mvn deploy

# ═══════════════════════════════════════════════════════════
#                    УПРАВЛЕНИЕ ЗАВИСИМОСТЯМИ
# ═══════════════════════════════════════════════════════════

# Показать дерево зависимостей
mvn dependency:tree

# Проанализировать зависимости
mvn dependency:analyze

# Показать эффективный POM (с учётом parent и profiles)
mvn help:effective-pom

# Скачать зависимости (без сборки)
mvn dependency:resolve

# Скачать исходники зависимостей
mvn dependency:sources

# Проверить обновления зависимостей
mvn versions:display-dependency-updates

# ═══════════════════════════════════════════════════════════
#                    ПРОФИЛИ
# ═══════════════════════════════════════════════════════════

# Сборка с профилем
mvn clean package -P prod

# Несколько профилей
mvn clean package -P prod,mysql

# Посмотреть активные профили
mvn help:active-profiles

# ═══════════════════════════════════════════════════════════
#                    ПЛАГИНЫ
# ═══════════════════════════════════════════════════════════

# Запуск Spring Boot приложения
mvn spring-boot:run

# Генерация документации JavaDoc
mvn javadoc:javadoc

# Code coverage (JaCoCo)
mvn jacoco:report

# Обновить версию проекта
mvn versions:set -DnewVersion=2.0.0

# ═══════════════════════════════════════════════════════════
#                    ДОПОЛНИТЕЛЬНЫЕ ОПЦИИ
# ═══════════════════════════════════════════════════════════

# Offline режим (не обращаться к remote репозиториям)
mvn -o clean package

# Debug режим
mvn -X clean package

# Пропустить тесты
mvn clean package -DskipTests

# Пропустить компиляцию тестов
mvn clean package -Dmaven.test.skip=true

# Запуск в многопоточном режиме
mvn -T 4 clean package     # 4 потока
mvn -T 1C clean package    # 1 поток на CPU ядро

# Установить системное свойство
mvn clean package -Dspring.profiles.active=prod

# Использовать альтернативный settings.xml
mvn clean package -s /path/to/settings.xml
```

---

## 3. Gradle

### Что такое Gradle

text

```
╔═══════════════════════════════════════════════════════════════════╗
║                            GRADLE                                  ║
╠═══════════════════════════════════════════════════════════════════╣
║                                                                    ║
║  Gradle — современный инструмент автоматизации сборки,            ║
║  использующий Groovy/Kotlin DSL вместо XML.                       ║
║                                                                    ║
║  Год создания: 2007                                                ║
║  Конфигурация: build.gradle (Groovy) или build.gradle.kts (Kotlin)║
║  Репозитории: Maven Central, JCenter (deprecated), Google         ║
║                                                                    ║
╠═══════════════════════════════════════════════════════════════════╣
║                                                                    ║
║  КЛЮЧЕВЫЕ ОСОБЕННОСТИ:                                             ║
║                                                                    ║
║  1️⃣  DSL ВМЕСТО XML                                               ║
║     Groovy или Kotlin — более читаемо и гибко                     ║
║                                                                    ║
║  2️⃣  ИНКРЕМЕНТАЛЬНЫЕ СБОРКИ                                       ║
║     Пересобирает только измененное                                ║
║                                                                    ║
║  3️⃣  BUILD CACHE                                                  ║
║     Переиспользование результатов сборки между проектами          ║
║                                                                    ║
║  4️⃣  GRADLE WRAPPER                                               ║
║     Встроенная версия Gradle в проекте                            ║
║                                                                    ║
║  5️⃣  МНОГОЯЗЫЧНОСТЬ                                               ║
║     Java, Kotlin, Groovy, Scala, C++, Android                     ║
║                                                                    ║
║  6️⃣  ПРОИЗВОДИТЕЛЬНОСТЬ                                           ║
║     Быстрее Maven благодаря кэшированию и параллелизму            ║
║                                                                    ║
╚═══════════════════════════════════════════════════════════════════╝
```

### Структура build.gradle

groovy

```
// ═══════════════════════════════════════════════════════════
// BUILD.GRADLE (GROOVY DSL)
// ═══════════════════════════════════════════════════════════

// ─────────────────────────────────────────────────────────────
// PLUGINS (Плагины)
// ─────────────────────────────────────────────────────────────

plugins {
    // Java plugin
    id 'java'
    
    // Spring Boot plugin
    id 'org.springframework.boot' version '3.1.0'
    
    // Spring Dependency Management
    id 'io.spring.dependency-management' version '1.1.0'
    
    // Application plugin (для создания executable)
    id 'application'
}


// ─────────────────────────────────────────────────────────────
// ОСНОВНАЯ ИНФОРМАЦИЯ О ПРОЕКТЕ
// ─────────────────────────────────────────────────────────────

group = 'com.example'
version = '1.0.0-SNAPSHOT'
sourceCompatibility = '17'


// ─────────────────────────────────────────────────────────────
// КОНФИГУРАЦИЯ
// ─────────────────────────────────────────────────────────────

configurations {
    // Исключение Logback из всех зависимостей
    all {
        exclude group: 'ch.qos.logback', module: 'logback-classic'
    }
    
    // Compile time only (аналог Maven provided)
    compileOnly {
        extendsFrom annotationProcessor
    }
}


// ─────────────────────────────────────────────────────────────
// РЕПОЗИТОРИИ
// ─────────────────────────────────────────────────────────────

repositories {
    // Maven Central
    mavenCentral()
    
    // Google Maven Repository
    google()
    
    // Приватный репозиторий
    maven {
        url 'https://nexus.company.com/repository/maven-releases/'
        credentials {
            username = project.findProperty("nexusUsername") ?: System.getenv("NEXUS_USERNAME")
            password = project.findProperty("nexusPassword") ?: System.getenv("NEXUS_PASSWORD")
        }
    }
}


// ─────────────────────────────────────────────────────────────
// ЗАВИСИМОСТИ
// ─────────────────────────────────────────────────────────────

dependencies {
    // ─────────────────────────────────────────────────────────
    // COMPILE TIME (нужны для компиляции и runtime)
    // ─────────────────────────────────────────────────────────
    
    // Spring Boot Web
    implementation 'org.springframework.boot:spring-boot-starter-web'
    
    // Spring Data JPA
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    
    // Используя переменные
    def jacksonVersion = '2.15.0'
    implementation "com.fasterxml.jackson.core:jackson-databind:${jacksonVersion}"
    
    
    // ─────────────────────────────────────────────────────────
    // RUNTIME ONLY (только при выполнении)
    // ─────────────────────────────────────────────────────────
    
    // PostgreSQL Driver
    runtimeOnly 'org.postgresql:postgresql'
    
    // H2 для тестов
    runtimeOnly 'com.h2database:h2'
    
    
    // ─────────────────────────────────────────────────────────
    // COMPILE ONLY (только компиляция, не включается в JAR)
    // ─────────────────────────────────────────────────────────
    
    // Lombok
    compileOnly 'org.projectlombok:lombok:1.18.28'
    annotationProcessor 'org.projectlombok:lombok:1.18.28'
    
    
    // ─────────────────────────────────────────────────────────
    // TEST (только для тестов)
    // ─────────────────────────────────────────────────────────
    
    // JUnit 5
    testImplementation 'org.junit.jupiter:junit-jupiter:5.9.3'
    
    // Spring Boot Test
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
    
    // Mockito
    testImplementation 'org.mockito:mockito-core:5.3.1'
    testImplementation 'org.mockito:mockito-junit-jupiter:5.3.1'
    
    // AssertJ
    testImplementation 'org.assertj:assertj-core:3.24.2'
    
    // Testcontainers
    testImplementation 'org.testcontainers:testcontainers:1.18.3'
    testImplementation 'org.testcontainers:postgresql:1.18.3'
}


// ─────────────────────────────────────────────────────────────
// ЗАДАЧИ (TASKS)
// ─────────────────────────────────────────────────────────────

// Конфигурация тестов
tasks.named('test') {
    // Использовать JUnit Platform
    useJUnitPlatform()
    
    // Параллельное выполнение
    maxParallelForks = Runtime.runtime.availableProcessors().intdiv(2) ?: 1
    
    // Всегда запускать тесты (даже если ничего не изменилось)
    outputs.upToDateWhen { false }
    
    // Финализатор
    finalizedBy jacocoTestReport
}

// JaCoCo code coverage
tasks.named('jacocoTestReport') {
    dependsOn test
    
    reports {
        xml.required = true
        html.required = true
    }
}

// Кастомная задача
task hello {
    doLast {
        println 'Hello, Gradle!'
    }
}

// Задача с зависимостью
task buildDocker(type: Exec, dependsOn: build) {
    group = 'Docker'
    description = 'Build Docker image'
    
    commandLine 'docker', 'build', '-t', "myapp:${version}", '.'
}


// ─────────────────────────────────────────────────────────────
// APPLICATION PLUGIN КОНФИГУРАЦИЯ
// ─────────────────────────────────────────────────────────────

application {
    // Main класс
    mainClass = 'com.example.Application'
}


// ─────────────────────────────────────────────────────────────
// JAR КОНФИГУРАЦИЯ
// ─────────────────────────────────────────────────────────────

tasks.named('jar') {
    manifest {
        attributes(
            'Main-Class': 'com.example.Application',
            'Implementation-Title': project.name,
            'Implementation-Version': project.version
        )
    }
}


// ─────────────────────────────────────────────────────────────
// ДОПОЛНИТЕЛЬНЫЕ СВОЙСТВА
// ─────────────────────────────────────────────────────────────

// Кастомные свойства
ext {
    springCloudVersion = '2022.0.3'
}

// Использование
dependencies {
    implementation platform("org.springframework.cloud:spring-cloud-dependencies:${springCloudVersion}")
}
```

### Gradle Wrapper

Bash

```
# ═══════════════════════════════════════════════════════════
#                    GRADLE WRAPPER
# ═══════════════════════════════════════════════════════════

# Что это?
# Gradle Wrapper — скрипты, которые позволяют запускать сборку
# БЕЗ установки Gradle. Нужная версия скачивается автоматически.

# Структура проекта с wrapper:
project/
├── gradle/
│   └── wrapper/
│       ├── gradle-wrapper.jar        # Bootstrap JAR
│       └── gradle-wrapper.properties # Конфигурация (версия)
├── gradlew        # Скрипт для Linux/Mac
├── gradlew.bat    # Скрипт для Windows
└── build.gradle


# gradle-wrapper.properties
distributionBase=GRADLE_USER_HOME
distributionPath=wrapper/dists
distributionUrl=https\://services.gradle.org/distributions/gradle-8.3-bin.zip
zipStoreBase=GRADLE_USER_HOME
zipStorePath=wrapper/dists


# Команды (используйте ./gradlew вместо gradle)

# Linux/Mac
./gradlew build

# Windows
gradlew.bat build


# Генерация wrapper (если его нет)
gradle wrapper --gradle-version 8.3

# Обновление wrapper
./gradlew wrapper --gradle-version 8.4
```

### Основные команды Gradle

Bash

```
# ═══════════════════════════════════════════════════════════
#                    ОСНОВНЫЕ КОМАНДЫ GRADLE
# ═══════════════════════════════════════════════════════════

# Список всех задач
./gradlew tasks
./gradlew tasks --all  # Включая скрытые

# Информация о проекте
./gradlew projects
./gradlew properties

# ═══════════════════════════════════════════════════════════
#                    СБОРКА
# ═══════════════════════════════════════════════════════════

# Очистка
./gradlew clean

# Компиляция
./gradlew compileJava

# Компиляция тестов
./gradlew compileTestJava

# Запуск тестов
./gradlew test

# Сборка (compile + test + package)
./gradlew build

# Сборка без тестов
./gradlew build -x test

# Ассемблирование (без тестов)
./gradlew assemble

# ═══════════════════════════════════════════════════════════
#                    ЗАВИСИМОСТИ
# ═══════════════════════════════════════════════════════════

# Дерево зависимостей
./gradlew dependencies

# Дерево для конкретной конфигурации
./gradlew dependencies --configuration compileClasspath
./gradlew dependencies --configuration runtimeClasspath

# Проверка обновлений
./gradlew dependencyUpdates  # Требует плагин

# ═══════════════════════════════════════════════════════════
#                    ЗАПУСК ПРИЛОЖЕНИЯ
# ═══════════════════════════════════════════════════════════

# Запуск приложения (application plugin)
./gradlew run

# Spring Boot приложение
./gradlew bootRun

# С параметрами JVM
./gradlew bootRun --args='--spring.profiles.active=dev'

# ═══════════════════════════════════════════════════════════
#                    ПУБЛИКАЦИЯ
# ═══════════════════════════════════════════════════════════

# Публикация в локальный репозиторий Maven
./gradlew publishToMavenLocal

# Публикация в remote репозиторий
./gradlew publish

# ═══════════════════════════════════════════════════════════
#                    ПРОИЗВОДИТЕЛЬНОСТЬ
# ═══════════════════════════════════════════════════════════

# Build Scan (детальная информация о сборке)
./gradlew build --scan

# Daemon (ускоряет сборку, держит JVM в памяти)
./gradlew build --daemon      # Включить
./gradlew build --no-daemon   # Отключить

# Параллельное выполнение
./gradlew build --parallel

# Инкрементальная компиляция (по умолчанию включена)
# Компилирует только изменённые файлы

# Build Cache
./gradlew build --build-cache

# Offline режим
./gradlew build --offline

# ═══════════════════════════════════════════════════════════
#                    ОТЛАДКА
# ═══════════════════════════════════════════════════════════

# Debug логи
./gradlew build --debug

# Info логи
./gradlew build --info

# Stacktrace при ошибках
./gradlew build --stacktrace
./gradlew build --full-stacktrace

# Профилирование
./gradlew build --profile
# Создаёт HTML отчёт в build/reports/profile/

# ═══════════════════════════════════════════════════════════
#                    GRADLE.PROPERTIES
# ═══════════════════════════════════════════════════════════

# gradle.properties (глобальные настройки)

# Увеличить память для Gradle Daemon
org.gradle.jvmargs=-Xmx2g -XX:MaxMetaspaceSize=512m

# Параллельное выполнение
org.gradle.parallel=true

# Build cache
org.gradle.caching=true

# Конфигурация по требованию
org.gradle.configureondemand=true

# Daemon
org.gradle.daemon=true
```

---

## 4. Maven vs Gradle

text

```
╔═══════════════════════════════════════════════════════════════════╗
║                       MAVEN vs GRADLE                              ║
╠═══════════════════════════════════════════════════════════════════╣
║                                                                    ║
║  Критерий          │ Maven              │ Gradle                  ║
║  ──────────────────┼────────────────────┼─────────────────────────║
║  Конфигурация      │ XML (pom.xml)      │ Groovy/Kotlin DSL       ║
║  Читаемость        │ Verbose, многословно│ Компактно, лаконично   ║
║  Кривая обучения   │ Проще (фиксир. фазы)│ Сложнее (гибкость)     ║
║  Скорость сборки   │ Медленнее          │ Быстрее (кэш, инкр.)   ║
║  Инкрементальность │ Нет                │ Да                      ║
║  Build Cache       │ Нет                │ Да                      ║
║  Параллелизм       │ Ограниченный       │ Хороший                 ║
║  Гибкость          │ Ограниченная       │ Очень гибкий            ║
║  Экосистема        │ Огромная (старая)  │ Растущая                ║
║  Android           │ Не поддерживается  │ Официальный инструмент  ║
║  Wrapper           │ Нет (нужен mvnw)   │ Встроенный              ║
║  Multi-проекты     │ Сложнее            │ Удобнее                 ║
║  IDE поддержка     │ Отличная           │ Отличная                ║
║  Популярность      │ Более популярен    │ Растёт                  ║
║                                                                    ║
╠═══════════════════════════════════════════════════════════════════╣
║                                                                    ║
║  КОГДА ВЫБИРАТЬ MAVEN:                                             ║
║  ✅ Корпоративные проекты с устоявшимися практиками               ║
║  ✅ Команда знакома с Maven                                        ║
║  ✅ Нужна простота и предсказуемость                              ║
║  ✅ Много legacy кода на Maven                                     ║
║                                                                    ║
║  КОГДА ВЫБИРАТЬ GRADLE:                                            ║
║  ✅ Android разработка (единственный вариант)                      ║
║  ✅ Большие проекты (быстрее компилируется)                        ║
║  ✅ Multi-module проекты                                           ║
║  ✅ Нужна гибкость и кастомизация                                 ║
║  ✅ Хотите использовать Kotlin DSL                                 ║
║                                                                    ║
╚═══════════════════════════════════════════════════════════════════╝
```

### Примеры эквивалентной конфигурации

XML

```
<!-- ═══════════════════════════════════════════════════════════ -->
<!-- MAVEN (pom.xml)                                             -->
<!-- ═══════════════════════════════════════════════════════════ -->

<project>
    <modelVersion>4.0.0</modelVersion>
    
    <groupId>com.example</groupId>
    <artifactId>my-app</artifactId>
    <version>1.0.0</version>
    <packaging>jar</packaging>
    
    <properties>
        <java.version>17</java.version>
        <spring.boot.version>3.1.0</spring.boot.version>
    </properties>
    
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <version>${spring.boot.version}</version>
        </dependency>
        
        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <version>42.6.0</version>
            <scope>runtime</scope>
        </dependency>
        
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter</artifactId>
            <version>5.9.3</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
    
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <version>${spring.boot.version}</version>
            </plugin>
        </plugins>
    </build>
</project>
```

groovy

```
// ═══════════════════════════════════════════════════════════
// GRADLE (build.gradle)
// ═══════════════════════════════════════════════════════════

plugins {
    id 'java'
    id 'org.springframework.boot' version '3.1.0'
}

group = 'com.example'
version = '1.0.0'
sourceCompatibility = '17'

repositories {
    mavenCentral()
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    runtimeOnly 'org.postgresql:postgresql:42.6.0'
    testImplementation 'org.junit.jupiter:junit-jupiter:5.9.3'
}

test {
    useJUnitPlatform()
}
```

---

## 5. NPM (Node Package Manager)

### Что такое NPM

text

```
╔═══════════════════════════════════════════════════════════════════╗
║                              NPM                                   ║
╠═══════════════════════════════════════════════════════════════════╣
║                                                                    ║
║  NPM — менеджер пакетов для JavaScript/Node.js экосистемы.        ║
║                                                                    ║
║  Год создания: 2010                                                ║
║  Конфигурация: package.json                                       ║
║  Lock файл: package-lock.json                                     ║
║  Репозиторий: npmjs.com (крупнейший в мире — > 2 млн пакетов)    ║
║  Альтернативы: Yarn, pnpm                                          ║
║                                                                    ║
╠═══════════════════════════════════════════════════════════════════╣
║                                                                    ║
║  ЧТО ДЕЛАЕТ NPM:                                                   ║
║                                                                    ║
║  1️⃣  Управление зависимостями проекта                             ║
║  2️⃣  Запуск скриптов (build, test, start)                         ║
║  3️⃣  Публикация собственных пакетов                               ║
║  4️⃣  Управление версиями пакетов                                  ║
║  5️⃣  Аудит безопасности зависимостей                              ║
║                                                                    ║
╚═══════════════════════════════════════════════════════════════════╝
```

### Структура package.json

JSON

```
{
  "name": "my-application",
  "version": "1.0.0",
  "description": "Example Node.js application",
  "main": "dist/index.js",
  "type": "module",
  
  "scripts": {
    "start": "node dist/index.js",
    "dev": "nodemon src/index.ts",
    "build": "tsc",
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage",
    "lint": "eslint src/**/*.ts",
    "format": "prettier --write src/**/*.ts",
    "prepare": "husky install"
  },
  
  "keywords": ["example", "nodejs"],
  "author": "Your Name <your.email@example.com>",
  "license": "MIT",
  
  "dependencies": {
    "express": "^4.18.2",
    "axios": "~1.4.0",
    "dotenv": "16.0.3"
  },
  
  "devDependencies": {
    "@types/node": "^20.3.1",
    "@types/express": "^4.17.17",
    "typescript": "^5.1.3",
    "nodemon": "^2.0.22",
    "jest": "^29.5.0",
    "eslint": "^8.42.0",
    "prettier": "^2.8.8",
    "husky": "^8.0.3"
  },
  
  "peerDependencies": {
    "react": ">=16.8.0"
  },
  
  "engines": {
    "node": ">=18.0.0",
    "npm": ">=9.0.0"
  }
}
```

### Semantic Versioning в NPM

text

```
┌─────────────────────────────────────────────────────────────────────┐
│                    SEMANTIC VERSIONING (SemVer)                      │
└─────────────────────────────────────────────────────────────────────┘

Формат: MAJOR.MINOR.PATCH (например, 1.4.2)

MAJOR — несовместимые изменения API
MINOR — новая функциональность (обратно совместимая)
PATCH — исправления багов


NPM ДИАПАЗОНЫ ВЕРСИЙ:
──────────────────────

"express": "4.18.2"     → Точная версия
"express": "^4.18.2"    → Совместимые минорные (4.18.2 - 4.x.x, но НЕ 5.0.0)
"express": "~4.18.2"    → Совместимые патчи (4.18.2 - 4.18.x, но НЕ 4.19.0)
"express": "*"          → Любая версия (НЕ рекомендуется!)
"express": ">=4.18.0"   → 4.18.0 и выше
"express": "<5.0.0"     → Меньше 5.0.0
"express": "4.18.x"     → 4.18.0, 4.18.1, ..., но НЕ 4.19.0


РЕКОМЕНДАЦИИ:
─────────────

✅ Используйте ^  для libraries (гибкость)
✅ Используйте точные версии для applications (стабильность)
✅ Всегда коммитьте package-lock.json (фиксирует точные версии)
```

### Основные команды NPM

Bash

```
# ═══════════════════════════════════════════════════════════
#                    ОСНОВНЫЕ КОМАНДЫ NPM
# ═══════════════════════════════════════════════════════════

# ─────────────────────────────────────────────────────────────
# ИНИЦИАЛИЗАЦИЯ И УСТАНОВКА
# ─────────────────────────────────────────────────────────────

# Создать package.json интерактивно
npm init

# Создать с defaults
npm init -y

# Установить все зависимости из package.json
npm install
# или короткая форма
npm i

# Установить в CI (использует package-lock.json, не обновляет его)
npm ci


# ─────────────────────────────────────────────────────────────
# ДОБАВЛЕНИЕ ЗАВИСИМОСТЕЙ
# ─────────────────────────────────────────────────────────────

# Установить пакет (добавляет в dependencies)
npm install express

# Сокращённая форма
npm i express

# Установить конкретную версию
npm i express@4.18.2

# Установить как dev dependency
npm install --save-dev typescript
npm i -D typescript

# Установить глобально
npm install -g typescript

# Установить из Git
npm i user/repo#branch


# ─────────────────────────────────────────────────────────────
# УДАЛЕНИЕ ЗАВИСИМОСТЕЙ
# ─────────────────────────────────────────────────────────────

# Удалить пакет
npm uninstall express
npm un express  # короткая форма


# ─────────────────────────────────────────────────────────────
# ОБНОВЛЕНИЕ
# ─────────────────────────────────────────────────────────────

# Проверить устаревшие пакеты
npm outdated

# Обновить все пакеты (в рамках semver из package.json)
npm update

# Обновить конкретный пакет
npm update express

# Обновить до latest (игнорируя semver)
npm install express@latest


# ─────────────────────────────────────────────────────────────
# ИНФОРМАЦИЯ
# ─────────────────────────────────────────────────────────────

# Список установленных пакетов
npm list

# Только top-level
npm list --depth=0

# Глобальные пакеты
npm list -g --depth=0

# Информация о пакете
npm info express

# Посмотреть package.json пакета
npm view express


# ─────────────────────────────────────────────────────────────
# БЕЗОПАСНОСТЬ
# ─────────────────────────────────────────────────────────────

# Проверить уязвимости
npm audit

# Исправить автоматически
npm audit fix

# Принудительное исправление (может сломать совместимость)
npm audit fix --force


# ─────────────────────────────────────────────────────────────
# СКРИПТЫ
# ─────────────────────────────────────────────────────────────

# Запустить скрипт из package.json
npm run build
npm run test

# Специальные скрипты (можно без "run")
npm start   # npm run start
npm test    # npm run test
npm stop    # npm run stop


# ─────────────────────────────────────────────────────────────
# ПУБЛИКАЦИЯ
# ─────────────────────────────────────────────────────────────

# Логин в npm registry
npm login

# Опубликовать пакет
npm publish

# Опубликовать с тегом
npm publish --tag beta


# ─────────────────────────────────────────────────────────────
# ОЧИСТКА
# ─────────────────────────────────────────────────────────────

# Очистить кэш
npm cache clean --force

# Удалить node_modules и переустановить
rm -rf node_modules package-lock.json
npm install


# ─────────────────────────────────────────────────────────────
# NPX — запуск пакетов без установки
# ─────────────────────────────────────────────────────────────

# Запустить пакет без установки
npx create-react-app my-app

# Запустить конкретную версию
npx typescript@4.9.0 --version

# Запустить локально установленный пакет
npx eslint src/
```

---

## 6. Другие Dependency Management Tools

text

```
╔═══════════════════════════════════════════════════════════════════╗
║          DEPENDENCY MANAGEMENT TOOLS ПО ЯЗЫКАМ                     ║
╠═══════════════════════════════════════════════════════════════════╣
║                                                                    ║
║  PYTHON                                                            ║
║  ───────                                                           ║
║  • pip + requirements.txt — базовый                                ║
║  • Pipenv — виртуальные окружения + Pipfile                       ║
║  • Poetry — современный, pyproject.toml                            ║
║  • conda — для data science                                        ║
║                                                                    ║
║  GO                                                                ║
║  ──                                                                ║
║  • Go Modules (go.mod) — встроенный с Go 1.11+                    ║
║                                                                    ║
║  RUST                                                              ║
║  ────                                                              ║
║  • Cargo (Cargo.toml) — официальный                                ║
║                                                                    ║
║  RUBY                                                              ║
║  ────                                                              ║
║  • Bundler (Gemfile) — стандарт                                    ║
║                                                                    ║
║  PHP                                                               ║
║  ───                                                               ║
║  • Composer (composer.json) — стандарт                             ║
║                                                                    ║
║  .NET                                                              ║
║  ────                                                              ║
║  • NuGet (.csproj) — официальный                                   ║
║                                                                    ║
║  C/C++                                                             ║
║  ─────                                                             ║
║  • CMake + Conan                                                   ║
║  • vcpkg (Microsoft)                                               ║
║                                                                    ║
╚═══════════════════════════════════════════════════════════════════╝
```

---

## 7. Использование в CI/CD

### Maven в CI/CD

YAML

```
# .gitlab-ci.yml для Maven проекта

variables:
  MAVEN_OPTS: "-Dmaven.repo.local=$CI_PROJECT_DIR/.m2/repository"
  MAVEN_CLI_OPTS: "--batch-mode --errors --fail-at-end --show-version"

# Кэширование .m2 репозитория
cache:
  paths:
    - .m2/repository

stages:
  - build
  - test
  - package
  - deploy

build:
  stage: build
  image: maven:3.9-eclipse-temurin-17
  script:
    - mvn $MAVEN_CLI_OPTS compile
  artifacts:
    paths:
      - target/classes

test:
  stage: test
  image: maven:3.9-eclipse-temurin-17
  script:
    - mvn $MAVEN_CLI_OPTS test
  artifacts:
    reports:
      junit:
        - target/surefire-reports/TEST-*.xml
    paths:
      - target/surefire-reports

package:
  stage: package
  image: maven:3.9-eclipse-temurin-17
  script:
    - mvn $MAVEN_CLI_OPTS package -DskipTests
  artifacts:
    paths:
      - target/*.jar
  only:
    - main

deploy:
  stage: deploy
  image: maven:3.9-eclipse-temurin-17
  script:
    - mvn $MAVEN_CLI_OPTS deploy -DskipTests
  only:
    - main
```

### Gradle в CI/CD

YAML

```
# .gitlab-ci.yml для Gradle проекта

variables:
  GRADLE_OPTS: "-Dorg.gradle.daemon=false"
  GRADLE_USER_HOME: "$CI_PROJECT_DIR/.gradle"

# Кэширование
cache:
  key: "$CI_COMMIT_REF_NAME"
  paths:
    - .gradle/wrapper
    - .gradle/caches

stages:
  - build
  - test
  - package

build:
  stage: build
  image: gradle:8.3-jdk17
  script:
    - ./gradlew assemble
  artifacts:
    paths:
      - build/libs/*.jar
    expire_in: 1 week

test:
  stage: test
  image: gradle:8.3-jdk17
  script:
    - ./gradlew test
  artifacts:
    reports:
      junit: build/test-results/test/TEST-*.xml
    paths:
      - build/reports/tests

package:
  stage: package
  image: gradle:8.3-jdk17
  script:
    - ./gradlew bootJar
  artifacts:
    paths:
      - build/libs/*.jar
  only:
    - main
```

### NPM в CI/CD

YAML

```
# .gitlab-ci.yml для Node.js проекта

image: node:20

# Кэширование node_modules
cache:
  key:
    files:
      - package-lock.json
  paths:
    - node_modules/
    - .npm/

stages:
  - install
  - lint
  - test
  - build
  - deploy

install:
  stage: install
  script:
    - npm ci --cache .npm --prefer-offline
  artifacts:
    paths:
      - node_modules/
    expire_in: 1 hour

lint:
  stage: lint
  needs:
    - install
  script:
    - npm run lint

security:
  stage: lint
  needs:
    - install
  script:
    - npm audit --audit-level=high
  allow_failure: true

test:
  stage: test
  needs:
    - install
  script:
    - npm run test -- --coverage
  coverage: '/All files[^|]*\|[^|]*\s+([\d\.]+)/'
  artifacts:
    reports:
      junit: junit.xml
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml

build:
  stage: build
  needs:
    - test
  script:
    - npm run build
  artifacts:
    paths:
      - dist/
  only:
    - main

deploy:
  stage: deploy
  needs:
    - build
  script:
    - npm run deploy
  only:
    - main
```

---

## 8. Шпаргалка

Bash

```
# ═══════════════════════════════════════════════════════════
#               DEPENDENCY MANAGEMENT CHEATSHEET
# ═══════════════════════════════════════════════════════════

# ═════════════════════════════════
#             MAVEN
# ═════════════════════════════════

mvn clean install               # Сборка + установка в локальный repo
mvn clean package               # Сборка без установки
mvn dependency:tree             # Дерево зависимостей
mvn versions:display-dependency-updates  # Проверка обновлений
mvn clean package -P prod       # С профилем


# ═════════════════════════════════
#             GRADLE
# ═════════════════════════════════

./gradlew build                 # Полная сборка
./gradlew clean build           # Очистка + сборка
./gradlew dependencies          # Дерево зависимостей
./gradlew build --scan          # Build scan (детальная информация)
./gradlew build --offline       # Offline режим


# ═════════════════════════════════
#              NPM
# ═════════════════════════════════

npm install                     # Установка всех зависимостей
npm ci                          # Clean install (для CI)
npm i express                   # Добавить зависимость
npm i -D jest                   # Добавить dev зависимость
npm outdated                    # Проверить обновления
npm audit                       # Проверить безопасность
npm run build                   # Запустить скрипт


# ═════════════════════════════════
#             PYTHON
# ═════════════════════════════════

pip install -r requirements.txt # Установка зависимостей
pip freeze > requirements.txt   # Сохранить зависимости
poetry install                  # Poetry
pipenv install                  # Pipenv


# ═════════════════════════════════
#              GO
# ═════════════════════════════════

go mod download                 # Скачать зависимости
go mod tidy                     # Очистить неиспользуемые
go mod vendor                   # Создать vendor/


# ═════════════════════════════════
#        ОБЩИЕ КОНЦЕПЦИИ
# ═════════════════════════════════

# Типы зависимостей:
# - dependencies        → Production зависимости
# - devDependencies     → Только для разработки
# - peerDependencies    → Требования к окружению
# - optionalDependencies → Опциональные

# Lock файлы (фиксируют точные версии):
# - package-lock.json (npm)
# - yarn.lock (Yarn)
# - Gemfile.lock (Bundler)
# - poetry.lock (Poetry)
# - go.sum (Go)

# Всегда коммитить lock файлы в Git!
```