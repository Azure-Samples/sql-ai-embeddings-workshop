# 2. Call REST services from the Azure SQL Database with External REST Endpoint Invocation

## External REST Endpoint Invocation

Azure SQL Database External REST Endpoint Invocation provides the ability to call REST endpoints from other Azure services such as OpenAI, Azure Event Hub, Azure Functions, PowerBI and more. Common use cases for developers to use External REST Endpoint Invocation are:

* Utilize Azure OpenAI services such as chat, embeddings, language, and content safety
* Ability to push business logic out of the database and into Azure Functions
* Pull/push data to/from external sources (including Azure Blob Storage) for ETL or derived data stores
* Participate in event-based architectures with Azure Event Hub or Kafka

External REST Endpoint Invocation can be called in an Azure SQL Database using the sp_invoke_external_rest_endpoint stored procedure.

## Getting started with REST in the Azure SQL Database

In this first section, you will test the External REST Endpoint Invocation (EREI) feature of the database to ensure you have connectivity to other Azure services by asking ChatGPT for a joke. This section will also create a database scoped credential. A database scoped credential is a record in the database that contains authentication information for connecting to a resource outside the database. For this lab, we will be creating one that contains the api key for connecting to Azure OpenAI services.

### Using T-SQL to check connectivity to Azure OpenAI and creating database scoped credentials

1. Using the new query sheet in VS Code, copy and paste the following code:

    ```SQL

    -- Create a master key for the database
    if not exists(select * from sys.symmetric_keys where [name] = '##MS_DatabaseMasterKey##')
    begin
        create master key encryption by password = N'V3RYStr0NGP@ssw0rd!';
    end
    go

    -- Create the database scoped credential for Azure OpenAI
    if not exists(select * from sys.database_scoped_credentials where [name] = 'https://igniteai@lab.LabInstance.Id.openai.azure.com/')
    begin
        create database scoped credential [https://igniteai@lab.LabInstance.Id.openai.azure.com/]
        with identity = 'HTTPEndpointHeaders', secret = '{"api-key":"@lab.Variable(aiKey)"}';
    end
    go
    ```

1. Then click the run button on the query sheet

    ![A picture of clicking the run button on the query sheet](./media/Screenshot%202024-10-24%20at%209.18.36 AM.png)

    The master key will be set, and the database scoped credential will be created.

1. Back in the query sheet, remove the previous code by highlighting it and pressing delete/backspace.

1. Let's test the connectivity between Azure OpenAI and the database and see the ability to call external REST endpoints in action. Copy and paste the following code into a blank query editor in VS Code:

    ```SQL
    declare @url nvarchar(4000) = N'https://igniteai@lab.LabInstance.Id.openai.azure.com/openai/deployments/gpt-4/chat/completions?api-version=2024-06-01';
    declare @payload nvarchar(max) = N'{"messages":[{"role":"system","content":"You are an expert joke teller."},                                   
                                       {"role":"system","content":"tell me a joke about a llama walking into a bar"}]}'
    declare @ret int, @response nvarchar(max);

    exec @ret = sp_invoke_external_rest_endpoint
        @url = @url,
        @method = 'POST', 
        @payload = @payload,
        @credential = [https://igniteai@lab.LabInstance.Id.openai.azure.com/],    
        @timeout = 230,
        @response = @response output;
        
    select @ret as ReturnCode, json_value(@response, '$.result.choices[0].message.content') as "Joke";
    ```

1. Click the run button on the query sheet. The result will be an amazing joke you can tell your friends and family!

    ![A picture of running the T-SQL to create a llama bar joke](./media/Screenshot%202024-10-25%20at%206.44.56 AM.png)

    **Question:** tell me a joke about a llama walking into a bar.
    
    **Response:** A llama walks into a bar and the bartender says, "Hey, we have a drink named after you!" The llama looks surprised and asks, "You have a drink named Larry?"

    Classic Larry.