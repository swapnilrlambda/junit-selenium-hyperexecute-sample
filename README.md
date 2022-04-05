<img height="100" alt="hyperexecute_logo" src="https://user-images.githubusercontent.com/1688653/159473714-384e60ba-d830-435e-a33f-730df3c3ebc6.png">

HyperExecute is a smart test orchestration platform to run end-to-end Selenium tests at the fastest speed possible. HyperExecute lets you achieve an accelerated time to market by providing a test infrastructure that offers optimal speed, test orchestration, and detailed execution logs.

The overall experience helps teams test code and fix issues at a much faster pace. HyperExecute is configured using a YAML file. Instead of moving the Hub close to you, HyperExecute brings the test scripts close to the Hub!

* <b>HyperExecute HomePage</b>: https://www.lambdatest.com/hyperexecute
* <b>Lambdatest HomePage</b>: https://www.lambdatest.com
* <b>LambdaTest Support</b>: [support@lambdatest.com](mailto:support@lambdatest.com)

To know more about how HyperExecute does intelligent Test Orchestration, do check out [HyperExecute Getting Started Guide](https://www.lambdatest.com/support/docs/getting-started-with-hyperexecute/)

# How to run Selenium automation tests on HyperExecute (using JUnit framework)

* [Pre-requisites](#pre-requisites)
   - [Download Concierge](#download-concierge)
   - [Configure Environment Variables](#configure-environment-variables)

* [Matrix Execution with JUnit](#matrix-execution-with-junit)
   - [Core](#core)
   - [Pre Steps and Dependency Caching](#pre-steps-and-dependency-caching)
   - [Post Steps](#post-steps)
   - [Artifacts Management](#artifacts-management)
   - [Test Execution](#test-execution)

* [Auto-Split Execution with JUnit](#auto-split-execution-with-junit)
   - [Core](#core-1)
   - [Pre Steps and Dependency Caching](#pre-steps-and-dependency-caching-1)
   - [Post Steps](#post-steps-1)
   - [Artifacts Management](#artifacts-management-1)
   - [Test Execution](#test-execution-1)

* [Secrets Management](#secrets-management)
* [Navigation in Automation Dashboard](#navigation-in-automation-dashboard)

# Pre-requisites

Before using HyperExecute, you have to download Concierge CLI corresponding to the host OS. Along with it, you also need to export the environment variables *LT_USERNAME* and *LT_ACCESS_KEY* that are available in the [LambdaTest Profile](https://accounts.lambdatest.com/detail/profile) page.

## Download Concierge

Concierge is a CLI for interacting and running the tests on the HyperExecute Grid. Concierge provides a host of other useful features that accelerate test execution. In order to trigger tests using Concierge, you need to download the Concierge binary corresponding to the platform (or OS) from where the tests are triggered:

Also, it is recommended to download the binary in the project's parent directory. Shown below is the location from where you can download the Concierge binary:

* Mac: https://downloads.lambdatest.com/concierge/darwin/concierge
* Linux: https://downloads.lambdatest.com/concierge/linux/concierge
* Windows: https://downloads.lambdatest.com/concierge/windows/concierge.exe

## Configure Environment Variables

Before the tests are run, please set the environment variables LT_USERNAME & LT_ACCESS_KEY from the terminal. The account details are available on your [LambdaTest Profile](https://accounts.lambdatest.com/detail/profile) page.

For macOS:

```bash
export LT_USERNAME=LT_USERNAME
export LT_ACCESS_KEY=LT_ACCESS_KEY
```

For Linux:

```bash
export LT_USERNAME=LT_USERNAME
export LT_ACCESS_KEY=LT_ACCESS_KEY
```

For Windows:

```bash
set LT_USERNAME=LT_USERNAME
set LT_ACCESS_KEY=LT_ACCESS_KEY
```

# Matrix Execution with JUnit

Matrix-based test execution is used for running the same tests across different test (or input) combinations. The Matrix directive in HyperExecute YAML file is a *key:value* pair where value is an array of strings.

Also, the *key:value* pairs are opaque strings for HyperExecute. For more information about matrix multiplexing, check out the [Matrix Getting Started Guide](https://www.lambdatest.com/support/docs/getting-started-with-hyperexecute/#matrix-based-build-multiplexing)

### Core

In the current example, matrix YAML file (*yaml/junit_hyperexecute_matrix_sample.yaml*) in the repo contains the following configuration:

```yaml
globalTimeout: 150
testSuiteTimeout: 150
testSuiteStep: 150
```

Global timeout, testSuite timeout, and testSuite timeout are set to 150 minutes.
 
The target platform is set to Win. Please set the *[runson]* key to *[mac]* if the tests have to be executed on the macOS platform.

```yaml
runson: win
```

The *matrix* constitutes of the following entries - *classname*. The entries represent the class names in the test code.

```yaml
matrix:
  classname: ["ToDoTest", "SelPlayGroundTest1", "SelPlayGroundTest2" ]
```

The *testSuites* object contains a list of commands (that can be presented in an array). In the current YAML file, commands for executing the tests are put in an array (with a '-' preceding each item). The Maven command *mvn test* is used to run tests located in the current project. In the current project, parallel execution is achieved at the *class* level. The *maven.repo.local* parameter in Maven is used for overriding the location where the dependent Maven packages are downloaded.

```yaml
testSuites:
  - mvn `-Dmaven.repo.local=$CACHE_DIR `-Dtest=$classname test site surefire-report:report
```

### Pre Steps and Dependency Caching

Dependency caching is enabled in the YAML file to ensure that the package dependencies are not downloaded in subsequent runs. The first step is to set the Key used to cache directories. The directory *m2_cache_dir* is created in the project's root directory.

```yaml
env:
  CACHE_DIR: ~/m2_cache_dir

cacheKey: '{{ checksum "pom.xml" }}'
cacheDirectories:
  - $CACHE_DIR
```

Steps (or commands) that must run before the test execution are listed in the *pre* run step. In the example, the Maven packages are downloaded in the *m2_cache_dir*. To prevent test execution at the *pre* stage, the *maven.test.skip* parameter is set to *true* so that only packages are downloaded and no test execution is performed.

```yaml
pre:
  - mkdir m2_cache_dir
  - mvn -Dmaven.repo.local=$CACHE_DIR -Dmaven.test.skip=true clean install
```

### Post Steps

Steps (or commands) that need to run after the test execution are listed in the *post* step. In the example, we *cat* the contents of *yaml/junit_hyperexecute_matrix_sample.yaml*

```yaml
post:
  - cat yaml/junit_hyperexecute_matrix_sample.yaml
```

### Artifacts Management

The *mergeArtifacts* directive (which is by default *false*) is set to *true* for merging the artifacts and combing artifacts generated under each task.

The *uploadArtefacts* directive informs HyperExecute to upload artifacts [files, reports, etc.] generated after task completion. In the example, *path* consists of a regex for parsing the site and sure-fire reports (i.e. *target/site/* and *target/surefire-reports/*) directory.

```yaml
mergeArtifacts: true

uploadArtefacts:
 - name: Final Report
   path:
    - target/site/**
 - name: Surefire Report
   path:
    - target/surefire-reports/**
```

HyperExecute also facilitates the provision to download the artifacts on your local machine. To download the artifacts, click on Artifacts button corresponding to the associated TestID.

<img width="1425" alt="junit_matrix_artefacts_1" src="https://user-images.githubusercontent.com/1688653/160455415-d145dd30-8521-4e5b-8c80-0fcbd730b506.png">

Now, you can download the artifacts by clicking on the Download button as shown below:

<img width="1425" alt="junit_matrix_artefacts_2" src="https://user-images.githubusercontent.com/1688653/160455432-05c9eb1e-f337-4350-af23-d020a2f1a9a5.png">

## Test Execution

The CLI option *--config* is used for providing the custom HyperExecute YAML file (i.e. *yaml/junit_hyperexecute_matrix_sample.yaml*). Run the following command on the terminal to trigger the tests in C# files on the HyperExecute grid. The *--download-artifacts* option is used to inform HyperExecute to download the artifacts for the job. The *--force-clean-artifacts* option force cleans any existing artifacts for the project.

```bash
./concierge --config yaml/junit_hyperexecute_matrix_sample.yaml --force-clean-artifacts --download-artifacts
```

Visit [HyperExecute Automation Dashboard](https://automation.lambdatest.com/hyperexecute) to check the status of execution:

<img width="1414" alt="junit_matrix_execution" src="https://user-images.githubusercontent.com/1688653/160455415-d145dd30-8521-4e5b-8c80-0fcbd730b506.png">

Shown below is the execution screenshot when the YAML file is triggered from the terminal:

<img width="1413" alt="junit_cli1_execution" src="https://user-images.githubusercontent.com/1688653/159760380-7d633a1f-dacc-4851-8142-95dbf25cef8a.png">

<img width="1101" alt="junit_cli2_execution" src="https://user-images.githubusercontent.com/1688653/159760390-4429ff96-7056-4318-911a-972d4570185f.png">

## Auto-Split Execution with JUnit

Auto-split execution mechanism lets you run tests at predefined concurrency and distribute the tests over the available infrastructure. Concurrency can be achieved at different levels - file, module, test suite, test, scenario, etc.

For more information about auto-split execution, check out the [Auto-Split Getting Started Guide](https://www.lambdatest.com/support/docs/getting-started-with-hyperexecute/#smart-auto-test-splitting)

### Core

Auto-split YAML file (*yaml/junit_hyperexecute_autosplit_sample.yaml*) in the repo contains the following configuration:

```yaml
globalTimeout: 150
testSuiteTimeout: 150
testSuiteStep: 150
```

Global timeout, testSuite timeout, and testSuite timeout are set to 150 minutes.
 
The *runson* key determines the platform (or operating system) on which the tests are executed. Here we have set the target OS as Windows.

```yaml
runson: win
```

Auto-split is set to true in the YAML file.

```yaml
 autosplit: true
```

*retryOnFailure* is set to true, instructing HyperExecute to retry failed command(s). The retry operation is carried out till the number of retries mentioned in *maxRetries* are exhausted or the command execution results in a *Pass*. In addition, the concurrency (i.e. number of parallel sessions) is set to 4.

```yaml
retryOnFailure: true
maxRetries: 5
concurrency: 4
```

### Pre Steps and Dependency Caching

Dependency caching is enabled in the YAML file to ensure that the package dependencies are not downloaded in subsequent runs. The first step is to set the Key used to cache directories. The directory *m2_cache_dir* is created in the project's root directory.

```yaml
env:
  CACHE_DIR: m2_cache_dir

# Dependency caching for Windows
cacheKey: '{{ checksum "pom.xml" }}'
cacheDirectories:
  - $CACHE_DIR
```

Steps (or commands) that must run before the test execution are listed in the *pre* run step. In the example, the Maven packages are downloaded in the *m2_cache_dir*. To prevent test execution at the *pre* stage, the *maven.test.skip* parameter is set to *true* so that only packages are downloaded and no test execution is performed.

```yaml
shell: bash

pre:
  - mkdir m2_cache_dir
  - mvn -Dmaven.repo.local=$CACHE_DIR -Dmaven.test.skip=true clean install
```

### Post Steps

Steps (or commands) that need to run after the test execution are listed in the *post* step. In the example, we *cat* the contents of *yaml/junit_hyperexecute_autosplit_sample.yaml*

```yaml
post:
  - cat yaml/junit_hyperexecute_autosplit_sample.yaml
```

The *testDiscovery* directive contains the command that gives details of the mode of execution, along with detailing the command that is used for test execution. Here, we are fetching the list of class names that would be further passed in the *testRunnerCommand*

```yaml
testDiscovery:
  type: raw
  mode: static
  command: grep 'public class' src/test/java/hyperexecute/*.java | awk '{print$3}'
```

Running the above command on the terminal will give a list of scenarios present in the *feature* files:

* SelPlayGroundTest1
* SelPlayGroundTest2
* ToDoTest

The *testRunnerCommand* contains the command that is used for triggering the test. The output fetched from the *testDiscoverer* command acts as an input to the *testRunner* command.

```yaml
testRunnerCommand: mvn `-Dmaven.repo.local=$CACHE_DIR `-Dtest=$test test site surefire-report:report
```

### Artifacts Management

The *mergeArtifacts* directive (which is by default *false*) is set to *true* for merging the artifacts and combing artifacts generated under each task.

The *uploadArtefacts* directive informs HyperExecute to upload artifacts [files, reports, etc.] generated after task completion. In the example, *path* consists of a regex for parsing the site and sure-fire reports (i.e. *target/site/* and *target/surefire-reports/*) directory.

```yaml
mergeArtifacts: true

uploadArtefacts:
 - name: Final Report
   path:
    - target/site/**
 - name: Surefire Report
   path:
    - target/surefire-reports/**
```

HyperExecute also facilitates the provision to download the artifacts on your local machine. To download the artifacts, click on Artifacts button corresponding to the associated TestID.

<img width="1425" alt="junit_autosplit_artefacts_1" src="https://user-images.githubusercontent.com/1688653/160455437-7871ba51-e46c-4bb0-8d34-48097ec427f9.png">

Now, you can download the artifacts by clicking on the Download button as shown below:

<img width="1425" alt="junit_autosplit_artefacts_2" src="https://user-images.githubusercontent.com/1688653/160455442-a2335e56-323c-46df-b75a-a7f21937719f.png">

### Test Execution

The CLI option *--config* is used for providing the custom HyperExecute YAML file (i.e. *yaml/junit_hyperexecute_autosplit_sample.yaml*). Run the following command on the terminal to trigger the tests in C# files on the HyperExecute grid. The *--download-artifacts* option is used to inform HyperExecute to download the artifacts for the job. The *--force-clean-artifacts* option force cleans any existing artifacts for the project.

```bash
./concierge --config yaml/junit_hyperexecute_autosplit_sample.yaml --force-clean-artifacts --download-artifacts
```

Visit [HyperExecute Automation Dashboard](https://automation.lambdatest.com/hyperexecute) to check the status of execution

<img width="1414" alt="junit_autosplit_execution" src="https://user-images.githubusercontent.com/1688653/160455437-7871ba51-e46c-4bb0-8d34-48097ec427f9.png">

Shown below is the execution screenshot when the YAML file is triggered from the terminal:

<img width="1412" alt="junit_autosplit_cli1_execution" src="https://user-images.githubusercontent.com/1688653/159760928-d0816808-e25f-45aa-bacb-4e17312ad4e3.png">

<img width="1408" alt="junit_autosplit_cli2_execution" src="https://user-images.githubusercontent.com/1688653/159760937-5cf297de-5182-469b-9a88-bec78383da1d.png">

## Secrets Management

In case you want to use any secret keys in the YAML file, the same can be set by clicking on the *Secrets* button the dashboard.

<img width="703" alt="junit_secrets_key_1" src="https://user-images.githubusercontent.com/1688653/152540968-90e4e8bc-3eb4-4259-856b-5e513cbd19b5.png">

Now create a *secret* key that you can use in the HyperExecute YAML file.

<img width="359" alt="junit_management_1" src="https://user-images.githubusercontent.com/1688653/153250877-e58445d1-2735-409a-970d-14253991c69e.png">

All you need to do is create an environment variable that uses the secret key:

```yaml
env:
  PAT: ${{ .secrets.testKey }}
```

## Navigation in Automation Dashboard

HyperExecute lets you navigate from/to *Test Logs* in Automation Dashboard from/to *HyperExecute Logs*. You also get relevant get relevant Selenium test details like video, network log, commands, Exceptions & more in the Dashboard. Effortlessly navigate from the automation dashboard to HyperExecute logs (and vice-versa) to get more details of the test execution.

Shown below is the HyperExecute Automation dashboard which also lists the tests that were executed as a part of the test suite:

<img width="1429" alt="junit_hyperexecute_automation_dashboard" src="https://user-images.githubusercontent.com/1688653/160455415-d145dd30-8521-4e5b-8c80-0fcbd730b506.png">

Here is a screenshot that lists the automation test that was executed on the HyperExecute grid:

<img width="1429" alt="junit_testing_automation_dashboard" src="https://user-images.githubusercontent.com/1688653/159760375-7f1e3e71-6ea9-449c-8a8d-c4d02c40f7c2.png">

## We are here to help you :)
* LambdaTest Support: [support@lambdatest.com](mailto:support@lambdatest.com)
* HyperExecute HomePage: https://www.lambdatest.com/support/docs/getting-started-with-hyperexecute/
* Lambdatest HomePage: https://www.lambdatest.com