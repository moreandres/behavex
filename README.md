# BehaveX

BehaveX is a test wrapper on top of Python Behave, that provides some additional capabilites over the Behave framework that are useful in testing pipelines.

BehaveX works over Behave based projects. 

Basically, this wrapper encapsulates the Behave framework functionality and includes the following features:
* Perform parallel test executions (multi-process executions)
  * By feature
  * By scenario
* Get additional test execution reports
  * Friendly HTML report
  * JSON report (enable exporting|integrating test execution information)
* Provide additional evidence as part of execution reports
  * Any testing evidence you get, you can paste it to a predefined folder path (by scenario) to be part of the HTML report
* Generate test logs per scenario
  * Whatever you log in test steps using the logging library, it will generate an individual log report for each scenario
* Mute test scenarios in build servers
  * By just adding the @MUTE tag to test scenarios, they will be executed, but they will not be part of the JUnit reports
* Generate metrics in HTML report for the executed test suite
  * Automation Rate, Pass Rate and Steps executions & duration
* Execute dry runs and see the full list of scenarios into the HTML report
  * This is an override of the Behave dry run implementation

![test execution report](https://github.com/hrcorval/behavex/blob/master/img/html_test_report.png?raw=true)

![test execution report](https://github.com/hrcorval/behavex/blob/master/img/html_test_report_2.png?raw=true)

![test execution report](https://github.com/hrcorval/behavex/blob/master/img/html_test_report_3.png?raw=true)


## Installing BehaveX

Execute the following command to install BehaveX with pip:

> pip install behavex

## Executing BehaveX

The execution is performed in the same way as you do when executing the behave from command line, but using the "behavex" command.

Examples:

> behavex -t @TAG_1  -t ~@TAG_2

> behavex -t @TAG_1,@TAG_2

> behavex -t @TAG --parallel-processes 4 --parallel-scheme scenario

> behavex -t @TAG --parallel-processes 3

> behavex -t @TAG --dry-run


## Constraints

* BehaveX is currently implemented over Behave **v1.2.6**, and not all Behave arguments are yet supported.
* Parallel execution implementation is based on concurrent Behave processes. So, whatever you have into the **before_all** and **after_all** hooks in **environment.py** module, it will be re-executed on every parallel process. Also, the same will happen with the **before_feature** and **after_feature** hooks when the parallel execution schema is set by scenario.
* The library is provided as is, and no tests over the framework have been implemented yet (there were tests in initial versions but they got deprecated). Any contribution on that end will help a lot on delivering with confidence new library versions.
* Some english translations might not be accurate (and docstrings are empty) so it is expected this is fixed soon.

### Additional Comments

* The stop argument does not work when performing parallel test executions.
* The JUnit reports have been replaced by the ones generated by the test wrapper, just to support muting tests

## Supported Behave arguments
The following Behave arguments are currently supported:
* no_color
* color
* define
* exclude
* include
* no_snippets
* no_capture
* name
* capture
* no_capture_stderr
* capture_stderr
* no_logcapture
* logcapture
* logging_level
* summary
* quiet
* stop
* tags
* tags-help

There might be more arguments that can be supported, it is just a matter of adapting the wrapper implementation to use these.

## Additional BehaveX arguments
* output_folder
  * Specifies the output folder for all execution reports (JUnit, HTML and JSon)
* dry-run
  * Overwrites the Behave dry-run implementation
  * Performs a dry-run by listing the scenarios as part of the output reports
* parallel_processes
  * Specifies the number of parallel Behave processes
* parallel_scheme
  * Performs the parallel test execution by [scenario|feature]

## Parallel test executions
The implementation for running tests in parallel is based on concurrent Behave instances executed in multiple processes.

As mentioned as part of the wrapper constraints, this approach implies that whatever you have in the Python Behave hooks in **environment.py** module, it will be re-executed on every parallel process.

BehaveX will be in charge of managing each parallel process, and consolidate all the information into the execution reports

Parallel test executions can be performed by **feature** or by **scenario**.

Examples:
> behavex -t @\<TAG\> --parallel-processes 2 --parallel-schema scenario

> behavex -t @\<TAG\> --parallel-processes 5 --parallel-schema feature

When the parallel-schema is set by **feature**, all tests within each feature will be run sequentially.

## Test execution reports
### HTML report
This is a friendly test execution report that contains information related to test scenarios, execution status, execution evidence and metrics. A filters bar is also provided to filter scenarios by name, tag or status.

It should be available at the following path:
> <output_folfer>/report.html


### JSON report
Contains information about test scenarios and execution status.

It should be available at the following path:
> <output_folfer>/report.json

The report is provided to enable exporting test execution data and to simplify the integration with third party tools.

### JUnit report

The wrapper overwrites the Behave JUnit reports, just to enable dealing with parallel executions and muted test scenarios

By default, there will be one JUnit file per feature, unless the parallel execution is performed by scenario, in which there will be one JUnit file per scenario.

Reports are available at the following path:
> <output_folfer>/behave/*.xml

## Attaching additional execution evidence to test report

It is considered a good practice to provide as much as evidence as possible in test executions reports to properly identify the root cause of issues.

Any evidence file you generate when executing a test scenario, it can be stored into a folder path that the wrapper provides for each scenario.

The evidence folter path is stored into the "evidence_path" context variable (or "context.evidence_path"). This variable is updated by the wrapper before executing each scenario, and all the files you copy into that path will be accesible from the HTML report linked to the executed scenario

## Test logs per scenario

The HTML report provides test execution logs per scenario. Everything that is being logged using the **logging** library will be stored into a test execution log file linked to the test scenario.

## Metrics

There are a few metrics that can be easily calculated for the executed suite:
* Automation Rate
* Pass Rate
* Steps execution counter and average execution time

All metrics are provided as part of the HTML report

## Dry runs

The wrapper overwrites the Behave dry run implementation just to be able to provide the outputs into the wrapper reports.

The HTML report generated as part of the dry run can be used to share the scenarios specifications with any stakeholder.

Example:
> behavex -t @TAG --dry-run

## Muting test scenarios

Sometimes it is necessary to have failing test scenarios to continue being executed in all build server plans, but having them muted until the test or product fix is provided.

Tests are muted by adding the @MUTE tag to each test scenario. Muted scenarios will be run but the execution will not be notified in the JUnit reports. However, you will see the execution information in the HTML report.