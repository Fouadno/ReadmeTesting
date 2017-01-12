## Bio4j default Blueprints model implementation

Bio4j has an intermediate Blueprints layer, which allows us to make a default implementation of the [abstract domain model](https://github.com/bio4j/bio4j) using [Tinkerpop Blueprints API](https://github.com/tinkerpop/blueprints/wiki) and at the same time stay independent from the choice of database technology.

See also technology specific implementations:

- [**Titan DB**](https://github.com/bio4j/titandb)
- [**Neo4j DB**](https://github.com/bio4j/neo4jdb)

### SBT dependency

To use it in your sbt-project, add this to `build.sbt`:

```scala
resolvers += "Era7 maven releases" at "http://releases.era7.com.s3.amazonaws.com"

libraryDependencies += "bio4j" % "blueprints" % "0.4.0"
```


[![Build Status](https://travis-ci.org/atteo/classindex.svg)](https://travis-ci.org/atteo/classindex)
[![Coverage Status](https://img.shields.io/coveralls/atteo/classindex.svg)](https://coveralls.io/r/atteo/classindex)
[![Maven Central](https://maven-badges.herokuapp.com/maven-central/org.atteo.classindex/classindex/badge.svg)](https://maven-badges.herokuapp.com/maven-central/org.atteo.classindex/classindex)
[![Apache 2](http://img.shields.io/badge/license-Apache%202-red.svg)](http://www.apache.org/licenses/LICENSE-2.0)

About
=====
ClassIndex is a much quicker alternative to every run-time annotation scanning library like Reflections or Scannotations.

ClassIndex consist of two parts:

1. An annotation processor which at compile-time generates an index of classes implementing given interface, classes annotated by given annotation or placed in a common package. Thanks to [automatic discovery](http://www.jcp.org/en/jsr/detail?id=269) the processor will be automatically executed when classindex.jar is added to the classpath.
2. Run-time API to read the content of generated indexes.

Why ClassIndex?
===============

Class path scanning is very slow process. Replacing it with compile-time indexing speeds Java applications bootstrap considerably.

Here are the results of the [benchmark](https://github.com/atteo/classindex-benchmark) comparing ClassIndex with the various scanning solutions.

| Library                  | Application startup time |
| :----------------------- |-------------------------:|
| None - hardcoded list    |                  0:00.18 |
| [Scannotation](http://scannotation.sourceforge.net/)             |                  0:05.11 |
| [Reflections](https://github.com/ronmamo/reflections)            |                  0:05.37 |
| Reflections Maven plugin |                                                          0:00.52 |
| [Corn](https://sites.google.com/site/javacornproject/corn-cps)   |                  0:24.60 |
| ClassIndex               |                  0:00.18 |

Notes: benchmark was performed on Intel i5-2520M CPU @ 2.50GHz, classpath size was set to 121MB.

Changes
=======
Version 3.4
- TypeElement and PackageElement cannot be used reliably as a key for the map
- bring back compatibility with Android API < 19 by not depending on the availability of StandardCharsets class

Version 3.3
- New methods to return names of classes as a string

Version 3.2
- Better Java8 compatibility
- Better filtering

Version 3.1
- Class filtering - mechanism to filter classes based on various criteria

Version 3.0

- Non-local named nested classes are also indexed (both static and inner classes)
- Fix: incremental compilation in IntelliJ IDEA
- You can now restrict the results to specific class loader
- package name nad groupId have changed to org.atteo.classindex

Version 2.2

- Fix: jaxb.index was in incorrect format

Version 2.1

- Fix: custom processor with indexAnnotation() call resulted in javac throwing Error

Version 2.0

- You can now use [ClassIndex.getClassSummary()](http://www.atteo.org/static/classindex/apidocs/org/atteo/classindex/ClassIndex.html#getClassSummary(java.lang.Class%29)) to retrieve first sentence of the Javadoc. For this to work specify storeJavadoc=true attribute when using IndexAnnotated or IndexSubclasses
- Requires Java 1.7

Version 1.4

- Fix FileNotFoundException when executed under Tomcat from Eclipse

Version 1.3

- Ignore classes which don't exist at runtime (#4).
    This fixes some issues in Eclipse.
- Allow to create custom processors which index subclasses and packages

Version 1.2

- Fix Eclipse support (#3)

Version 1.1

- Fix incremental compilation (#1)


Usage
=====

Class Indexing
--------------

There are two annotations which trigger compile-time indexing:

* [@IndexSubclasses](http://www.atteo.org/static/classindex/apidocs/org/atteo/classindex/IndexSubclasses.html)
	* when placed on interface makes an index of all classes implementing the interface,
	* when placed on a class makes an index of its subclasses
	* and finally when placed in package-info.java it creates an index of all classes inside that package (directly - without subpackages).
* [@IndexAnnotated](http://www.atteo.org/static/classindex/apidocs/org/atteo/classindex/IndexAnnotated.html) when placed on an annotation makes an index of all classes marked with that annotation.

To access the index at run-time use static methods of [ClassIndex](http://www.atteo.org/static/classindex/apidocs/org/atteo/classindex/ClassIndex.html) class.

```java
@IndexAnnotated
public @interface Entity {
}
 
@Entity
public class Car {
}
 
...
 
for (Class<?> klass : ClassIndex.getAnnotated(Entity.class)) {
    System.out.println(klass.getName());
}
```

For subclasses of the given class the index file name and format is compatible with what [ServiceLoader](http://docs.oracle.com/javase/7/docs/api/java/util/ServiceLoader.html) expects. Keep in mind that ServiceLoader also requires for the classes to have zero-argument default constructor.

For classes inside given package the index file is named "jaxb.index", it is located inside the package folder and it's format is compatible with what [JAXBContext.newInstance(String)](http://docs.oracle.com/javase/7/docs/api/javax/xml/bind/JAXBContext.html#newInstance(java.lang.String)) expects.

Javadoc storage
---------------

From version 2.0 [@IndexAnnotated](http://www.atteo.org/static/classindex/apidocs/org/atteo/classindex/IndexAnnotated.html) and [@IndexSubclasses](http://www.atteo.org/static/classindex/apidocs/org/atteo/classindex/IndexSubclasses.html) allow to specify storeJavadoc attribute. When set to true Javadoc comment for the indexed classes will be stored. You can retrieve first sentence of the Javadoc using [ClassIndex.getClassSummary()](http://www.atteo.org/static/classindex/apidocs/org/atteo/classindex/ClassIndex.html#getClassSummary(java.lang.Class)).

```java
@IndexAnnotated(storeJavadoc = true)
public @interface Entity {
}
 
/**
 * This is car.
 * Detailed car description follows.
 */
@Entity
public class Car {
}
 
...
 
assertEquals("This is car", ClassIndex.getClassSummary(Car.class));
```

Class filtering
---------------

Filtering allows you to select only classes with desired characteristics. Here are some basic samples:

* Selecting only top-level classes

```java
ClassFilter.only()
	.topLevel()
	.from(ClassIndex.getAnnotated(SomeAnnotation.class));
```

* Selecting only classes which are top level and public at the same time

```java
ClassFilter.only()
	.topLevel()
	.withModifiers(Modifier.PUBLIC)
	.from(ClassIndex.getAnnotated(SomeAnnotation.class));

```

* Selecting classes which are top-level or enclosed in given class:

```java
ClassFilter.any(
	ClassFilter.only().topLevel(),
	ClassFilter.only().enclosedIn(WithInnerClassesInside.class)
).from(ClassIndex.getAnnotated(SomeAnnotation.class);
```

Indexing when annotations cannot be used
----------------------------------------

Sometimes you cannot easily use annotations to trigger compile time indexing because you don't control the source code of the classes which should be annotated. For instance you cannot add @IndexAnnotated meta-annotation to [@Entity](http://docs.oracle.com/javaee/7/api/javax/persistence/Entity.html) annotation. Although not so straightforward, it is still possible to use ClassIndex in this case.

There are two steps necessary:

1.	First create a custom processor

	```java
	public class MyImportantClassIndexProcessor extends ClassIndexProcessor {
		public MyImportantClassIndexProcessor() {
			indexAnnotations(Entity.class);
		}
	}
	```
	In the constructor specify what indexes should be created by calling apriopriate methods:
	* indexAnnotations(...) - to create index of classes annotated with given annotations
	* indexSubclasses(...) - to create index of subclasses of given parent classes
	* indexPackages(...) - to create index of classes inside given packages.

2.	Register the processor by creating the file 'META-INF/services/javax.annotation.processing.Processor' in your classpath with the full class name of your processor, see the example [here](https://github.com/atteo/classindex/blob/master/classindex/src/test/resources/META-INF/services/javax.annotation.processing.Processor)

Important note: you also need to ensure that your custom processor is always available on the classpath when compiling indexed classes. When that is not the case there will not be any error - those classes will be missing in the index.

Making shaded jar
-----------------
During compilation ClassIndex writes index files. When creating a shaded jar those index files get overwritten by default. To not lost any indexed classes ClassIndex provides special transformer for Maven which merges the index files instead of overwriting them. To use it add the configuration below to your POM file:

```xml
<build>
    <plugins>
        <plugin>
            <artifactId>maven-shade-plugin</artifactId>
            <version>@maven-shade-plugin.version@</version>
            <executions>
                <execution>
                    <phase>package</phase>
                    <goals>
                        <goal>shade</goal>
                    </goals>
                    <configuration>
                        <transformers>
                            <transformer implementation="org.atteo.classindex.ClassIndexTransformer"/>
                        </transformers>
                    </configuration>
                </execution>
            </executions>
            <dependencies>
                <dependency>
                    <groupId>org.atteo.classindex</groupId>
                    <artifactId>classindex-transformer</artifactId>
                    <version>@class index version@</version>
                </dependency>
            </dependencies>
        </plugin>
    </plugins>
</build>
```

Eclipse
=======
Eclipse uses its own Java compiler which is not strictly standard compliant and requires extra configuration.
In Java Compiler -> Annotation Processing -> Factory Path you need to add ClassIndex jar file.
See the [screenshot](https://github.com/atteo/classindex/issues/5#issuecomment-15365420).

License
=======

ClassIndex is available under [Apache License 2.0](https://www.apache.org/licenses/LICENSE-2.0).

Download
========

You can download the library from [here](http://search.maven.org/remotecontent?filepath=org/atteo/classindex/classindex/3.3/classindex-3.3.jar) or use the following Maven dependency:

```xml
<dependency>
    <groupId>org.atteo.classindex</groupId>
    <artifactId>classindex</artifactId>
    <version>3.3</version>
</dependency>
```





API for PlayerData Bukkit Plugin
==
* Plugin is located at http://dev.bukkit.org/server-mods/playerdata/

This is the API for PlayerData. Documentation coming soon.

To start using the API you will want to get a 'PlayerHandler' object. This can be gotten from 'PlayerDataStatic'.

ALWAYS check the API version in your plugin before using any of the api. The api version can be gotten from PlayerDataStatic.

If the API version does not match the version your plugin is compiled for, you should NOT use the api.


Hello, World!
=============

Hello world in every programming language.

Inspired by[Helloworldcollection.de](http://helloworldcollection.de/)

As I watch the collection expand, this project has blown up more than I ever thought possible.
Thanks to everyone who continues to contribute, new languages are created every day!


Spin-Off project smartly suggested and implemented by @zenware
Meet [FizzBuzz](https://github.com/zenware/FizzBuzz) the evolution of [hello-world](https://github.com/leachim6/hello-world)


Openfire
========

[![Build Status](https://travis-ci.org/igniterealtime/Openfire.svg?branch=master)](https://travis-ci.org/igniterealtime/Openfire)  [![Project Stats](https://www.openhub.net/p/Openfire/widgets/project_thin_badge.gif)](https://www.openhub.net/p/Openfire)

About
-----

[Openfire] is a XMPP server licensed under the Open Source Apache License.


[Openfire] - an [Ignite Realtime] community project.

Bug Reporting
-------------

Only a few users have access for for filling bugs in the tracker. New
users should:

1. Create a forums account (only e-mail is a requirement, you can skip all the other fields).
2. Login to a forum account
3. Press New in your toolbar and choose Discussion
4. Choose the [Openfire Dev forum](http://community.igniterealtime.org/community/developers/openfire_dev) of Openfire and add the tag 'bug_report' to your new post

Please search for your issues in the bug tracker before reporting.

Resources
---------

- Bug Tracker: http://issues.igniterealtime.org/browse/OF
- Nightly Builds: http://www.igniterealtime.org/downloads/nightly_openfire.jsp

Ignite Realtime
===============

[Ignite Realtime] is an Open Source community composed of end-users and developers around the world who 
are interested in applying innovative, open-standards-based Real Time Collaboration to their businesses and organizations. 
We're aimed at disrupting proprietary, non-open standards-based systems and invite you to participate in what's already one 
of the biggest and most active Open Source communities.

[Openfire]: http://www.igniterealtime.org/projects/openfire/index.jsp
[Ignite Realtime]: http://www.igniterealtime.org
[XMPP (Jabber)]: http://xmpp.org/


<img src="http://static.jboss.org/hibernate/images/hibernate_logo_whitebkg_200px.png" />


Hibernate ORM is a component/library providing Object/Relational Mapping (ORM) support
to applications and other components/libraries.  It is also provides an implementation of the
JPA specification, which is the standardized Java specification for ORM.  See 
[Hibernate.org](http://hibernate.org/orm/) for additional information. 

[![Build Status](http://ci.hibernate.org/job/hibernate-orm-master-h2-main/badge/icon)](http://ci.hibernate.org/job/hibernate-orm-master-h2-main/)


Quickstart
==========

     git clone git://github.com/hibernate/hibernate-orm.git
     cd hibernate-orm
     ./gradlew clean build

The build requires a Java 8 JDK as JAVA_HOME, but will ensure Java 6 compatibility.
 

Resources
=========
     
Hibernate uses [Gradle](http://gradle.org) as its build tool.  See the _Gradle Primer_ section below if you are new to
Gradle.

Contributors should read the [Contributing Guide](CONTRIBUTING.md)

See the guides for setting up [IntelliJ](https://developer.jboss.org/wiki/ContributingToHibernateUsingIntelliJ) or
[Eclipse](https://developer.jboss.org/wiki/ContributingToHibernateUsingEclipse) as your development environment.  [Building Hibernate ORM](https://community.jboss.org/wiki/BuildingHibernateORM4x) 
is somewhat outdated, but still has


CI Builds
=========

Hibernate makes use of [Jenkins](http://jenkins-ci.org) for its CI needs.  The project is built continuous on each 
push to the upstream repository.   Overall there are a few different jobs, all of which can be seen at 
[http://ci.hibernate.org/view/ORM/](http://ci.hibernate.org/view/ORM/)



Gradle primer
=============

This section describes some of the basics developers and contributors new to Gradle might 
need to know to get productive quickly.  The Gradle documentation is very well done; 2 in 
particular that are indispensable:

* [Gradle User Guide](https://docs.gradle.org/current/userguide/userguide_single.html) is a typical user guide in that
it follows a topical approach to describing all of the capabilities of Gradle.
* [Gradle DSL Guide](https://docs.gradle.org/current/dsl/index.html) is quite unique and excellent in quickly
getting up to speed on certain aspects of Gradle.


Using the Gradle Wrapper
------------------------

For contributors who do not otherwise use Gradle and do not want to install it, Gradle offers a very cool
features called the wrapper.  It lets you run Gradle builds without a previously installed Gradle distro in 
a zero-conf manner.  Hibernate configures the Gradle wrapper for you.  If you would rather use the wrapper and 
not install Gradle (or to make sure you use the version of Gradle intended for older builds) you would just use
the command `gradlew` (or `gradlew.bat`) rather than `gradle` (or `gradle.bat`) in the following discussions.  
Note that `gradlew` is only available in the project's root dir, so depending on your `pwd` you may need to adjust 
the path to `gradlew` as well.

Executing Tasks
---------------

Gradle uses the concept of build tasks (equivalent to Ant targets or Maven phases/goals). You can get a list of
available tasks via 

    gradle tasks

To execute a task across all modules, simply perform that task from the root directory.  Gradle will visit each
sub-project and execute that task if the sub-project defines it.  To execute a task in a specific module you can 
either:

1. `cd` into that module directory and execute the task
2. name the "task path".  For example, in order to run the tests for the _hibernate-core_ module from the root directory you could say `gradle hibernate-core:test`

Common Java related tasks
-------------------------

* _build_ - Assembles (jars) and tests this project
* _buildDependents_ - Assembles and tests this project and all projects that depend on it.  So think of running this in hibernate-core, Gradle would assemble and test hibernate-core as well as hibernate-envers (because envers depends on core)
* _classes_ - Compiles the main classes
* _testClasses_ - Compiles the test classes
* _compile_ (Hibernate addition) - Performs all compilation tasks including staging resources from both main and test
* _jar_ - Generates a jar archive with all the compiled classes
* _test_ - Runs the tests
* _publish_ - Think Maven deploy
* _publishToMavenLocal_ - Installs the project jar to your local maven cache (aka ~/.m2/repository).  Note that Gradle 
never uses this, but it can be useful for testing your build with other local Maven-based builds.
* _eclipse_ - Generates an Eclipse project
* _idea_ - Generates an IntelliJ/IDEA project (although the preferred approach is to use IntelliJ's Gradle import).
* _clean_ - Cleans the build directory



# AppleSNQuery

---

AppleSNQuery is an application used for querying device information of Apple,which includes device activated information,warranty information,etc.
Only by typing your device serial number and click query button,you can get the information of your Apple device.

- [https://github.com/JeffinBao/AppleSNQuery](https://github.com/JeffinBao/AppleSNQuery])

## Function Introduction (Chinese Version)

---

![](http://ww4.sinaimg.cn/bmiddle/84460b13jw1eqzb1fg589j20ri8r61hp.jpg)

## Where to Download 

---

- [Google Play](https://play.google.com/store/apps/details?id=com.applesnquery.app)

- [WanDouJia(豌豆荚)](http://www.wandoujia.com/apps/com.applesnquery.app)

- [XiaoMi(小米应用商店)](http://app.mi.com/detail/91541)

- [Flyme(魅族应用商店)](http://app.meizu.com/apps/public/detail?package_name=com.applesnquery.app)

## Screen Shots

---

Main Page | Result Page
----------|------------
![](http://ww1.sinaimg.cn/bmiddle/84460b13jw1eqzan9i5nwj21ay27dq9c.jpg) | ![](http://ww1.sinaimg.cn/bmiddle/84460b13jw1eqzanb72krj21ay27d7ce.jpg)

## Contact Me

---

- [Weibo:zeroreh (微博)](http://weibo.com/u/2219182867)
- [Douban:zeroreh (豆瓣)](http://www.douban.com/people/zeroreh/)






[![Maven Central](https://maven-badges.herokuapp.com/maven-central/org.pitest/pitest/badge.svg?style=flat)](https://maven-badges.herokuapp.com/maven-central/org.pitest/pitest)
[![Build Status](https://travis-ci.org/hcoles/pitest.png?branch=master)](https://travis-ci.org/hcoles/pitest)

======

Pitest (aka PIT) is a state of the art mutation testing system for Java and the JVM.

Read all about it at http://pitest.org

## Releases

### 1.1.10-SNAPSHOT

Nothing yet

### 1.1.9

* #132 - Allow analysis of only files touched in last commit (thanks Tomasz Luch)

### 1.1.8

* #239 - Provide a shortcut to set history files via maven
* #240 - Support for regexes (thanks sebi-hgdata)
* #243 - Use ephemeral ports to communicate with minions

### 1.1.7

* #196 - Raise minimum java version to 1.6
* #231 - Process hangs

### 1.1.6

* #10  - Add maven report goal (thanks jasonmfehr) 
* #184 - Remove undocumented project file feature
* #219 - Performance improvement for report generation (thanks tobiasbaum) 
* #190 - Allow custom properties for plugins

Note this release contains a known issue (#231). Please upgrade.

### 1.1.5

* Fix for #148 - Stackoverflow with TestNG data providers when using JMockit
* Fix for #56 - Not reporting junit incompatibilities
* Fix for #174 - Invalid linecoverage.xml with static initializers 
* Fix for #183 - Can't run GWTMockito tests
* Fix for #179 - Broken `includeLaunchClasspath=false` on Windows
* #173 - Read exclusions and groups from maven surefire config

### 1.1.4

* #157         - Support maven -DskipTests flag (thanks lkwg82)
* Fix for #163 - Should not include test tree in coverage threshold
* #166         - Allow classpath exclusions for maven plugin (thanks TomRK1089)
* #155         - Restore Java 5 compatibility
* Fix for #148 - Issue with JMockit + TestNG (thanks estekhin and KyleRogers)

### 1.1.3

* Fix for #158 - Tests incorrectly excluded from mutants
* Fix for #153 - SCM plugin broken for maven 2
* Fix for #152 - Does not work with IBM jdk

### 1.1.2

* Fix for #150 - line coverage under reported

### 1.1.1

* Block based coverage (fixes 79/131)
* End support for running on Java 5 (java 5 bytecode still supported)
* Skip flag for maven modules (#106)
* Stop declaring TestNG as a dependency 
* New parameter propagation mutator (thanks UrsMetz)

### 1.1.0

* Change scheme for identifying mutants (see https://github.com/hcoles/pitest/issues/125)
* Support alternate test apis via plugin system
* Report error when supplied mutator name does not match (thanks artspb)
* Report exit codes from coverage child process (thanks KyleRogers)
* Treat JUnit tests with ClassRule annotation as one unit (thanks devmop)

Please note that any stored history files or sonar results are invalidated by this release.

### 1.0.0

* Switch version numbering scheme
* Upgrade to ASM 5.0.2
* Fix for #114 - fails to run for java 8 when -parameters flag is set
* #99 Support additionalClasspathElements property in maven plugin (thanks artspb)
* #98 Do not mutate java 7 try with resources (thanks @artspb)
* #109 extended remove conditional mutator (thanks @vrthra)


### 0.33

* Move to Github
* Upgrade of ASM to support Java 8 bytecode (thanks to "iirekm") 
* Partial support for JUnit categories (thanks to "chrisr")
* New Remove Increments Mutator (thanks to Rahul Gopinath)
* Minor logging improvements (thanks to Kyle Rogers aka Stephan Penndorf)
* Fix for #92 - broken maven 2 support
* Fix for #75 - incorrectly ignored tests in classes with both @Ignore and @BeforeClass / @AfterClass

### 0.32

* restores java 7 compatibility
* new remove conditionals mutator
* support for mutating static initializers with TestNG
* properly isolate classpaths when running via Ant
* break builds on coverage threshold
* allow JVM to be specified 
* support user defined test selection strategies
* support user defined output format
* support user defined test prioritisation
* fix for issue blocking usage with [Robolectric](http://robolectric.org/)

Note, setup for Ant based projects changes in this release. See [ant setup](http://pitest.org/quickstart/ant/) for details on usage.

### 0.31

* Maven 2 compatibility restored
* Much faster line coverage calculation
* Fix for #78 - Error when PowerMockito test stores mock as member

This release also changes a number of internal implementation details, some of which may be of interest/importance to those maintaining tools that
integrate with PIT.

Mutations are now scoped internally as described in [https://groups.google.com/forum/#!topic/pitusers/E0-3QZuMYjE](https://groups.google.com/forum/#!topic/pitusers/E0-3QZuMYjE)

A new class (org.pitest.mutationtest.tooling.EntryPoint) has been introduced that removes some of the duplication that existed in the various ways of launching mutation analysis. 

### 0.30

* Support for parametrized [Spock](http://code.google.com/p/spock/) tests
* Support for [JUnitParams](http://code.google.com/p/junitparams/) tests
* Fix for #73 - JUnit parameterised tests calling mutee during setup failing during mutation phase
* Fix to #63 - ant task fails when empty options supplied
* Ability to override maven options from command line
* Ability to fail a build if it does not achieve a given mutation score
* Performance improvement when tests use @BeforeClass or @AfterClass annotations
* Slightly improved scheduling over multiple threads
* Improved maven multi project support
* Integration with source control for maven users


### 0.29

* Incremental analysis (--historyInputLocation and --historyOutputLocation)
* Inlined code detection turned on by default
* Quieter loggging by default
* Improved Java 7 support
* Upgrade of ASM from 3.3 to 4
* Fix for concurrency issues during coverage collection
* Fix for #53 - problems with snapshot junit versions
* Fix for #59 - duplicate dependencies set via maven


### 0.28

* Inlined finally block detection (--detectInlinedCode)
* New experimental switch statement mutator (contributed by Chris Rimmer)
* Do not mutate Groovy classes
* Fix for #33 - set user.dir to match surefire
* Fix for #43 - optionally suppress timestamped folders (--timestampedReports=true/false)
* Fix for #44 - concurrent modification exception when gathering coverage
* Fix for #46 - incorrect setting of flags by ant task
* Smaller memory footprint for main process
* Faster coverage gathering for large codebases
* Faster classpath scanning for large codebases
* Support for JUnit 3 suite methods
* Fixes for incorrect detection of JUnit 3 tests

**Known issue** - Fix for #33 may not resolve issue for maven 2 users.

Detection of Groovy code has not yet been tested with Groovy 2 which may generate substantially different
byte code to earlier versions.

### 0.27

* Much prettier reports
* Now avoids mutating assert statements
* Removed inScopeClasses option - use targetClasses and targetTests instead
* Fix for 100% CPU usage when child JVM crashes
* Fix for #35 #38 - experimental member variable mutator now corrects stack
* Fix for #39 - order of classpath elements now maintained when running from maven

**Upgrading users may need to modify their build due to removal of the inScopeClasses parameter**

### 0.26

* Ant support
* New experimental mutator for member variables 
* Fix for #12 #27 - no longer hangs when code under test launches non daemon threads
* Fix for #26 - now warns when no test library found on classpath
* Fix for #30 - now errors if mutated classes have no line or source debug
* Fix for #32 - now correctly handles of JUnit assumptions 

**Known issue** - The new member variable mutator may cause errors in synchronized errors. The mutator is
however disabled by default, and the generated errors are correctly handled by PIT.

### 0.25

* TestNG support (experimental)
* Fix for issue where mutations in nested classes not isolated from each other
* Fix for broken classpath isolation for projects using xstream
* Improved handling of JUnit parametrized tests
* Ability to limit mutations to specific classpath roots (--mutableCodePaths)
* Ability to add non launch classpath roots (--classPath) (experimental)
* Read configuration values from XML (experimental)
* Option to not throw error when no mutations found
* Consistent ordering of classes in HTML report
* Statistics written to console
* Classes no longer loaded during initial classpath scanning
* New syntax to easily enable all mutation operations

### 0.24

* JMockit support
* Option to output results in XML or CSV
* Fix for #11
* Improved INLINE_CONSTS mutator

### 0.23

* Fix for issue 7 - source files not located

### 0.22

* Upgrade of Xstream to 1.4.1 to enable OpenJDK 7 support
* Fix for #5 - corruption of newline character in child processes
* Ability to set child process launch arguments

### 0.21

* Significant performance improvements
* Support for powermock via both classloader (requires PowerMockIgnore annotation) and java agent
* Minor error reporting and usability improvements
* Fix for major defect around dependency analysis
* PIT dependencies no longer placed on classpath when running via maven
* Support for excluding certain classes or tests
* Support for verbose logging

### 0.20

* Limit number of mutations per class
* Upgrade xstream to 1.3.1
* Make available from maven central

### 0.19

* Built in enum methods now excluded from mutation
* Fixed bug around reporting of untested classes
* Support for excluding tests greater than a certain distance from class
* Support for excluding methods from mutation analysis
* Performance improvements
* Removed support for launching mutation reports from JUnit runner

### 0.18

* First public release

## Credits

Pitest is mainly the work of [me](https://twitter.com/0hjc) but has benefited from contributions from many others. 

Notable contributions not visible [here](https://github.com/hcoles/pitest/graphs/contributors) as they were made before this code was migrated to github include 

* Nicolas Rusconi - Ant Task
* Struan Kerr-Liddell - Improvements to html report
* Stephan Pendorf - Multiple improvments including improved mutators
 
Although PIT does not incorporate any code from the Jumble project (http://jumble.sourceforge.net/), the Jumble codebase was used as a guide when developing some aspects of PIT.

## Other stuff

The codebase is checked up on in a few places that give slower feedback than the github hooks.

[maven2 on IBM JDK check](https://hjc.ci.cloudbees.com/job/maven2_triangle_example/)

[Sonarqube analysis](http://nemo.sonarqube.org/dashboard/index/793182)




OutSystems Now (Android)
========================

OutSystems Now brings your OutSystems experience to any device, providing a fast 
way to access all your applications, including CRM, Customer Portal, or any other 
custom app built with the OutSystems Platform.

The OutSystems Now source code is made available for you to create your own version
of the application. This way you can apply your own branding, which includes the 
application name, logo and splash screens. You can also have control over push
notifications and enhance your business with a true mobile experience.

Requirements
=============

You will required an Android development environment such as Eclipse and a Google developer
license (if you wish to submit the application to Google Play). The newly created
application will be under you responsibility and support.

LICENSE
=======

This application is released under the MIT License.


# sample-apps-android-sdk22
Android Sample, requires Android SDK 22


#RxMVVM

My way to MVVM using RxJava with new Android databinding

##Summary
* Use [MVVM][1] to separate Android Framework with a [clean architecture][2] to my domain logic.
* Use [Android Databinding][3] to glue view model and Android
* Asynchronous communications implemented with [Rx][4].
* Rest API from [ComicVine][5]
* Use [Frodo][6] to debug Rx

TODO LIST
---------

* Better UI, with Material Design concepts and so on
* Add unit tests, allways fail on that :(
* Add Dependency Injection
* [WIP] Implement an Annotation processor to remove most of the View Model boilerplate code 
* Implement a local datasource with Realm to test it


Developed By
------------

Fernando Franco Giráldez - <ffrancogiraldez@gmail.com>

<a href="https://twitter.com/thanerian">
  <img alt="Follow me on Twitter" src="http://imageshack.us/a/img812/3923/smallth.png" />
</a>
<a href="http://es.linkedin.com/pub/fernando-franco-giraldez/22/803/b44/es">
  <img alt="Add me to Linkedin" src="http://imageshack.us/a/img41/7877/smallld.png" />
</a>

License
-------

    Copyright 2015 Fernando Franco Giráldez
    Licensed under the Apache License, Version 2.0 (the "License");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.
    
[1]: https://en.wikipedia.org/wiki/Model_View_ViewModel
[2]: http://blog.8thlight.com/uncle-bob/2012/08/13/the-clean-architecture.html
[3]: http://developer.android.com/intl/es/tools/data-binding/guide.html
[4]: http://reactivex.io/
[5]: http://www.comicvine.com/api/
[6]: https://github.com/android10/frodo


<div align="center">
  <img src="https://www.tensorflow.org/images/tf_logo_transp.png"><br><br>
</div>
-----------------

|  **`Linux CPU`**   |  **`Linux GPU PIP`** | **`Mac OS CPU`** |  **`Android`** |
|-------------------|----------------------|------------------|----------------|
| [![Build Status](http://ci.tensorflow.org/buildStatus/icon?job=tensorflow-master-cpu)](http://ci.tensorflow.org/job/tensorflow-master-cpu) | [![Build Status](http://ci.tensorflow.org/buildStatus/icon?job=tensorflow-master-gpu_pip)](http://ci.tensorflow.org/job/tensorflow-master-gpu_pip) | [![Build Status](http://ci.tensorflow.org/buildStatus/icon?job=tensorflow-master-mac)](http://ci.tensorflow.org/job/tensorflow-master-mac) | [![Build Status](http://ci.tensorflow.org/buildStatus/icon?job=tensorflow-master-android)](http://ci.tensorflow.org/job/tensorflow-master-android) |

**TensorFlow** is an open source software library for numerical computation using
data flow graphs.  Nodes in the graph represent mathematical operations, while
the graph edges represent the multidimensional data arrays (tensors) that flow
between them.  This flexible architecture lets you deploy computation to one
or more CPUs or GPUs in a desktop, server, or mobile device without rewriting
code.  TensorFlow also includes TensorBoard, a data visualization toolkit.

TensorFlow was originally developed by researchers and engineers
working on the Google Brain team within Google's Machine Intelligence research
organization for the purposes of conducting machine learning and deep neural
networks research.  The system is general enough to be applicable in a wide
variety of other domains, as well.

**If you'd like to contribute to TensorFlow, be sure to review the [contribution
guidelines](CONTRIBUTING.md).**

**We use [GitHub issues](https://github.com/tensorflow/tensorflow/issues) for
tracking requests and bugs, but please see
[Community](tensorflow/g3doc/resources/index.md#community) for general questions
and discussion.**

## Installation
*See [Download and Setup](tensorflow/g3doc/get_started/os_setup.md) for instructions on how to install our release binaries or how to build from source.*

People who are a little bit adventurous can also try our nightly binaries:

* Linux CPU only: [Python 2](http://ci.tensorflow.org/view/Nightly/job/nightly-matrix-cpu/TF_BUILD_CONTAINER_TYPE=CPU,TF_BUILD_IS_OPT=OPT,TF_BUILD_IS_PIP=PIP,TF_BUILD_PYTHON_VERSION=PYTHON2,label=cpu-slave/lastSuccessfulBuild/artifact/pip_test/whl/tensorflow-0.9.0-cp27-none-linux_x86_64.whl) ([build history](http://ci.tensorflow.org/view/Nightly/job/nightly-matrix-cpu/TF_BUILD_CONTAINER_TYPE=CPU,TF_BUILD_IS_OPT=OPT,TF_BUILD_IS_PIP=PIP,TF_BUILD_PYTHON_VERSION=PYTHON2,label=cpu-slave/)) / [Python 3.4](http://ci.tensorflow.org/view/Nightly/job/nightly-matrix-cpu/TF_BUILD_CONTAINER_TYPE=CPU,TF_BUILD_IS_OPT=OPT,TF_BUILD_IS_PIP=PIP,TF_BUILD_PYTHON_VERSION=PYTHON3,label=cpu-slave/lastSuccessfulBuild/artifact/pip_test/whl/tensorflow-0.9.0-cp34-cp34m-linux_x86_64.whl) ([build history](http://ci.tensorflow.org/view/Nightly/job/nightly-matrix-cpu/TF_BUILD_CONTAINER_TYPE=CPU,TF_BUILD_IS_OPT=OPT,TF_BUILD_IS_PIP=PIP,TF_BUILD_PYTHON_VERSION=PYTHON3,label=cpu-slave/)) / [Python 3.5](http://ci.tensorflow.org/view/Nightly/job/nightly-python35-linux-cpu/lastSuccessfulBuild/artifact/pip_test/whl/tensorflow-0.9.0-cp35-cp35m-linux_x86_64.whl) ([build history](http://ci.tensorflow.org/view/Nightly/job/nightly-python35-linux-cpu/))
* Linux GPU: [Python 2](http://ci.tensorflow.org/view/Nightly/job/nigntly-matrix-linux-gpu/TF_BUILD_CONTAINER_TYPE=GPU,TF_BUILD_IS_OPT=OPT,TF_BUILD_IS_PIP=PIP,TF_BUILD_PYTHON_VERSION=PYTHON2,label=gpu-linux/lastSuccessfulBuild/artifact/pip_test/whl/tensorflow-0.9.0-cp27-none-linux_x86_64.whl) ([build history](http://ci.tensorflow.org/view/Nightly/job/nigntly-matrix-linux-gpu/TF_BUILD_CONTAINER_TYPE=GPU,TF_BUILD_IS_OPT=OPT,TF_BUILD_IS_PIP=PIP,TF_BUILD_PYTHON_VERSION=PYTHON2,label=gpu-linux/)) / [Python 3.4](http://ci.tensorflow.org/view/Nightly/job/nigntly-matrix-linux-gpu/TF_BUILD_CONTAINER_TYPE=GPU,TF_BUILD_IS_OPT=OPT,TF_BUILD_IS_PIP=PIP,TF_BUILD_PYTHON_VERSION=PYTHON3,label=gpu-linux/lastSuccessfulBuild/artifact/pip_test/whl/tensorflow-0.9.0-cp34-cp34m-linux_x86_64.whl) ([build history](http://ci.tensorflow.org/view/Nightly/job/nigntly-matrix-linux-gpu/TF_BUILD_CONTAINER_TYPE=GPU,TF_BUILD_IS_OPT=OPT,TF_BUILD_IS_PIP=PIP,TF_BUILD_PYTHON_VERSION=PYTHON3,label=gpu-linux/)) / [Python 3.5](http://ci.tensorflow.org/view/Nightly/job/nigntly-matrix-linux-gpu/TF_BUILD_CONTAINER_TYPE=GPU,TF_BUILD_IS_OPT=OPT,TF_BUILD_IS_PIP=PIP,TF_BUILD_PYTHON_VERSION=PYTHON3.5,label=gpu-linux/lastSuccessfulBuild/artifact/pip_test/whl/tensorflow-0.9.0-cp35-cp35m-linux_x86_64.whl) ([build history](http://ci.tensorflow.org/view/Nightly/job/nigntly-matrix-linux-gpu/TF_BUILD_CONTAINER_TYPE=GPU,TF_BUILD_IS_OPT=OPT,TF_BUILD_IS_PIP=PIP,TF_BUILD_PYTHON_VERSION=PYTHON3.5,label=gpu-linux/))
* Mac CPU only: [Python 2](http://ci.tensorflow.org/view/Nightly/job/nightly-matrix-cpu/TF_BUILD_CONTAINER_TYPE=CPU,TF_BUILD_IS_OPT=OPT,TF_BUILD_IS_PIP=PIP,TF_BUILD_PYTHON_VERSION=PYTHON2,label=mac-slave/lastSuccessfulBuild/artifact/pip_test/whl/tensorflow-0.9.0-py2-none-any.whl) ([build history](http://ci.tensorflow.org/view/Nightly/job/nightly-matrix-cpu/TF_BUILD_CONTAINER_TYPE=CPU,TF_BUILD_IS_OPT=OPT,TF_BUILD_IS_PIP=PIP,TF_BUILD_PYTHON_VERSION=PYTHON2,label=mac-slave/)) / [Python 3](http://ci.tensorflow.org/view/Nightly/job/nightly-matrix-cpu/TF_BUILD_CONTAINER_TYPE=CPU,TF_BUILD_IS_OPT=OPT,TF_BUILD_IS_PIP=PIP,TF_BUILD_PYTHON_VERSION=PYTHON3,label=mac-slave/lastSuccessfulBuild/artifact/pip_test/whl/tensorflow-0.9.0-py3-none-any.whl) ([build history](http://ci.tensorflow.org/view/Nightly/job/nightly-matrix-cpu/TF_BUILD_CONTAINER_TYPE=CPU,TF_BUILD_IS_OPT=OPT,TF_BUILD_IS_PIP=PIP,TF_BUILD_PYTHON_VERSION=PYTHON3,label=mac-slave/))
* [Android](http://ci.tensorflow.org/view/Nightly/job/nightly-matrix-android/TF_BUILD_CONTAINER_TYPE=ANDROID,TF_BUILD_IS_OPT=OPT,TF_BUILD_IS_PIP=NO_PIP,TF_BUILD_PYTHON_VERSION=PYTHON2,label=android-slave/lastSuccessfulBuild/artifact/bazel-out/local_linux/bin/tensorflow/examples/android/tensorflow_demo.apk) ([build history](http://ci.tensorflow.org/view/Nightly/job/nightly-matrix-android/TF_BUILD_CONTAINER_TYPE=ANDROID,TF_BUILD_IS_OPT=OPT,TF_BUILD_IS_PIP=NO_PIP,TF_BUILD_PYTHON_VERSION=PYTHON2,label=android-slave/))

#### *Try your first TensorFlow program*
```shell
$ python
```
```python
>>> import tensorflow as tf
>>> hello = tf.constant('Hello, TensorFlow!')
>>> sess = tf.Session()
>>> sess.run(hello)
Hello, TensorFlow!
>>> a = tf.constant(10)
>>> b = tf.constant(32)
>>> sess.run(a+b)
42
>>>
```

##For more information

* [TensorFlow website](http://tensorflow.org)
* [TensorFlow whitepaper](http://download.tensorflow.org/paper/whitepaper2015.pdf)
* [TensorFlow MOOC on Udacity] (https://www.udacity.com/course/deep-learning--ud730)

The TensorFlow community has created amazing things with TensorFlow, please see the [resources section of tensorflow.org](https://www.tensorflow.org/versions/master/resources#community) for an incomplete list.


# [FitNesse](http://fitnesse.org/)  [![maven central](https://maven-badges.herokuapp.com/maven-central/org.fitnesse/fitnesse/badge.svg?style=flat)](https://maven-badges.herokuapp.com/maven-central/org.fitnesse/fitnesse)

Welcome to FitNesse, the fully integrated stand-alone acceptance testing framework and wiki.

To get started, check out [http://fitnesse.org](http://fitnesse.org)!



## Quick start

* [A One-Minute Description of FitNesse](http://fitnesse.org/FitNesse.UserGuide.OneMinuteDescription)
* [Download FitNesse](http://fitnesse.org/FitNesseDownLoad) and [Plugins](http://fitnesse.org/PlugIns)
* [The FitNesse User Guide](http://fitnesse.org/.FitNesse.UserGuide)



## Bug tracker

Have a bug or a feature request? [Please open a new issue](https://github.com/unclebob/fitnesse/issues). 


## Community

Have a question that's not a feature request or bug report? [Ask on the mailing list.](http://groups.yahoo.com/group/fitnesse)

## Edge builds

The latest stable build of FitNesse can be [downloaded here](https://cleancoder.ci.cloudbees.com/job/fitnesse/lastStableBuild/).

**Note**: the edge Jenkins build produces 2 jars. `fitnesse.jar` is for use in Maven or Ivy. Users who just want to run FitNesse by itself should download `fitnesse-standalone.jar` instead of `fitnesse.jar`.

## Developers

Issues and pull requests are administered at [GitHub](https://github.com/unclebob/fitnesse/issues).

### Building

[Apache Ant](http://ant.apache.org/) and a proper internet connection is sufficient to build FitNesse. The build process will bootstrap itself by downloading Ivy (dependency management) and from there will download the modules required to build and test FitNesse.

To build and run all tests, run the command

```
$ ant
``` 

which builds the `all` target. 

### Running

To start the FitNesse wiki locally, for example to browse the local version of the User Guide

```
$ ant run
```

### Testing

To run the unit tests:

```
$ ant unit_test
```

To run the acceptance tests:

```
$ ant acceptance_tests
```

There is a second source directory, `srcFitServerTests`, which contains units
tests that test invocation of Fit servers written in Ruby, C++, and .NET. These
tests are not run as part of the normal ant test-related targets. When using an
IDE, make sure it does not invoke these tests when running the "normal" tests
under the `src` directory.

Direct any questions to the [FitNesse Yahoo group](https://groups.yahoo.com/neo/groups/fitnesse/info).


### Working with Eclipse and IntelliJ

There are a few things to keep in mind when working from an IDE:

1. The Ant build file does some extra things apart from compiling the code.
    * It sets the FitNesse version in a META-INF/FitNesseVersion.txt
    * It copies the dependencies to the lib folder so they can be used by the acceptance tests.

   Perform a
   ```
   $ ant post-compile
   ```
   to execute those actions. In your IDE it is possible to define "post-compilation" steps. If
   you set the "post-compile" target from the build file, you won't have any trouble with
   cleaning, building and executing tests from your IDE.

2. Apache Ivy is used for dependency management. Your IDE can be set up to support Ivy.
    * In IntelliJ set IvyIDEA in "Project Structure" -> "Modules" -> "Dependencies".
    * In Eclipse, install IvyDE and set it up.

   Alternatively,
   ```
   $ ant retrieve
   ```
   will download the dependencies and copy them to lib/, from where your
   IDE can pick them up.



android-examples
================

Welcome to the source code for Android examples from the vogella.com online tutorials and books.

All of the source code in this archive is licensed under the EPL license except as noted.

Using in Android Studio
=======================

Some of the projects should have a `build.gradle` file suitable for
importing the project into Android Studio. If not please send a pull request to add one.

Note, though, that you
may need to adjust the `compileSdkVersion` in `build.gradle` if it
requests an SDK that you have not downloaded and do not wish to
download. Similarly, you may need to adjust the `buildToolsVersion`
value to refer to a version of the "Build-tools" that you have downloaded
from the SDK Manager.

Using in Eclipse
================

Most of the projects can be imported using the normal Eclipse import process. 

- Some of these projects are not set up to support Eclipse, because
the nature of the project is to demonstrate something specific for
Android Studio or Gradle for Android.

- Some of these projects are not set up to support Eclipse, as Eclipse
is no longer officially enhanced by Google, and so new examples are developed with Android Studio.





EventBus
========
EventBus is publish/subscribe event bus optimized for Android.<br/>
<img src="EventBus-Publish-Subscribe.png" width="500" height="187"/>

EventBus...

 * simplifies the communication between components
    * decouples event senders and receivers
    * performs well with Activities, Fragments, and background threads
    * avoids complex and error-prone dependencies and life cycle issues
 * makes your code simpler
 * is fast
 * is tiny (<50k jar)
 * is proven in practice by apps with 100,000,000+ installs
 * has advanced features like delivery threads, subscriber priorities, etc.

 [![Build Status](https://travis-ci.org/greenrobot/EventBus.svg?branch=master)](https://travis-ci.org/greenrobot/EventBus)

EventBus in 3 steps
-------------------
1. Define events:<br/>
<code>public class MessageEvent { /* Additional fields if needed */ }</code><br/><br/>
2. Prepare subscribers:<br/>
<code>eventBus.register(this);</code><br/>
<code>public void onEvent(AnyEventType event) {/* Do something */};</code><br/><br/>
3. Post events:<br/>
<code>eventBus.post(event);</code>

Add EventBus to your project
----------------------------
EventBus is available on Maven Central. Please ensure that you are using the latest version by [checking here](http://search.maven.org/#search%7Cga%7C1%7Cg%3A%22de.greenrobot%22%20AND%20a%3A%22eventbus%22)

Gradle:
```
    compile 'de.greenrobot:eventbus:2.4.0'
```

Maven:
```
<dependency>
    <groupId>de.greenrobot</groupId>
    <artifactId>eventbus</artifactId>
    <version>2.4.0</version>
</dependency>
```

[Or download EventBus from Maven Central](http://search.maven.org/#search%7Cga%7C1%7Cg%3A%22de.greenrobot%22%20AND%20a%3A%22eventbus%22)

How-to, Developer Documentation
-------------------------------
Details on EventBus and its API are available in the [HOWTO document](HOWTO.md).

How does EventBus compare to other solutions, like Otto from Square? Check this [comparison](COMPARISON.md).

Additional Features and Notes
-----------------------------

* **NOT based on annotations:** Querying annotations are slow on Android, especially before Android 4.0. Have a look at this [Android bug report](http://code.google.com/p/android/issues/detail?id=7811).
* **Based on conventions:** Event handling methods are called "onEvent".
* **Performance optimized:** It's probably the fastest event bus for Android.
* **Convenience singleton:** You can get a process wide event bus instance by calling EventBus.getDefault(). You can still call new EventBus() to create any number of local busses.
* **Subscriber and event inheritance:** Event handler methods may be defined in super classes, and events are posted to handlers of the event's super classes including any implemented interfaces. For example, subscriber may register to events of the type Object to receive all events posted on the event bus.

FAQ
---
**Q:** How is EventBus different to Android's BroadcastReceiver/Intent system?<br/>
**A:** Unlike Android's BroadcastReceiver/Intent system, EventBus uses standard Java classes as events and offers a more convenient API. EventBus is intended for a lot more uses cases where you wouldn't want to go through the hassle of setting up Intents, preparing Intent extras, implementing broadcast receivers, and extracting Intent extras again. Also, EventBus comes with a much lower overhead.

 **Q:** How to do pull requests?<br/>
 **A:** Ensure good code quality and consistent formatting. EventBus has a good test coverage: if you propose a new feature or fix a bug, please add a unit test.

Release History, License
------------------------
[CHANGELOG](CHANGELOG.md)

EventBus binaries and source code can be used according to the [Apache License, Version 2.0](LICENSE).

More Open Source by greenrobot
==============================
[__greenrobot-common__](https://github.com/greenrobot/greenrobot-common) is a set of utility classes and hash functions for Android & Java projects.

[__greenDAO__](https://github.com/greenrobot/greenDAO) is an ORM optimized for Android: it maps database tables to Java objects and uses code generation for optimal speed.

[Follow us on Google+](https://plus.google.com/b/114381455741141514652/+GreenrobotDe/posts) to stay up to date.


Android phased seek bar
=======================

android phased seek bar allow create a seek bar with predefined states.

[![Android Arsenal](https://img.shields.io/badge/Android%20Arsenal-android--phased--seek--bar-brightgreen.svg?style=flat)](https://android-arsenal.com/details/1/919)

Preview
=======
![preview](https://raw.githubusercontent.com/ademar111190/android-phased-seek-bar/master/images/sample.gif)

License
============
GNU Lesser General Public License at version 3, more details at LICENSE


Developed By
============

* Ademar Oliveira - <ademar111190@gmail.com>


This README.md file is displayed on your project page. You should edit this 
file to describe your project, including instructions for building and 
running the project, pointers to the license under which you are making the 
project available, and anything else you think would be useful for others to
know.

We have created an empty license.txt file for you. Well, actually, it says,
"<Replace this text with the license you've chosen for your project.>" We 
recommend you edit this and include text for license terms under which you're
making your code available. A good resource for open source licenses is the 
[Open Source Initiative](http://opensource.org/).

Be sure to update your project's profile with a short description and 
eye-catching graphic.

Finally, consider defining some sprints and work items in Track & Plan to give 
interested developers a sense of your cadence and upcoming enhancements.


# This is my README


Neuroph - Java Neural Network Platform Neuroph
======

Neuroph is lightweight Java neural network framework to develop common neural network architectures. 
It contains well designed, open source Java library with small number of basic classes which correspond to basic NN concepts. 
Also has nice GUI neural network editor to quickly create Java neural network components. 
It has been released as open source under the Apache 2.0 license, and it's FREE for you to use it.


Android Signature Pad
====================

Android Signature Pad is an Android library for drawing smooth signatures. It uses variable width Bézier curve interpolation based on [Smoother Signatures](http://corner.squareup.com/2012/07/smoother-signatures.html) post by [Square](https://squareup.com).

![Screenshot](https://github.com/gcacace/android-signaturepad/raw/master/header.png)

## Features
 * Bézier implementation for a smoother line
 * Variable point size based on velocity
 * Customizable pen color and size
 * Bitmap and SVG support

##Installation

Latest version of the library can be found on Maven Central.

### For Gradle users

Open your `build.gradle` and make sure that Maven Central repository is declared into `repositories` section:
```gradle
   repositories {
       mavenCentral()
   }
```
Then, include the library as dependency:
```gradle
compile 'com.github.gcacace:signature-pad:1.2.0'
```

### For Maven users

Add this dependency to your `pom.xml`:
```xml
<dependency>
  <groupId>com.github.gcacace</groupId>
  <artifactId>signature-pad</artifactId>
  <version>1.2.0</version>
  <type>aar</type>
</dependency>
```
 
##Usage

*Please see the `/SignaturePad-Example` app for a more detailed code example of how to use the library.*

1. Add the `SignaturePad` view to the layout you want to show.
    ```xml

 <com.github.gcacace.signaturepad.views.SignaturePad
     xmlns:android="http://schemas.android.com/apk/res/android"
     xmlns:app="http://schemas.android.com/apk/res-auto"
     android:id="@+id/signature_pad"
     android:layout_width="match_parent"
     android:layout_height="match_parent"
     app:penColor="@android:color/black"
     />
    ```
2. Configure attributes.
 * `penMinWidth` - The minimum width of the stroke (default: 3dp).
 * `penMaxWidth` - The maximum width of the stroke (default: 7dp).
 * `penColor` - The color of the stroke (default: Color.BLACK).
 * `velocityFilterWeight` - Weight used to modify new velocity based on the previous velocity (default: 0.9).
 * `clearOnDoubleClick` - Double click to clear pad (default: false) 

3. Configure signature events listener

 An `OnSignedListener` can be set on the view:
  ```java
  
 mSignaturePad = (SignaturePad) findViewById(R.id.signature_pad);
 mSignaturePad.setOnSignedListener(new SignaturePad.OnSignedListener() {
     @Override
     public void onSigned() {
         //Event triggered when the pad is signed
     }
 
     @Override
     public void onClear() {
         //Event triggered when the pad is cleared
     }
 });
  ```
 
4. Get signature data
 * `getSignatureBitmap()` - A signature bitmap with a white background.
 * `getTransparentSignatureBitmap()` - A signature bitmap with a transparent background.
 * `getSignatureSvg()` - A signature Scalable Vector Graphics document.

## Cordova Plugin

Thanks to [netinhoteixeira](https://github.com/netinhoteixeira/), there is a Cordova plugin using that library.
Please refer to https://github.com/netinhoteixeira/cordova-plugin-signature-view.

## NativeScript Plugin
Thanks to [bradmartin](https://github.com/bradmartin), there is a NativeScript plugin.
Please refer to [https://github.com/bradmartin/nativescript-signaturepad](https://github.com/bradmartin/nativescript-signaturepad).

## Caveats

Currently doesn't support screen rotations. Pull requests are welcome!

## License

    Copyright 2014-2016 Gianluca Cacace

    Licensed under the Apache License, Version 2.0 (the "License");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.


