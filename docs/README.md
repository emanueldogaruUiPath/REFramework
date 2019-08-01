# REFramework as Custom Activity

## Introduction

The Reframework Custom Activity has the same functionality as the classic REFramework. Simply put, the goal of the
REFramework is to facilitate the development of complex automations by providing the most common
functionalities out of box, such as:
- Exception Handling
- Retry Mechanism
- Configuration Mechanism
- Logging

## How does it work

The REFramework is very well suited for iterative processes, where the data can be splitted into several data. That is reason the REFrameworl is built around TransactionData (the entire data to be processed) and TransactionItem (chunk of data to be processed during one iteration).

As seen in the diagram https://github.com/emanueldogaruUiPath/REFramework/blob/master/docs/Architecture%20REFramework.png the flow of the activity is the following:
1) the config file and the extra configurations are retrieved. From this point on every workflow can define an
argument in_Config and use said configurations
2) the in/io arguments are retrieved. From this point on every workflow can access these data by defining an
argument with the same name
3) the framework invokes CloseAllApplications. In case of exception KillAllProcesses is invoked
4) the framework invokes InitTransactionData. From this point on every following workflow can define and use an
argument in_TransactionData. In case of exceptions the workflow is retried a number
of MaxInitRetryNumber times. If the error persists a variable in_GeneralException is initialized with the error
type. Go to step 10.
5) the framework invokes InitAllApplications. In case of exceptions the workflow is retried a number
of MaxInitRetryNumber times. I the error persists a variable in_GeneralException is initialized with the error
type. Go to step 9
6) the framework invokes GetTransactionItem that will initialize the
variable TransactionItem. If TransactionItem is nothing than go to step 9. If an error occurs the
variable in_GeneralException is initialized. Go to step 9. If TransactionItem is not nothing go to step 7.
7) the framework invokes ProcessTransaction. in case of success TransactionNumber is incremented and
framework advances to step 8.
a) If TransactionItem is of type Queue, TransactionNumber is incremented and framework advances to step
8.
b) If TransactionItem is not of type Queue and an application exception occurs and MaxRetryNumber is not
reached, framework invokes CloseAllApplications and goes to step 5.
c) If TransactionItem is not of type Queue and an application exception occurs and MaxRetryNumber is
reached, init variable in_TransactionApplicationError and go to step 8.
d) If a Business Exception occurs, init variable TransactionBussinesError and go to step 8
8) the framework invokes HandleTransactionStatus. This workflow has access
to in_TransactionBussinesError and in_TransactionApplicationError whose values depend of the status of the
processing step 7.
9) the framework invokes EndProcess. This workflow has access to in_GeneralException which was set during an
unrecoverable application exception
10) the framework invokes CloseAllApplications. In case of exception KillAllProcesses is invoked.
11) arguments out/io are updated

Additional to the above flow, one must consider also that the workflow HandleEachException is called every time an
exception occurs in any workflow. This facilitates exception handling at any level.

## How to use it 

The package contains only the REFramework activity which should be used as following:
- Add the activity to the workflow
- If needed, expand Utilities pane from the bottom of the activity and select TransactionData type
and TransactionItem type. Press Generate Samples button to generate xaml samples files. The samples will
be generated in the folder named Framework only if the samples do not already exist.
The following workflows are mandatory:
- GetTransactionItem - with mandatory arguments:
- - in_TransactionNumber (Int32)
- - io_TransactionData
- - out_TransactionItem
- ProcessTransaction - with mandatory argument:
- - in_TransactionItem
The other workflows are optional:
- InitTransactionData - which should provide at least the mandatory argument: out_TransactionData
- InitAllApplications
- HandleTransactionStatus - with mandatory arguments:
- - in_TransactionItem
- - in_TransactionApplicationError (System.Exception)
- - in_TransactionBussinesError (UiPath.Core.BussinesRuleException)
- EndProcess
- CloseAllApplications
- KillAllProcesses
- HandleEachException

### Workflow arguments considerations
Besides the mandatory arguments every workflow can use the following arguments
- in_Config : Dictionary<string, string>
- in_TransactionNumber : Int32 initialized at the beginning of the REFramework with 0 and incremented
automatically according to the associated diagramme.
- in_TransactionData
- in_GeneralException

Moreover, additional to the arguments predefined, one can create any number of custom variables that will flow
through the framework as following:
- first the variable must be defined in a workflow as an output argument (for example out_customArgument)
- once defined, every other workflow that follows can access the variable as in or io argument:
in_customArgument or io_customArgument. The only requirement is to keep the same name of the
variable.
One can exchange data outside the REFramework by using the Arguments window (see button Edit Arguments):
- the in/io arguments are accessible to any workflow. For example if we define an in argument named in_arg1
any workflow can access it by defining an input argumentwith the same name
- similarly, the output argument can be defined in any workflow during the process
Regarding the arguments, one must pay attention to keep consistency between the arguments' type.

## ProcessConfiguration

One can optionally use a config file to define modifiable parameters. Such a config file can be obtained by pressing
the button Generate Config which will create an Excel and a JSON file in the folder Data (attention if the config file already exists
it will not be overwritten). All the configurations can be accessed through the in_Config variable. Moreover, one can import the config file directly from an asset using the option Config Asset as JSON.

The Extra Configurations feature can be used to override the config file, i.e. a configuration from Extra Configuration
table has priority over the same configuration from the Excel file.
The config file is optional.
Another set of arguments is related to workflow timeouts. For each workflow, one can define the maximum
execution time. If the timeout is reached, an exception will be thrown. 

## Considerations regarding the Exception Handling

There are several ways to deal with exceptions:
1) First, the workflow HandleEachException will be invoked anytime an exception is thrown during any workflow.
This offers the possibility to log any errors even if it won’t affect the flow of the process. Let’s take as example
a situation where InitAllApplications fails the first time is invoked but succeeds the second time. The only way
to log the exception is to use treat it during HandleEachException.
2) Exceptions that are thrown during ProcessTransaction. They should be dealt within the
step HandleTransactionStatus
a) Business exceptions - in this case the current transaction is finished, and framework moves to process next
available transaction item
b) Applications exceptions - in this case ProcessTransaction is retried according to config file
3) Exceptions that are thrown in any other xaml file after the retry mechanism failed to recover. They are saved in
the global object GeneralException that can be accessed from the step EndProcess (and all others that follow)


## Using Queue Item Statuses feature

There are situations when the processing that has to be done to a single item queue become too complex to be treated individually into ProcessTransaction workflow. Imagine a document processing automation where for each document one has to digitize document, validate digitization, process document, etc. In this case, if the entire processing of a queue would be done in single iteration in ProcessTransaction and an error occured during one of the last steps, during the retry, all initials steps that have been succesfully  would have to be redone. This, of course, is not optimal and too time consuming. To address this issue, we implemented a feature named Queue Item Statuses that works as following:
- first we split the processing that has to be done to queue in several steps: eg. digitize, validate, process. This steps should be defined as an array of strings in option Queue Item Statuses.
- in ProcessTransaction we define actions that should be taken for each item at every step. We can differentiate the current step by using the input argument in_TransactionStatus
- once the REFramework starts, every item retrieved is processed in first associated step (eg. digitize). If the processing is succesful, REFramework will add a new queue item that has in its ItemInformation a next status that needs processing (eg. validate). This goes on until every queue item is processed at every step. 

For a practical example please take a look at the demo  https://github.com/emanueldogaruUiPath/REFramework/tree/master/examples/REFrameworkWithQueuesAdvanced
