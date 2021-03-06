[Documentation main page](https://frinxio.github.io/Frinx-docs/)
[SBE operations manual](https://frinxio.github.io/Frinx-docs/FRINX_Smart_Build_Engine/operations_manual.html)
# Designing a new application with the FRINX ODL distribution

<!-- TOC START min:1 max:4 link:true update:true -->
- [Designing a new application with the FRINX ODL distribution](#designing-a-new-application-with-the-frinx-odl-distribution)
  - [1\. Project structure](#1-project-structure)
  - [2\. POM (Project Object Model) files](#2-pom-project-object-model-files)
  - [3\. Features module](#3-features-module)
  - [4\. Configuration module](#4-configuration-module)
  - [5\. Integration with the FRINX distribution](#5-integration-with-the-frinx-distribution)
  - [6\. Building the application](#6-building-the-application)
  - [7\. Checkstyle, naming schemes, Javadoc](#7-checkstyle-naming-schemes-javadoc)
  - [8\. Logging](#8-logging)
  - [9\. SonarQube](#9-sonarqube)
  - [10\. Developing and testing with a distribution](#10-developing-and-testing-with-a-distribution)

<!-- TOC END -->

## 1\. Project structure

Each project or plugin using ODL should have the same directory structure. This has several advantages. For a programmer, jumping from one project to another is very easy, because he knows where all the key files are (features.xml, controller-config, etc.). For this purpose, ODL developed a maven archetype (template) to automatically create project structure with POM files, base wiring and configuration.

The project is created by issuing the following command in a terminal:

      mvn archetype:generate -DarchetypeGroupId=org.opendaylight.controller -DarchetypeArtifactId=opendaylight-startup-archetype \
    -DarchetypeRepository=http://nexus.opendaylight.org/content/repositories/<Snapshot-Type>/ \
    -DarchetypeCatalog=http://nexus.opendaylight.org/content/repositories/<Snapshot-Type>/archetype-catalog.xml \
    -DarchetypeVersion=<Archetype-Version>


where: **Snapshot-Type** can be either **opendaylight.release** or **opendaylight.snapshot**  
**Archetype-Version** is the latest version of maven-startup-archetype

**How to get the archetype version:**

1.  open artifactory.frinx
2.  open either frinx-release or frinx-snapshot
3.  move to org/opendaylight/controller/opendaylight-startup-archetype
4.  choose the latest 

The command will generate several prompts; you can enter the values according to the example [here][1]. At the end the project structure is generated.

    [INFO] Using following parameters for creating project from Archetype: opendaylight-startup-archetype:**<archetype-name>**  
    [INFO] ----------------------------------------------------------------------------
    [INFO] Parameter: groupId, Value: jp.customer.example
    [INFO] Parameter: artifactId, Value: example
    [INFO] Parameter: version, Value: 1.0.0-SNAPSHOT
    [INFO] Parameter: package, Value: jp.customer.example
    [INFO] Parameter: packageInPathFormat, Value: jp/customer/example
    [INFO] Parameter: classPrefix, Value: Example
    [INFO] Parameter: package, Value: jp.customer.example
    [INFO] Parameter: version, Value: 1.0.0-SNAPSHOT
    [INFO] Parameter: copyright, Value: customer
    [INFO] Parameter: groupId, Value: jp.customer.example
    [INFO] Parameter: artifactId, Value: example
    [INFO] Parameter: copyrightYear, Value: 2015
    [INFO] project created from Archetype in dir: /home/username/code/example
    [INFO] ------------------------------------------------------------------------
    [INFO] BUILD SUCCESS
    [INFO] ------------------------------------------------------------------------


## 2\. POM (Project Object Model) files

POM files are special maven files where we can configure plugins and declare dependencies. The latter is the most common use for an ODL application programmer. POM files are hierarchical, meaning a POM file can declare another POM file as its parent and inherit all its plugins and dependencies. Almost all plugins used in ODL are configured in odlparent POM [more info here][2] so you do not need to specify it in your POM file. However you do have to specify that your POM file parent is the odlparent POM.

      <parent>
    <groupId>org.opendaylight.odlparent</groupId>
    <artifactId>odlparent-lite</artifactId>
    <version>insert version name here</version>
    <relativePath/>
      </parent>


In here, you can for example turn off the enforcement of checkstyle (you can set it to NOT fail the build if the checkstyle fails -- but this is discouraged). Dependency management plays an important role in a clean project. In general, transitive dependencies shouldn't be used at all. As a project grows, from time to time it pays off to clear pom files from unused or transitive dependencies.

**Very helpful command:**

    mvn dependency:analyze


The above command analyzes the dependencies and prints out a report stating which dependencies are unused or used but not declared. Here is an example response:

    [INFO] --- maven-dependency-plugin:2.10:analyze (default-cli) @ example-impl ---
    [WARNING] Used undeclared dependencies found:
    [WARNING]org.apache.commons:commons-lang3:jar:3.4:compile
    [WARNING]org.mockito:mockito-core:jar:1.9.5:test
    [WARNING]org.opendaylight.controller:sal-binding-api:jar:1.1.4-Beryllium-SR4.5-frinxodl-SNAPSHOT:compile
    [WARNING]commons-io:commons-io:jar:2.4:compile
    [WARNING]com.google.code.findbugs:jsr305:jar:3.0.0:compile
    [WARNING] Unused declared dependencies found:
    [WARNING]commons-fileupload:commons-fileupload:jar:1.3.1:compile
    [INFO] ------------------------------------------------------------------------


After your POM files are adjusted accordingly, you should see 'Build Success'. Note: this command only recognizes the usage of the dependencies in JAVA code. Therefore it will list all dependencies in features POM file as unused.

## 3\. Features module

Features module POM files are not inherited from odlparent or odlparent-lite, but from features-parent ([see here][3]).  
This means that all feature dependencies need to be specified in this POM as well as the parent POM of the project. We will use the name of the features module and the version in integration with the Frinx distribution. All features within ODL should be prefixed by '**odl-**', but you can change this manually in the feature file.

## 4\. Configuration module

In Beryllium, config sub-system that was used to configure the project is deprecated. All new projects and plugins are encouraged to use Blueprint.  
However the usage of config sub-system will be supported from Boron onwards until all existing projects have migrated to Blueprint.

## 5\. Integration with the FRINX distribution

In order for the distribution to recognize your new application, you need to edit three files in the distribution project:

        modified:   pom.xml
        modified:   features-core/pom.xml
        modified:   features-core/src/main/resources/features.xml


In the first of the above files we need to specify a variable for the version that will be used in the distribution:

       <feature.example.version>1.0.0-SNAPSHOT</feature.example.version>


In the second POM file we need to add a dependency to our application artifacts:

    <dependency>
      <groupId>jp.customer.example</groupId>
      <artifactId>example-features</artifactId>
      <version>${feature.example.version}</version>
      <classifier>features</classifier>
      <type>xml</type>
    </dependency>


In features.xml we specify the feature repository:

    <repository>mvn:jp.customer.example/example-features/${feature.example.version}/xml/features</repository>


## 6\. Building the application

First, our application needs to be built in order to publish artifacts into **~/.m2/repository** . For this We use the command:

    mvn clean install -DskipTests


Skipping tests shouldn't be used regularly; we do it only for the purposes of this demo, as SingleFeatureTest takes about 20 minutes.

The first build of the application usually takes longer, because all the artifacts and dependencies need to be downloaded. If you want to force the downloading of the artifacts even after the first build, you can use -U argument

    mvn -U clean install


The second step is to build the distribution project. You can build it safely without tests as you didn't change any of its code. Note: it will require a few GB of memory.

After the distribution is built, move to `distribution/distribution-karaf/target/assembly` . This is the root directory from which you can run karaf. Run it with the command:

    ./bin/karaf


and install the example feature:

    feature install:odl-example


## 7\. Checkstyle, naming schemes, Javadoc

Checkstyle is a plugin that encourages a unified style of coding throughout all ODL applications and projects. It controls whitespaces, ordering of imports etc. You can learn more [here][4]

Javadoc plays an important part in public APIs. Public methods should be documented extensively. They should provide information on HOW (and sometimes WHEN) to use a given method, not what the implementation looks like. Javadoc can be generated through maven:

    mvn javadoc:javadoc


## 8\. Logging

The best tutorial for logging can be found here:

<https://wiki.opendaylight.org/view/BestPractices/Logging_Best_Practices>

In ODL, the configuration file for logging is located here: `etc/org.ops4j.pax.logging.cfg` (within the karaf directory)

Here you can configure the size of log file, default location, enabling/disabling of rotation, overall log appearance and (the most used setting) - the specific log level for a specific namespace. The last one can be set by either issuing a command in the karaf prompt:

    log:set TRACE jp.customer.example


or adding the following line into the file org.ops4j.pax.logging.cfg: `log4j.logger.jp.customer.example = TRACE`

This will set trace logging for all the classes within the given namespace. You can specify a particular class when you don't want unnecessary log pollution. Log levels are hierarchical, from the finest level to error.

**ERROR**

**WARN**

**INFO**

**DEBUG**

**TRACE**

This means that if you enable a level, all levels above this one will also be displayed. For example, if you set DEBUG level, then INFO, WARN and ERROR will also be displayed, if you set WARN level, only WARN and ERROR will be displayed.

For bug reports you should always use at least DEBUG level.

## 9\. SonarQube

SonarQube (or any other code analysis tool) is much needed to write and maintain clean and tested code. Sonar is used widely in ODL. There are numerous ways to check your code with Sonar.

1.  run it on a server and create a special Jenkins job to do the sonar analysis (it usually takes longer than an ordinary verify job), but it can be executed nightly/weekly, depending on your team's needs.
2.  run the server locally and do the Sonar analysis by issuing the maven command

    mvn sonar:sonar

3.  install Sonar plugins for your IDE (e.g. SonarLint in Eclipse)

SonarQube has a set of rules (called Quality Profiles) that executes against the codebase. The rules fall into different categories based on the nature of the rule, complexity or impact on the code.

It also creates a clear annotation of code coverage, marking covered and uncovered lines, marking uncovered conditions. Other features include counting duplcated lines or checking javadoc comments.

It's an ideal tool for a project manager as it contains a lot of statistics and progress over time.

## 10\. Developing and testing with a distribution

When you want to test your application with the Frinx distribution, you sometimes have to make a lot of changes/bug fixes on the fly. As such, creating a distribution each time can be very time consuming. There are a few things that can make it easier:

*   instead of building the distribution, build only your part of the application that changed (e.g. if you made changes in impl module, build only that one). Copy the resulting jar from `target/ directory` of the module to  
    `distribution/distribution-karaf/target/assembly/system/org/opendaylight/<project_name>/<project_module>/<project_version>`  
    Restart ODL. (NOTE: We highly recommend deleting the data/ folder between restarts, as it contains cache data that may influence the result of the testing.)

*   make your feature a boot feature. Instead of installing your feature everytime ODL is started, write it as a boot feature in the following file `etc/org.apache.karaf.features.cfg`

    In this file, append your feature to the end of the comma separated list of features to install at startup - add the line:

    `featuresBoot=config,standard,region,package,kar,ssh,management,<your_feature>`

*   adjust logs to TRACE or DEBUG level for your namespace in the files instead of commandline

    Add line to `etc/org.ops4j.pax.logging.cfg` :

    `log4j.logger.jp.customer.example = TRACE`

 [1]: https://wiki.opendaylight.org/view/OpenDaylight_Controller:MD-SAL:Startup_Project_Archetype "here"
 [2]: https://git.opendaylight.org/gerrit/gitweb?p=odlparent.git;a=blob;f=odlparent/pom.xml;hb=refs/heads/master "more info here"
 [3]: https://git.opendaylight.org/gerrit/gitweb?p=odlparent.git;a=blob;f=features-parent/pom.xml;hb=refs/heads/master "see here"
 [4]: https://wiki.opendaylight.org/view/BestPractices/Coding_Guidelines "here"
