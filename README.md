# hsac-fitnesse-fixtures
[![Maven Central](https://img.shields.io/maven-central/v/nl.hsac/hsac-fitnesse-fixtures.svg?maxAge=86400)](https://mvnrepository.com/artifact/nl.hsac/hsac-fitnesse-fixtures)

This project assists in testing (SOAP) web services and web applications by providing an application to define and run tests. To this end it contains a baseline installation of FitNesse (an acceptance testing wiki framework) and some FitNesse fixture (base) classes.

The fixtures provided aim to assist in testing (SOAP) web services and web applications (using Selenium) minimizing the amount of (custom) Java code needed to define tests.

The baseline FitNesse installation offers the following features:
* Ability to easily create a standalone (no JDK or Maven required) FitNesse environment.
* Run FitNesse tests on a build server, reporting the results in both JUnit XML format and HTML.
* [HSAC's fitnesse-plugin](https://github.com/fhoeben/hsac-fitnesse-plugin) to add additional Wiki features (random values, calculating relative dates, 
  Slim scenarios without need to specify all parameters, Slim scripts that take a screenshot after each step).
* [Praegus' toolchain-plugin](https://github.com/praegus/toolchain-fitnesse-plugin), improving the wiki's look and feel and page editing features, combining:
    - [Bootstrap-plus wiki theme](https://github.com/praegus/fitnesse-bootstrap-plus-theme) (Website: https://praegus.github.io/fitnesse-bootstrap-plus-theme/)
    - [Autocomplete responder](https://github.com/praegus/fitnesse-autocomplete-responder)
* FitNesse installation for test/fixture developers containing:
    - the fixture base classes (and Selenium drivers for _Chrome_, _Edge_ and _Firefox_),
    - easy fixture debugging.

The fastest way to get started: just download the 'standalone.zip' from the 
[Releases](https://github.com/fhoeben/hsac-fitnesse-fixtures/releases/latest), extract, run it (you'll just need a
Java runtime) and explore the example tests and maybe add a couple of your own.

## To create your own test project
When you want to use the project's baseline to create and maintain your own test suite, we recommend creating your own Maven project based on the project. This will allow you to run and maintain your test set, in version control, without the need to keep your own copies of dependencies (neither Selenium WebDrivers nor Java libraries).

To create a Maven project to run your own tests, based on this project's baseline (assuming you already have a working [Maven installation](https://maven.apache.org/guides/getting-started/maven-in-five-minutes.html):
 * Go to the directory below which you want to create the project.
 * Use the [archetype, `nl.hsac:fitnesse-project`,](https://github.com/fhoeben/fitnesse-project-archetype) to generate the project: 
   * (Maven should find the latest version of the archetype automatically, but it doesn't always.) Check the latest version number on the [releases page](https://github.com/fhoeben/fitnesse-project-archetype/releases/latest), and use this instead of `<latest-version>` in the next command.
   * On the commandline execute: `mvn archetype:generate -DarchetypeGroupId=nl.hsac -DarchetypeArtifactId=fitnesse-project -DarchetypeVersion=<latest-version>` (or use your IDE's ability to create a project from an archetype).
   * Answer the prompts for 'groupId', 'artifactId' and 'package' for the project.
 * _Alternatively_ you could also manually copy and update a sample project: 
   * Clone or download the [sample project](https://github.com/fhoeben/sample-fitnesse-project) to a new directory. (This is the approach used in the [Installation Guide](https://github.com/fhoeben/hsac-fitnesse-fixtures/wiki/Installation-Guide).)
   * Update the 'groupId' and 'artifactId' in the [`pom.xml`](https://github.com/fhoeben/sample-fitnesse-project/blob/master/pom.xml) downloaded as part of the sample to reflect your own organisation and project name.
   * Move the file for Java class [`FixtureDebugTest`](https://github.com/fhoeben/sample-fitnesse-project/blob/master/src/test/java/nl/hsac/fitnesse/sample_project/FixtureDebugTest.java) to a package/directory of your choice.
 * Start the wiki, as described in the generated project's [`README.md`](https://github.com/fhoeben/fitnesse-project-archetype/blob/master/src/main/resources/archetype-resources/README.md#running-locally).
 * Start writing tests (and custom fixtures if needed)...

## To create the standalone FitNesse installation:
Execute `mvn clean package -DskipTests`, the standalone installation is present in the wiki
directory and as `...-standalone.zip` file in the target directory. It can be distributed by just copying the wiki directory or by copying and extracting the zip file to a location without spaces in its own name, or in its parent's names).
This standalone installation can be started using `java -jar fitnesse-standalone.jar` from the wiki directory (or directory where the _standalone.zip_ was extracted).

A zip file containing released versions of this project can be downloaded from the [Releases](https://github.com/fhoeben/hsac-fitnesse-fixtures/releases/latest) or [Maven Central](https://repository.sonatype.org/service/local/artifact/maven/redirect?r=central-proxy&g=nl.hsac&a=hsac-fitnesse-fixtures&c=standalone&p=zip&v=RELEASE).
A similar zip file containing the latest *snapshot* (i.e. not released but based on the most recent code) version is published as part of the automated build of this project at https://fhoeben.github.io/hsac-fitnesse-fixtures-test-results/hsac-fitnesse-fixtures-snapshot-standalone.zip.

## To run the tests on a build server
Tests can be run on a build (or Continuous Integration) server in two ways. One can either gather the dependencies, compile and run tests using a build tool such as Maven or Gradle. Or one can use a docker container which contains a compiled version of the project, add the wiki page to that and have it run tests.

### Running using Maven
Have the build server checkout the project and execute `mvn clean test-compile failsafe:integration-test`. Append `failsafe:verify` to the command if you want the build to fail in case of test failures.
The result in JUnit XML results can be found in: `target/failsafe-reports` (most build servers will pick these up automatically)
The HTML results can be found in: `target/fitnesse-results/index.html`

The FitNesse suite to run can be specified by changing the value of the `@Suite` annotation in `nl.hsac.fitnesse.fixture.FixtureDebugTest`, or (preferably) by adding a system property, called `fitnesseSuiteToRun`, specifying the suite to run to the build server's mvn execution.
By using the `@SuiteFilter` and `@ExcludeSuiteFilter` annotations, or (preferably) by adding `suiteFilter` and/or `excludeSuiteFilter` system properties, one can provide tags to in- or exclude and further filter the tests to run within the specified suite. Provide multiple tags by comma-separating them.

When multiple test runs are done (for instance in parallel to reduce total execution times) one should configure the the HTML output directory for each run to be a subdirectory of a common root (e.g. `target/fitnesse-results/run1` and `target/fitnesse-results/run2`) and then use `nl.hsac.fitnesse.junit.reportmerge.HtmlReportIndexGenerator` to generate a combined overview of all runs.

The Selenium configuration (e.g. what browser on what platform) to use when testing websites can be overridden by using system properties (i.e. `seleniumGridUrl` and either `seleniumBrowser` or `seleniumCapabilities`).
This allows different configurations on the build server to test with different browsers, without requiring different Wiki content, but only requiring a different build configuration.

### Running using Docker
Using docker one can start a container (i.e. virtual machine) containing a compiled version of the project and have it execute the tests defined. The main advantage of this approach is there is no need to gather dependencies and compile on each test run. This can be quite advantages when running builds on fully virtualized infrastructure where no state is maintained between runs, and all dependencies have to be downloaded afresh each run.

The way the actual tests are run is quite similar inside a docker container to what was described above for 'Running using Maven' in that a jUnit test is executed to run a suite of wiki pages.
Which suite is executed, filters to include and exclude, and Selenium can be configured without modifing the images provided.

Two images are provided on Docker Hub to run tests (or to be used as base for custom images, of course):
 * [hsac/fitnesse-fixtures-test-jre8](https://hub.docker.com/r/hsac/fitnesse-fixtures-test-jre8/) which contains a Java runtime and this project, which can be used to run tests that either do not need Selenium (i.e. do not test using BrowserTest) or use an existing/external Selenium Grid.
 * [hsac/fitnesse-fixtures-test-jre8-chrome](https://hub.docker.com/r/hsac/fitnesse-fixtures-test-jre8-chrome/) which adds this project to a Selenium standalone Chrome image and can be used to run tests with a Chrome browser (without the need to have/maintain a Selenium Grid).

An additional image is provided to combine multiple test run's results: [hsac/fitnesse-fixtures-combine](https://hub.docker.com/r/hsac/fitnesse-fixtures-combine).

Detailed instructions on how to use the images are provided in each image's description.

The automated build process also generates docker images for each commit in [GitLab's docker registry](https://gitlab.com/hsac-fitnesse/hsac-fitnesse-fixtures/container_registry), so if you want the latest code snapshot in an image you can get that there.
The master branch updates the 'latest' images, other branches have a tag based on their name. *Please note:* these images are temporary, and subject to periodic cleanup.

### Reports
The automated build process of this project generates some reports which can give an impression of the reports generated from test runs. The project's [acceptance tests](https://gitlab.com/hsac-fitnesse/hsac-fitnesse-fixtures/-/jobs/artifacts/master/file/test-results/index.html?job=reports-combine) and [examples](https://gitlab.com/hsac-fitnesse/hsac-fitnesse-fixtures/-/jobs/artifacts/master/file/example-results/index.html?job=example-reports-combine) both cover multiple runs in a single report. Clicking on a run's name in the 'Overview Pages' table takes you to a single run's test report.

## Fixture developer installation:
Import this project in your favorite Java IDE (with Maven support).

To start FitNesse: have the IDE execute `mvn compile dependency:copy-dependencies exec:exec`. The port used by FitNesse can be controlled by changing the `fitnesse.port` property's value in pom.xml.
FitNesse will be available at `http://localhost:<fitnesse.port>/`, example usage of the symbols and fixtures can be seen in `http://localhost:<fitnesse.port>/HsacExamples`.

To debug a fixture used in a FitNesse page: change the `@Suite` annotation's value in `nl.hsac.fitnesse.fixture.FixtureDebugTest` to contain the page name, then just debug this test.

## Documentation
More information about this project can be found on its [GitHub Wiki](https://github.com/fhoeben/hsac-fitnesse-fixtures/wiki)
