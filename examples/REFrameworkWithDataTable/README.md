This project is a simple example that shows how to use the activity when no Orchestrator/Queues are involved.

The robot reads an Excel that has two columns (containing numbers) and tries to calculate the difference between the two columns. 
If the processing is succesful, the result is added to a third column. If any errors occur, a remark is added to the third column

This example also shows 
- how to used Json configuration file
- how to use ExtraConfigurations feature
- how to pass variables between different workflows (see ProcessedReport created in InitTransactionData and used in EndProcess)
- how to use HandleTransactionStatus.xaml workflow
