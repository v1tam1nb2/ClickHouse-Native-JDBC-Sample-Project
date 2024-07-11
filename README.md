# ClickHouse-Native-JDBC-Sample-Project
ClickHouse-Native-JDBCの検証

# 参考

- [Maven でプロジェクトの作成からビルド、実行までをやってみた](https://a4dosanddos.hatenablog.com/entry/2015/02/22/163009)

# 検証

## プロジェクトの作成

```bash
mvn archetype:generate -DgroupId=com.mycompany.app -DartifactId=my-app -DarchetypeArtifactId=maven-archetype-quickstart -DarchetypeVersion=1.4 -DinteractiveMode=false
```

## POMの作成

最終的な`pom.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.mycompany.app</groupId>
  <artifactId>my-app</artifactId>
  <version>1.0-SNAPSHOT</version>

  <name>my-app</name>
  <!-- FIXME change it to the project's website -->
  <url>http://www.example.com</url>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>1.7</maven.compiler.source>
    <maven.compiler.target>1.7</maven.compiler.target>
  </properties>

  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.11</version>
      <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>com.github.housepower</groupId>
        <artifactId>clickhouse-native-jdbc-shaded</artifactId>
        <version>2.7.1</version>
    </dependency>
  </dependencies>

  <build>
    <pluginManagement><!-- lock down plugins versions to avoid using Maven defaults (may be moved to parent pom) -->
      <plugins>
        <!-- clean lifecycle, see https://maven.apache.org/ref/current/maven-core/lifecycles.html#clean_Lifecycle -->
        <plugin>
          <artifactId>maven-clean-plugin</artifactId>
          <version>3.1.0</version>
        </plugin>
        <!-- default lifecycle, jar packaging: see https://maven.apache.org/ref/current/maven-core/default-bindings.html#Plugin_bindings_for_jar_packaging -->
        <plugin>
          <artifactId>maven-resources-plugin</artifactId>
          <version>3.0.2</version>
        </plugin>
        <plugin>
          <artifactId>maven-compiler-plugin</artifactId>
          <version>3.8.0</version>
        </plugin>
        <plugin>
          <artifactId>maven-surefire-plugin</artifactId>
          <version>2.22.1</version>
        </plugin>
        <plugin>
          <artifactId>maven-jar-plugin</artifactId>
          <version>3.0.2</version>
            <configuration>
               <archive>
                 <manifest>
                     <mainClass>com.mycompany.app.SimpleQuery</mainClass>
                  </manifest>
              </archive>
            </configuration>
        </plugin>
        <plugin>
          <artifactId>maven-install-plugin</artifactId>
          <version>2.5.2</version>
        </plugin>
        <plugin>
          <artifactId>maven-deploy-plugin</artifactId>
          <version>2.8.2</version>
        </plugin>
        <!-- site lifecycle, see https://maven.apache.org/ref/current/maven-core/lifecycles.html#site_Lifecycle -->
        <plugin>
          <artifactId>maven-site-plugin</artifactId>
          <version>3.7.1</version>
        </plugin>
        <plugin>
          <artifactId>maven-project-info-reports-plugin</artifactId>
          <version>3.0.0</version>
        </plugin>
        <plugin>
          <groupId>org.codehaus.mojo</groupId>
          <artifactId>exec-maven-plugin</artifactId>
          <configuration>
            <mainClass>com.mycompany.app.SimpleQuery</mainClass>
          </configuration>
        </plugin>
      </plugins>
    </pluginManagement>
  </build>
</project>
```

## コードの作成

以下を参考。

- [Maven in 5 Minutes](https://maven.apache.org/guides/getting-started/maven-in-five-minutes.html)
- [https://github.com/housepower/ClickHouse-Native-JDBC/blob/master/examples/src/main/java/examples/SimpleQuery.java](https://github.com/housepower/ClickHouse-Native-JDBC/blob/master/examples/src/main/java/examples/SimpleQuery.java)

`/ClickHouse-Native-JDBC-Sample-Project/my-app/src/main/java/com/mycompany/app/SimpleQuery.java` に作成

```java
/*
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *     http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

/**
 * ここは環境に合わせて変更する
 */
package com.mycompany.app;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.Statement;

/**
 * SimpleQuery
 */
public class SimpleQuery {

    public static void main(String[] args) throws Exception {
        Class.forName("com.mysql.cj.jdbc.Driver");
        try (Connection connection = DriverManager.getConnection("jdbc:clickhouse://<your-ip>:9000?client_name=ck-example")) {
            try (Statement stmt = connection.createStatement()) {
                try (ResultSet rs = stmt.executeQuery(
                        "SELECT (number % 3 + 1) as n, sum(number) FROM numbers(10000000) GROUP BY n")) {
                    while (rs.next()) {
                        System.out.println(rs.getInt(1) + "\t" + rs.getLong(2));
                    }
                }
            }
        }
    }
}
```

## ビルドする

```bash
cd my-app
mvn package
```

## 実行する

```bash
mvn exec:java
```

以下のようになればOK。

```bash
[INFO] Scanning for projects...
[INFO]
[INFO] ----------------------< com.mycompany.app:my-app >----------------------
[INFO] Building my-app 1.0-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO]
[INFO] --- exec-maven-plugin:3.3.0:java (default-cli) @ my-app ---
1       16666668333333
2       16666661666667
3       16666665000000
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  0.570 s
[INFO] Finished at: 2024-07-11T08:45:24+09:00
[INFO] ------------------------------------------------------------------------
```

## TODO

以下を実行するとエラーになるが未解決。

```bash
java -cp target/my-app-1.0-SNAPSHOT.jar com.mycompany.app.SimpleQuery
```

```shell
Exception in thread "main" java.sql.SQLException: No suitable driver found for jdbc:clickhouse://172.26.236.113:9000?client_name=ck-example
        at java.sql/java.sql.DriverManager.getConnection(DriverManager.java:702)
        at java.sql/java.sql.DriverManager.getConnection(DriverManager.java:251)
        at com.mycompany.app.SimpleQuery.main(SimpleQuery.java:29)
```
