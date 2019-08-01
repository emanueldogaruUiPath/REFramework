## Adanced REFramework with queues

A more advanced example that shows how to use REFramework with Queues that can go to several statuses

The example aim to demonstrate 
- how to use Queue Item Statuses feature
- how to easily create a report with statuses for each transaction
- how ApplicationException and BusinessRuleException are treated differently. Note that after an ApplicationException is thrown the REFramework closes all applications and restarts InitAllApplications


Prerequisite: run PrepareData.xaml. This will create a new queue REFrameworkQueue and add three items. In order to run this flow make sure that the robot has rights to add, edit and delete queues. Alternatively you can create manualy a queue REFrameworkQueue and add three items each one having at least the following informations: Name, From and Status=New.

After running the framework take a look at the report to see the order the items have been processed and how the exceptions have been handled.
For example in case of item Ref2 where no exception occured, the transaction have been processed in all the states. In case of item Ref1 during the status WaitingForData we encounter a BusinessRuleException. In that case the item is marked an failed and the framework will not process the next status.
Finally, in case of Ref3 status FinalProcessing at the first run we encounter an ApplicationError. The item is retried and the second time everything is succesfully processed.
