# **Lab 3: Building and Securing a Safe Clinical Report Search API** 

In this lab, participants step into a real-world healthcare scenario
where a hospital needs a secure and intelligent way to search clinical
discharge summaries using natural language. Using Microsoft SQL Server
2025, SQL Server Management Studio Copilot, Azure OpenAI Service, and
Data API Builder, learners will build a secure semantic search API that
enables staff to retrieve relevant clinical reports while ensuring
patient data privacy through masking, RBAC, and managed identity. This
hands-on experience demonstrates how modern AI capabilities can be
integrated directly into SQL Server to create enterprise-ready, secure
healthcare solutions.

**Objectives:**

By the end of this lab, participants will be able to:

- Enable AI and vector search in Microsoft SQL Server 2025

- Generate embeddings using Azure OpenAI Service

- Implement semantic search with vector indexing

- Secure clinical data using masking, RBAC, and Managed Identity

- Expose search functionality as a REST API using Data API Builder

- Accelerate development with SQL Server Management Studio Copilot

## Exercise 1: Provision SQL Server on Azure VM

1.  Open browser enter +++https://portal.azure.com+++ and sign in with your
    Azure subscription account. Enter +++Azure SQL+++ in the search bar
    and select it.

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%203/media/image1.png)

2.  In the pane for **SQL Server on Azure Virtual Machines**,
    select **Show options**.

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%203/media/image2.png)

3.  In the **Select an image offer** box, choose a SQL Server image
    (such as **Free SQL Server License: SQL Server 2025 Enterprise
    Developer on Windows Server 2025**). Select **Create virtual machine**.

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%203/media/image3.png)

4.  On the **Basics** tab, provide the following information and then
    click **Next: Disk**

    - Subscription : @lab.CloudSubscription.Name

    - In the **Resource group** - @lab.CloudResourceGroup(ZAVA-Connect-RG).Name

    - Virtual machine name- +++azuresqlvm@lab.LabInstance.Id+++

    - **Region** - @lab.CloudResourceGroup(ZAVA-Connect-RG).Location

    - **Availability options** – *Availability zone*.

        1.  **Zone options** – Self-selected zone

        2.  **Availability zone** -Zone 1

    - In the **Image** list, select **Free SQL Server License: SQL Server
    2025 Enterprise Developer on Windows Server 2025** if it's not already
    selected.

    - Select **See all sizes** for the **Size** of the virtual machine and
    search for the **E4ds_v5** offering. This is one of the minimum
    recommended VM sizes for SQL Server on Azure VMs. be sure to clean up
    your resources once you're done with them to prevent any unexpected
    charges.

    Enter admin details as below:

    - Username : +++sqlvmuser+++
    - Password: +++AZvmsql12345+++

    - Under **Inbound port rules**, choose **Allow selected ports**, and
    then select **RDP (3389)** from the dropdown list.

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%203/media/image4.png)

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%203/media/image5.png)

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%203/media/image6.png)

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%203/media/image7.png)

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%203/media/image8.png)

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%203/media/image9.png)

5.  Keep default disk type values and click **Next: Networking**

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%203/media/image10.png)

6.  On **Management** tab:

    - Enable system assigned management identity

    - Enable periodic assessment

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%203/media/image11.png)
    
    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%203/media/image12.png)

7.  Navigate to **SQL Server settings** tab.

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%203/media/image13.png)

8.  On SQL server setting page, select below values and then click on
    **Review + create.**

    - **SQL connectivity**: Public(internet)

    - **SQL authentication**: Enable

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%203/media/image14.png)

9.  Once the validation is passed, click on **Create**.

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%203/media/image15.png)

10. Wait for the deployment to complete.

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%203/media/image16.png)

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%203/media/image17.png)

11. Copy the **Public IP address** to connect from SSMS in next task.

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%203/media/image18.png)

## Exercise 2: Create Azure OpenAI resource and deploy embedding models

1.  Switch back to Azure and search for +++Azure OpenAI+++ and select it.

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%203/media/image19.png)

2.  Click on **Create** and then select **Azure OpenAI** from the dropdown.

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%203/media/image20.png)

3.  Enter below values and click **Next**.

    Subscription : **@lab.CloudSubscription.Name**

    Resource Group – **@lab.CloudResourceGroup(ZAVA-Connect-RG).Name**

    Region – **@lab.CloudResourceGroup(ZAVA-Connect-RG).Location**

    Name : +++azsqlaoai@lab.LabInstance.Id**

    Pricing tier – Select **Standard S0**

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%203/media/image21.png)

4.  Keep the default value and click Next.

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%203/media/image22.png)

5.  Keep default tag and click Next.

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%203/media/image23.png)

6.  Review the details and click **Create**.

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%203/media/image24.png)

7.  Wait for the deployment successful and click on **Go to resource.**

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%203/media/image25.png)

8.  Expand **Resource management** and select **keys and endpoints** from left
    navigation menu and copy **endpoint and key** value to a notepad to
    use in next tasks.

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%203/media/image26.png)

9.  Click on **Overview** and select **Go to Foundry portal**

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%203/media/image27.png)

10. Click on **Deployments** under Shared resource from left navigation
    menu. Select **Deploy model-\> Deploy base model.**

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%203/media/image28.png)

11. Search for +++text-embedding+++, select **text-embedding-3-small**
    model and click **Confirm**.

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%203/media/image29.png)

12. Keep the default values and click **Customize**.

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%203/media/image30.png)

13. Set Tokens per Minute Rate limit to max and click **Deploy**.

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%203/media/image31.png)

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%203/media/image32.png)

## Exercise 3: Create Storage account and Store the file

1.  In the Azure portal, search for and Select Storage Accounts, and
    create a new **Storage Account** by entering the following:

    - **Name**: +++sa@lab.LabInstance.Id+++

    - **Subscription:** @lab.CloudSubscription.Name

    - **Resource Group**: @lab.CloudResourceGroup(ZAVA-Connect-RG).Name

    - **Region:** @lab.CloudResourceGroup(ZAVA-Connect-RG).Location

    - **Preferred storage account type**: Azure Blob Storage or Azure Data
    Lake Storage Gen2

    ![A screenshot of a computer Description automatically
    generated](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%203/media/image33.png)

2.  Click on **Create**.

    ![A screenshot of a computer Description automatically
    generated](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%203/media/image34.png)

3.  Wait for the deployment successful and select on **Go to resource**.

    ![A screenshot of a computer Description automatically
    generated](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%203/media/image35.png)

4.  Expand **Data storage from the left menu** and
    select **Containers**.

    ![A screenshot of a computer Description automatically
    generated](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%203/media/image36.png)

5.  Select **Add Container**.

    ![A screenshot of a computer Description automatically
    generated](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%203/media/image37.png)

6.  Enter container name as +++public+++ and click on **Create**.

    ![A screenshot of a computer Description automatically
    generated](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%203/media/image38.png)

7.  Select your **public** container.

    ![A screenshot of a computer Description automatically
    generated](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%203/media/image39.png)

8.  Click on **Upload**.

    ![A screenshot of a computer Description automatically
    generated](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%203/media/image40.png)

9.  Browse for files, and navigate to C:\LabFiles folder select
    the **clinical_reports.csv** file. Click on **Upload**.

    ![A screenshot of a computer Description automatically
    generated](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%203/media/image41.png)

10. Expand **Settings**, and select **Share acess tokens**.

    ![A screenshot of a computer Description automatically
    generated](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%203/media/image42.png)

11. Select **Generate SAS token and URL**, and copy the **Blob SAS
    token** to a notepad to use later in the lab.

    ![A screenshot of a computer Description automatically
    generated](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%203/media/image43.png)

    ![A screenshot of a computer Description automatically
    generated](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%203/media/image44.png)

## Exercise 4: Connect SQL server 2025 via SSMS 

1.  Doble click on SSMs form task bar and select **Sign in with Microsoft**

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%203/media/image45.png)

12. Enter below details and click **Continue**.

    - Server name : **Azure VM public ip address that you copied earlier,1433 (eg :8.234.343.54,1433)**

    - Authentication : **SQL Server Authentication**

    - Select **Trust Server certificate** checkbox

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%203/media/image50.png)

1.  In the top right, select the **user** icon and select sign in.

2.  Select Work or School account and click Continue.

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%203/media/image46.png)

3.  Sign in with your assigned cloud slice account.

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%203/media/image47.png)

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%203/media/image48.png)

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%203/media/image49.png)

## Exercise 5: Enable SQL Server 2025 AI Capabilities

1.  In **SSMS → New Query** → run the below query:

    ```
    CREATE DATABASE ContosoClinicalReports;
    GO
    USE ContosoClinicalReports;
    GO
    ```

    ![A screenshot of a computer Description automatically
    generated](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%203/media/image51.png)

2.  Run below query to enable outbound REST.

    ```
    USE master;
    GO
    sp_configure 'external rest endpoint enabled', 1;
    GO
    RECONFIGURE WITH OVERRIDE;
    GO
    ```

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%203/media/image52.png)

3.  Run below query to **enable preview features in database.** This
    enables VECTOR type , VECTOR INDEX , AI_GENERATE_EMBEDDINGS and
    VECTOR_SEARCH

    ```
    USE ContosoClinicalReports;
    GO
    ALTER DATABASE SCOPED CONFIGURATION  
    SET PREVIEW_FEATURES = ON;
    GO
    ```

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%203/media/image53.png)

4.  Run below query to create ClinicalReports table

    ```
    CREATE TABLE dbo.ClinicalReports
    (
        ReportID      INT           NOT NULL PRIMARY KEY,
        AgeGroup      NVARCHAR(50)  NULL,
        Department    NVARCHAR(100) NULL,
        ReportDate    DATE          NULL,
        ReportText    NVARCHAR(MAX) NULL,   -- discharge summary / note text
        PatientMRN    NVARCHAR(20)  NULL,   -- will be masked later
        Diagnosis     NVARCHAR(500) NULL
    );
    GO
    ```

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%203/media/image54.png)

5.  Run below query to **Create Master Key.**

    **Note: If you have used different password then replace the password in
    the query below.**

    ```
    USE ContosoClinicalReports;
    GO
    IF NOT EXISTS (SELECT 1 FROM sys.symmetric_keys WHERE name = '##MS_DatabaseMasterKey##')
    BEGIN
        CREATE MASTER KEY ENCRYPTION BY PASSWORD = 'AZvmsql12345';
    END
    GO 
    ```

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%203/media/image55.png)

6.  Run below query to create Database Scoped Credential.

    **Note: Replace CREDENTIAL value with OpenAI Endpoint and SECRET value
    with OpenAI Key in the below query.**

    ```
    CREATE DATABASE SCOPED CREDENTIAL [Paste OpenAI Endpoint] 
    WITH 
       IDENTITY = 'HTTPEndpointHeaders', 
       SECRET   = '{"api-key":"Paste OpenAI Key"}' 
    GO 
    ```

    ![A screenshot of a computer Description automatically
    generated](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%203/media/image56.png)

7.  Run below query to create External Model.

    >[!Note] Replace Location with text-embedding URL from Microsoft Foundry
    Portal and Credential value with OpenAI Endpoint.

    ```
    CREATE EXTERNAL MODEL ClinicalEmbeddingModel 
    WITH ( 
        -- Full embeddings endpoint URL: host + deployment + path + api-version 
        LOCATION   = 'https://azsqlaoai0216.openai.azure.com/openai/deployments/embeddings/embeddings?api-version=2024-02-15-preview', 
        API_FORMAT = 'Azure OpenAI', 
        MODEL_TYPE = EMBEDDINGS, 
        MODEL      = 'text-embedding-3-small', 
        -- Reference the credential we created above (named by host URL) 
        CREDENTIAL = [Paste OpenAI Endpoint], 
        PARAMETERS = '{ "sql_rest_options": { "retry_count": 10 } }'  
    ); 
    GO 
    ```

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%203/media/image57.png)

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%203/media/image58.png)

8.  Run below query to create Credential for Blob Storage.

	>[!Note] Replace the SECRET value with Storage account SAS token.

    ```
    -- Create a credential for Blob storage
    CREATE DATABASE SCOPED CREDENTIAL ClinicalReportsBlobCred
    WITH IDENTITY = 'SHARED ACCESS SIGNATURE',
        SECRET = '<Paste the SAS Token>';
    GO
    ```

    ![A screenshot of a computer Description automatically
    generated](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%203/media/image59.png)

9.  Run below query to create the external data source credential.

	>[!Note] Replace the Location value with your storage account name.

    ```
    -- Now create the external data source using that credential
    CREATE EXTERNAL DATA SOURCE ClinicalReportsBlob
    WITH (
        TYPE = BLOB_STORAGE,
        LOCATION = 'https://<storage account name>.blob.core.windows.net/public',
        CREDENTIAL = ClinicalReportsBlobCred
    );
    GO
    ```

    ![A screenshot of a computer Description automatically
    generated](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%203/media/image60.png)

10.  Run below query to bulk insert:

    ```
    TRUNCATE TABLE dbo.ClinicalReports;
    GO
    BULK INSERT dbo.ClinicalReports
    FROM 'clinical_reports.csv'
    WITH
    (
        DATA_SOURCE       = 'ClinicalReportsBlob',
        FORMAT            = 'CSV',
        FIRSTROW          = 2,
        FIELDTERMINATOR   = ',',
        ROWTERMINATOR     = '0x0a',
        FIELDQUOTE        = '"',
        CODEPAGE          = '65001',
        TABLOCK
    );
    GO
    ```

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%203/media/image61.png)

11.  Run below query to verify the table data:

    ```
    SELECT COUNT(*) FROM dbo.ClinicalReports;
    ```

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%203/media/image62.png)

## Exercise 6: Azure OpenAI Integration (SQL Server 2025 Pattern)

1.  Run below query to test the embeddings

    ```
    SELECT AI_GENERATE_EMBEDDINGS
    (
    N'Patient with fever and cough' 
    USE MODEL ClinicalEmbeddingModel
    ) AS Emb;
    GO
    ```

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%203/media/image63.png)

## Exercise 7: Create Embeddings Table

1.  Run below query

    ```
    DROP TABLE IF EXISTS dbo.ReportEmbeddings;
    GO

    CREATE TABLE dbo.ReportEmbeddings
    (
        ReportID INT PRIMARY KEY,
        Embedding VECTOR(1536)
    );
    GO
    ```

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%203/media/image64.png)

2.  Run below query

    ```
    INSERT INTO dbo.ReportEmbeddings (ReportID, Embedding)
    SELECT ReportID,
        AI_GENERATE_EMBEDDINGS (ReportText USE MODEL ClinicalEmbeddingModel)
    FROM dbo.ClinicalReports;
    GO
    ```

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%203/media/image65.png)

    >[!Note] As the number of rows are too much, session can be interrupted
    sometimes. In this case, you can load the embeddings in two batches.
    First load 100 rows and then again 100 rows with the help of below
    query: (Run the below query twice to load 200 records)

    ```
    INSERT INTO dbo.ReportEmbeddings (ReportID, Embedding)
    SELECT TOP (100) ReportID,
        AI_GENERATE_EMBEDDINGS (ReportText USE MODEL ClinicalEmbeddingModel)
    FROM dbo.ClinicalReports
    WHERE ReportID NOT IN (SELECT ReportID FROM dbo.ReportEmbeddings);
    ```

	You can check how many rows are processed:

    ```
    SELECT COUNT(*) AS ProcessedReports
    FROM dbo.ReportEmbeddings;
    ```

    ![A screenshot of a computer Description automatically
    generated](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%203/media/image66.png)

## Exercise 8: Create Vector Index (DiskANN) and Exact vs ANN Search

1.  Run below query to create vector index

    ```
    CREATE VECTOR INDEX IX_ReportEmbeddings
    ON dbo.ReportEmbeddings (Embedding)
    WITH (METRIC = 'cosine', TYPE = 'diskann');
    GO
    ```

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%203/media/image67.png)

2.  Run below query for exact search

    ```
    DECLARE @query NVARCHAR(MAX) =
    N'post-operative wound infection';

    DECLARE @qvec VECTOR(1536);
    SELECT @qvec = AI_GENERATE_EMBEDDINGS(@query USE MODEL ClinicalEmbeddingModel);
    SELECT TOP 5
        r.ReportID,
        r.Department,
        VECTOR_DISTANCE('cosine', @qvec, e.Embedding) AS Distance
    FROM dbo.ReportEmbeddings e
    JOIN dbo.ClinicalReports r ON r.ReportID = e.ReportID
    ORDER BY Distance ASC;
    ```

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%203/media/image68.png)

3.  Run the query to ANN Search (DiskANN)

    ```
    USE ContosoClinicalReports;   -- use your exact DB name
    GO

    -- 1) Doctor query
    DECLARE @prompt NVARCHAR(MAX) =
    N'post-operative wound infection with fever and pain';

    -- 2) Create the query embedding
    DECLARE @qvec VECTOR(1536);
    SELECT @qvec = AI_GENERATE_EMBEDDINGS(@prompt USE MODEL ClinicalEmbeddingModel);

    -- 3) ANN search using VECTOR_SEARCH (DiskANN index recommended)
    SELECT TOP (5)
        r.ReportID,
        r.Department,
        r.ReportDate,
        LEFT(r.ReportText, 250) AS ReportPreview,
        s.distance AS CosineDistance,
        CAST(1.0 - s.distance AS DECIMAL(19,6)) AS Similarity
    FROM VECTOR_SEARCH(
            TABLE      = dbo.ReportEmbeddings AS t,
            COLUMN     = Embedding,
            SIMILAR_TO = @qvec,
            METRIC     = 'cosine',
            TOP_N      = 5
        ) AS s
    JOIN dbo.ReportEmbeddings e
    ON t.ReportID = e.ReportID
    JOIN dbo.ClinicalReports r
    ON r.ReportID = e.ReportID
    ORDER BY s.distance ASC;
    GO
    ```

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%203/media/image69.png)

4.  Run below query Measure Recall (Exact vs ANN)

    ```
    CREATE OR ALTER PROCEDURE dbo.MeasureRecall
    @query NVARCHAR(MAX),
    @top INT = 5
    AS
    BEGIN
        DECLARE @qvec VECTOR(1536);
        SELECT @qvec = AI_GENERATE_EMBEDDINGS(@query USE MODEL ClinicalEmbeddingModel);
        WITH exact AS (
            SELECT TOP (@top) ReportID
            FROM dbo.ReportEmbeddings
            ORDER BY VECTOR_DISTANCE('cosine', @qvec, Embedding)
        ),
        ann AS (
            SELECT TOP (@top) t.ReportID
            FROM VECTOR_SEARCH(
                TABLE = dbo.ReportEmbeddings AS t,
                COLUMN = Embedding,
                SIMILAR_TO = @qvec,
                METRIC = 'cosine',
                TOP_N = @top
            ) AS s
        )
        SELECT COUNT(*) * 1.0 / @top AS Recall
        FROM exact
        WHERE ReportID IN (SELECT ReportID FROM ann);
    END
    GO
    ```

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%203/media/image70.png)

## Exercise 9: Apply Masking + RBAC

1.  Run the query

    ```
    ALTER TABLE dbo.ClinicalReports
    ALTER COLUMN PatientMRN NVARCHAR(20)
    MASKED WITH (FUNCTION = 'partial(2,"XXX",0)');
    GO
    CREATE ROLE HospitalStaffRole;
    GRANT SELECT ON dbo.ClinicalReports TO HospitalStaffRole;
    GO
    ```

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%203/media/image71.png)

## Exercise 10: Build the Core Search Stored Procedure with SSMS Copilot 

**Goal**: Create the semantic search logic — use free SSMS Copilot to
speed up coding.

1.  In SSMS → Tools → Options → Copilot → Sign in with your GitHub
    account

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%203/media/image72.png)

2.  New query window → open Copilot chat pane or type /copilot

3.  Ask Copilot:

    +++Generate stored procedure for semantic search on clinical reports
      using vector embeddings and Azure OpenAI+++

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%203/media/image73.png)

4.  Open a new query and click **Apply** on the Copilot response, and then run
    the query

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%203/media/image74.png)

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%203/media/image75.png)


## Conclusion:

By completing this lab, participants have successfully provisioned infrastructure, enabled SQL Server 2025 AI features, generated vector embeddings using Azure OpenAI, implemented semantic search with DiskANN indexing, and secured sensitive patient data using masking and role-based access control. They also exposed the search functionality as a secure REST API for hospital applications. Overall, learners gained practical experience in building an end-to-end AI-powered, privacy-compliant clinical search system that combines database intelligence, cloud AI services, and secure API development into one integrated solution.
