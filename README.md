# Gradle Clover plugin

![Clover Logo](https://www.appfusions.com/download/attachments/13959273/logoClover.png?version=1&modificationDate=1353815596170)

The plugin provides generation of code coverage reports using [Clover](http://www.atlassian.com/software/clover/).

## Usage

To use the Clover plugin, include in your build script:

    apply plugin: 'clover'

The plugin JAR needs to be defined in the classpath of your build script. It is directly available on
[Maven Central](http://search.maven.org/#search%7Cgav%7C1%7Cg%3A%22org.gradle.api.plugins%22%20AND%20a%3A%22gradle-clover-plugin%22).
Alternatively, you can download it from GitHub and deploy it to your local repository. The following code snippet shows an
example on how to retrieve it from Maven Central:

    buildscript {
        repositories {
            mavenCentral()
        }

        dependencies {
            classpath 'org.gradle.api.plugins:gradle-clover-plugin:0.7'
        }
    }

To define the Clover dependency please use the `clover` configuration name in your `dependencies` closure. If you are working
with a multi-module project make sure you apply the plugin and declare the `clover` dependency within the `allprojects` closure.

    dependencies {
        clover 'com.cenqua.clover:clover:3.1.10'
    }

## Tasks

The Clover plugin defines the following tasks:

* `cloverGenerateReport`: Generates Clover code coverage report.
* `cloverAggregateReports`: Aggregate Clover code coverage reports in a multi-module project setup. This task can only be
run from the root directory of your project and requires at least one submodule. This task depends on `cloverGenerateReport`.
* `cloverAggregateReportsWithHistory`: Aggregate Clover code coverage reports including historical data in a multi-module project setup. This task can only be
run from the root directory of your project and requires at least one submodule. This task depends on `cloverGenerateReport`.

## Convention properties

* `initString`: The location to write the Clover coverage database (defaults to `.clover/clover.db`). The location you
define will always be relative to the project's build directory.
* `classesBackupDir`: The temporary backup directory for classes (defaults to `file("${sourceSets.main.classesDir}-bak")`).
* `licenseLocation`: The [Clover license](http://confluence.atlassian.com/display/CLOVER/How+to+configure+your+clover.license)
to be used (defaults to `'clover.license'`). The location can either be a file or URL defined as String.
* `includes`: A list of String Ant Glob Patterns to include for instrumentation (defaults to `'**/*.java'` for Java projects, defaults
to `'**/*.java'` and `'**/*.groovy'` for Groovy projects).
* `excludes`: A list of String Ant Glob Patterns to exclude for instrumentation. By default no files are excluded.
* `testIncludes`: A list of String Ant Glob Patterns to include for instrumentation for
[per-test coverage](http://confluence.atlassian.com/display/CLOVER/Unit+Test+Results+and+Per-Test+Coverage) (defaults to
`'**/*Test.java'` for Java projects, defaults to `'**/*Test.java'` and `'**/*Test.groovy'` for Groovy projects).
* `additionalSourceDirs`: Defines custom source sets to be added for instrumentation e.g. `sourceSets.custom.allSource.srcDirs`.
* `additionalTestDirs`: Defines custom test source sets to be added for instrumentation e.g. `sourceSets.integTest.allSource.srcDirs`.
* `targetPercentage`: The required target percentage total coverage e.g. "10%". The build fails if that goals is not met.
If not specified no target percentage will be checked.
* `optimizeTests`: If `true`, Clover will try to [optimize your tests](https://confluence.atlassian.com/display/CLOVER/About+Test+Optimization);
if `false` Clover will not try to optimize your tests. Test optimization is disabled by default. Note that Clover does not
yet fully support test optimization for Groovy code; see [CLOV-1152](https://jira.atlassian.com/browse/CLOV-1152) for more information.
* `snapshotFile`: The location of the Clover snapshot file used for test optimization, relative to the project directory.
The snapshot file should survive clean builds, so it should *not* be placed in the project's build directory. The default
location is `.clover/coverage.db.snapshot`.
* `includeTasks`: A list of task names, allows to explicitly specify which test tasks should be introspected and used to gather coverage information - useful if there are more than one `Test` tasks in a project.
* `excludeTasks`: A list of task names, allows to exclude test tasks from introspection and gathering coverage information - useful if there are more than one `Test` tasks in a project.
* `historyDir`: The location to write the Clover history database (defaults to `.clover/history`).
* `classpath.plusConfigurations`: The configurations whose files are to be added as classpath entries. 

Within `clover` you can define [coverage contexts](http://confluence.atlassian.com/display/CLOVER/Using+Coverage+Contexts)
in a closure named `contexts`. There are two types of coverage contexts: statement contexts and method contexts. You can
define as many contexts as you want. Each of the context type closure define the following attributes:

* `name`: A unique name that identifies the context. If you want to apply the context to your report you got to use this
name as part of report `filter` attribute.
* `regexp`: The specified structure or pattern that matches a context as part of the instrumented source code. Make sure
that you correctly escape special characters.

Within `clover` you can define which report types should be generated in a closure named `report`:

* `xml`: Generates XML report (defaults to `true`).
* `json`: Generates JSON report (defaults to `false`).
* `html`: Generates HTML report (defaults to `false`).
* `pdf`: Generates PDF report (defaults to `false`).
* `filter`: A comma or space separated list of contexts to exclude when generating coverage reports.
See [Using Coverage Contexts](http://confluence.atlassian.com/display/CLOVER/Using+Coverage+Contexts). By default no filter
is applied.

The Clover plugin defines the following convention properties in the `clover` closure:

### Example

    clover {
        classesBackupDir = file("${sourceSets.main.classesDir}-backup")
        licenseLocation = 'clover-license.txt'
        excludes = ['**/SynchronizedMultiValueMap.java']
        targetPercentage = '85%'

        contexts {
            statement {
                name = 'log'
                regexp = '^.*LOG\\..*'
            }

            method {
                name = 'main'
                regexp = 'public static void main\\(String args\\[\\]\\).*'
            }
        }

        report {
            html = true
            pdf = true
            filter = 'log,if,else'
        }
    }

## Project Properties

The Clover plugin defines the following properties:

* `-PcloverInstrumentedJar`: The `jar` task uses clover to build a JAR with clover instrumented classes. This property can be used to prepare a JAR or EAR for distributed code coverage. 

When mixing `-PcloverInstrumentedJar` and any of the tasks, make sure the "jar" task is executed before "test" (i.e. `gradle jar test cloverGenerateReport -PcloverInstrumentedJar`)


## FAQ

**How do I use the code coverage results produced by this plugin with the Gradle Sonar plugin?**

You will have to configure the [Gradle Sonar plugin](http://www.gradle.org/docs/current/userguide/sonar_plugin.html) 
to parse the Clover coverage result file. The convention property [cloverReportPath](http://gradle.org/docs/current/groovydoc/org/gradle/api/plugins/sonar/model/SonarProject.html#cloverReportPath) 
defines the path to the code coverage XML file. This path is only used if the property [dynamicAnalysis](http://gradle.org/docs/current/groovydoc/org/gradle/api/plugins/sonar/model/SonarProject.html#dynamicAnalysis) 
has the value of `reuseReports`. The following code snippet shows an example.

    sonar {
        project {
            cloverReportPath = file("$reportsDir/clover/clover.xml")
        }
    }

In Sonar you will need to install the [Clover plugin](http://docs.codehaus.org/display/SONAR/Clover+Plugin). In order to
make Sonar use the Clover code coverage engine, the property `sonar.core.codeCoveragePlugin` must be set to `clover` under 
Configuration | General Settings | Code Coverage. If you run `gradle sonarAnalyze -i` you will see that the Gradle Sonar plugin parses your file. 
It should look something like this:

    Parsing /Users/ben/dev/projects/testproject/build/reports/clover/clover.xml
