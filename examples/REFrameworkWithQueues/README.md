## Basic REFramework with queues

A simple example that shows how to use REFramework with Orchestrator and Queues

The example also demonstrate 
- how to easily create a report with statuses for each transaction
- how ApplicationException and BusinessRuleException are treated differently. Note that after an ApplicationException is thrown the REFramework closes all applications and restarts InitAllApplications

Prerequisite:
First, run the workflow PrepareData.xaml. This will create a new queue REFrameworkQueue and add three items. In order to run this flow make sure that the robot has rights to add, edit and delete queues. Alternatively you can create manualy a queue REFrameworkQueue and add three items each one having at least the following informations: Name, From and Status.
