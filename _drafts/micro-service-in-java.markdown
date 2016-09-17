---
title:  "Writing a micro service in Java with Dropwizard"
date:   2014-01-26 17:23:00
categories: micro-service java dropwizard
---

Let's start out by creating a pom.xml

{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
 
    <groupId>com.ballofcode.service</groupId>
    <artifactId>account</artifactId>
    <version>0.0.1-SNAPSHOT</version>
 
    <dependencies>
        <dependency>
            <groupId>com.yammer.dropwizard</groupId>
            <artifactId>dropwizard-core</artifactId>
            <version>0.6.2</version>
        </dependency>
    </dependencies>
    
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    </properties>
    
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.1</version>
                <configuration>
                    <source>1.7</source>
                    <target>1.7</target>
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>1.6</version>
                <configuration>
                    <createDependencyReducedPom>true</createDependencyReducedPom>
                    <filters>
                        <filter>
                            <artifact>*:*</artifact>
                            <excludes>
                                <exclude>META-INF/*.SF</exclude>
                                <exclude>META-INF/*.DSA</exclude>
                                <exclude>META-INF/*.RSA</exclude>
                            </excludes>
                        </filter>
                    </filters>
                </configuration>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>shade</goal>
                        </goals>
                        <configuration>
                            <transformers>
                                <transformer implementation="org.apache.maven.plugins.shade.resource.ServicesResourceTransformer"/>
                                <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                                    <mainClass>com.ballofcode.service.account.AccountService</mainClass>
                                </transformer>
                            </transformers>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-jar-plugin</artifactId>
                <version>2.3.2</version>
                <configuration>
                    <archive>
                        <manifest>
                            <addDefaultImplementationEntries>true</addDefaultImplementationEntries>
                        </manifest>
                    </archive>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
{% endhighlight %}

I then imported the pom into IntelliJ IDEA, because I seem to like it better for Java development than vim.

Next we're going to create a configuration class for our service. DropWizard requires us to always have a configuration class, but since we don't really have anything to configure yet, we'll leave it empty.

{% highlight java %}
package com.ballofcode.service.account;

import com.yammer.dropwizard.config.Configuration;

public class AccountServiceConfiguration extends Configuration {
}
{% endhighlight %}

Moving on we'll create our actual service.

{% highlight java %}
package com.ballofcode.service.account;

import com.yammer.dropwizard.Service;
import com.yammer.dropwizard.config.Bootstrap;
import com.yammer.dropwizard.config.Environment;

public class AccountService extends Service<AccountServiceConfiguration> {

    public static void main(String[] args) throws Exception {
        new AccountService().run(args);
    }

    @Override
    public void initialize(Bootstrap<AccountServiceConfiguration> bootstrap) {
        bootstrap.setName("account-service");
    }

    @Override
    public void run(AccountServiceConfiguration configuration, Environment environment) throws Exception {
        // nothing yet
    }
}
{% endhighlight %}
