# Lab 1: SQL-First Semantic Case Retrieval Agent with SQL Server 2025 and Azure OpenAI

**Objectives**

The overall objective of this lab is to design and implement a
**SQL-first semantic retrieval system** using SQL Server 2025 and Azure
OpenAI, enabling natural language clinical case search directly within
the database engine.

### Exercise 1: **Provision SQL Server on Azure VM**

1.  Open browser enter  Azure SQL hub at
    +++https://aka.ms/azuresqlhub+++ or
    +++https://portal.azure.com+++ and sign in with your Azure subscription
    account. Enter +++Azure SQL+++ in the search bar and select it.

    ![](./media/image1.png)

2.  In the pane for **SQL Server on Azure Virtual Machines**,
    select **Show options**.

    ![](./media/image2.png)

3.  In the **Select an image offer** box, choose a SQL Server image
    (such as **Free SQL Server License: SQL Server 2025 Enterprise Developer on Windows Server 2025**).
    Select **Create virtual machine**.

    ![](./media/image3.png)

4.  On the **Basics** tab, provide the following information and then
    click **Next: Disk**

    - Subscription :- Select your subscription

    - In the **Resource group** - select an existing resource group from the
    list or choose **Create new** to create a new resource group.

    - Virtual machine name- **azuresqlvm**

    - **Region** - Central US

    - **Availability options** – *Availability zone*.

        1.  **Zone options** – Self-selected zone

        2.  **Availability zone** -Zone 1

    - In the **Image** list, select **Free SQL Server License: SQL Server
    2025 Enterprise Developer on Windows Server 2025** if it's not already
    selected.

    - Select **See all sizes** for the **Size** of the virtual machine and
    search for the **E4ds_v5** offering. This is one of the minimum
    recommended VM sizes for SQL Server on Azure VMs. be sure to clean up
    your resources once you're done with them to prevent any unexpected
    charges.

    Enter admin details as below

    Username : **sqlvmuser**
    Password: **AZvmsql12345**

    - Under **Inbound port rules**, choose **Allow selected ports**, and
    then select **RDP (3389)** from the dropdown list.

    ![](./media/image4.png)

    ![](./media/image5.png)

    ![](./media/image6.png)

    ![](./media/image7.png)

    ![](./media/image8.png)

    ![](./media/image9.png)

5.  Keep default disk type values and click **Next: Networking**

    ![](./media/image10.png)

6.  On Management page,

    ![](./media/image11.png)

    ![](./media/image12.png)

7.  Click on **SQL Server management** tab

    ![](./media/image13.png)

8.  On SQL server setting spage, select below values and then click on **Review + create.**

    **SQL connectivity**: Public(internet)
    
    **SQL authentication:** Enable

    ![](./media/image14.png)

9.  Once the validation is passed, click on **Create**.

    ![](./media/image15.png)

10. Wait for the deployment to complete.

    ![](./media/image16.png)

    ![](./media/image17.png)

11. Make sure you copy the **Public IP Address** to connect from SSMS in
    next task

    ![](./media/image18.png)

### Exercise 2: Setup SQL Server 2025 environment 

**Goal:** You have SQL Server 2025 running and can connect with SSMS or
VS Code.

1.  Connect using SSMS:

    - Server: \<VM Public IP\>,1433 (or local instance)

    - Auth: SQL Authentication

    - Check “Trust server certificate” (if needed)

    ![](./media/image19.png)

    ![](./media/image20.png)

### Exercise 3: Create Azure OpenAI resource and deploy Embedding Model 

1.  Switch back to Azure and search for **Azure OpenAI** and select it.

    ![](./media/image21.png)

2.  Click on Create-\> Azure OpenAI.

    ![](./media/image22.png)

3.  Enter below values and click **Next**.

    Subscription: your Azure subscription

    Resource Group: Resourcegroup1

    Region: eastus

    Name: **azsqlaoaiXXXX**(replace XXXX with unique number)

    Pricing tier: Standard S0

    ![](./media/image23.png)

4.  Keep the default value and click Next.

    ![](./media/image24.png)

5.  Keep default tag and click Next.

    ![](./media/image25.png)

6.  Review the details and click Create.

    ![](./media/image26.png)

7.  Wait for the deployment successful and click on **Go to resource.**

    ![](./media/image27.png)

8.  Expand Resource management-\> keys and endpoints from left
    navigation menu and copy endpoint and key value in a notepad to use
    in next tasks.

    ![](./media/image28.png)

9.  Click on **Overview** and select **Go to Foundry portal**

    ![](./media/image29.png)

10. Click on Deployments under Shared resource from left navigation
    menu. Select Depplot model-\> Deploy base model.

    ![](./media/image30.png)

11. Search for text-embedding , select **text-embedding-3-small** model and
    click Confirm.

    ![](./media/image31.png)

12. Keep the default values and click **Customize**.

    ![](./media/image32.png)

13. Set Tokens per Minute Rate limit to max and click **Deploy**.

    ![](./media/image33.png)

    ![](./media/image34.png)

### Exercise 4: Create Data base and tables

1.  Switch back to **SSMS**. Right click on server and select New database
    to Get patient case data into SQL Server.

    ![](./media/image35.png)

2.  Enter the database name as +++ContosoHospitalDB+++ and click OK.

    +++CREATE DATABASE ContosoHospitalDB;+++

    ![](./media/image36.png)

3.  Run below query to create a table

    ```
    USE ContosoHospitalDB;
    GO

    DROP TABLE IF EXISTS dbo.PatientEmbeddings;
    DROP TABLE IF EXISTS dbo.PatientNotes;
    GO

    CREATE TABLE dbo.PatientNotes
    (
        EncounterID        INT           NOT NULL PRIMARY KEY,
        AgeGroup           NVARCHAR(50)   NULL,
        Gender             NVARCHAR(20)   NULL,
        AdmissionDate      DATE          NULL,
        Department         NVARCHAR(80)   NULL,
        ChiefComplaint     NVARCHAR(200) NULL,
        Summary            NVARCHAR(MAX) NULL,
        DiagnosisKeywords  NVARCHAR(400) NULL,
        DischargePlan      NVARCHAR(200) NULL
    );
    GO
    ```

    ![](./media/image37.png)

### Exercise 5: Import PatientNotes.csv 

1 . Run below query to Enable required SQL Server 2025 features

    ```
    USE ContosoHospitalDB;
    GO

    -- 1. Enable external REST endpoint usage
    EXEC sp_configure 'external rest endpoint enabled', 1;
    RECONFIGURE WITH OVERRIDE;
    GO

    -- 2. Enable preview features (needed for AI/vector features in many SQL 2025 builds)
    ALTER DATABASE SCOPED CONFIGURATION
    SET PREVIEW_FEATURES = ON;
    GO
    ```

    ![](./media/image38.png)

### Exercise 6: Create external embedding model

1.  Run below query to Create master key (needed once per DB)
    
    ```
    USE ContosoHospitalDB;
    GO
    IF NOT EXISTS(SELECT * FROM sys.symmetric_keys WHERE name = '##MS_DatabaseMasterKey##')
    BEGIN
        CREATE MASTER KEY ENCRYPTION BY PASSWORD = N'ContosoSQLAI@2025!';
    END;
    GO
    ```

    ![](./media/image39.png)

2.  Run below query to Create database scoped credential. The credential
    name must match the URL you reference in the external model.

    ```
    USE ContosoHospitalDB;
    GO
    IF EXISTS (SELECT 1 FROM sys.database_scoped_credentials WHERE name = N'https://<resource>.openai.azure.com/')
    BEGIN
        DROP DATABASE SCOPED CREDENTIAL [https://<resource>.openai.azure.com/];
    END
    GO

    CREATE DATABASE SCOPED CREDENTIAL [https://<resource>.openai.azure.com/]
    WITH
        IDENTITY = 'HTTPEndpointHeaders',
        SECRET = '{"api-key":"<YOUR_AZURE_OPENAI_KEY>"}'
    GO
    ```

    ![](./media/image40.png)

3.  Switch back to Foundry portal and copy the model endpoint value:

    ![](./media/image41.png)

4.  Update below query with Azure openAI end point and Embeddign smodel
    location (Foundry portal ) and run to create external model

    ```
    USE ContosoHospitalDB;
    GO

    IF EXISTS (SELECT 1 FROM sys.external_models WHERE name = 'ClinicalEmbeddingModel')
        DROP EXTERNAL MODEL ClinicalEmbeddingModel;
    GO

    CREATE EXTERNAL MODEL ClinicalEmbeddingModel
    WITH
    (
    LOCATION   = 'https://<resource>.openai.azure.com/openai/deployments/<embedding-deployment>/embeddings?api-version=2024-02-01',
    API_FORMAT = 'Azure OpenAI',
    MODEL_TYPE = EMBEDDINGS,
    MODEL      = 'text-embedding-3-small',
    CREDENTIAL = [https://<resource>.openai.azure.com/],
    PARAMETERS = '{ "sql_rest_options": { "retry_count": 10 } }'
    );
    GO
    ```

    ![](./media/image42.png)

### Exercise 7: Generate embeddings and store vectors

1.  Run below command to create embeddings table (VECTOR column)

    ```
    USE ContosoHospitalDB;
    GO

    DROP TABLE IF EXISTS dbo.PatientEmbeddings;
    GO

    CREATE TABLE dbo.PatientEmbeddings
    (
        EncounterID INT NOT NULL PRIMARY KEY,
        Embedding   VECTOR(1536) NOT NULL
    );
    GO
    ```

    ![](./media/image43.png)

2.  Run below query to populate embeddings

    ```
    USE ContosoHospitalDB;
    GO

    INSERT INTO dbo.PatientEmbeddings (EncounterID, Embedding)
    SELECT
        pn.EncounterID,
        AI_GENERATE_EMBEDDINGS(pn.Summary USE MODEL ClinicalEmbeddingModel)
    FROM dbo.PatientNotes pn
    WHERE pn.Summary IS NOT NULL;
    GO
    ```

    ![](./media/image44.png)

3.  Run below query to validate embeddings:

    +++SELECT COUNT(*) AS TotalEmbeddings FROM dbo.PatientEmbeddings;+++
    +++SELECT * FROM dbo.PatientEmbeddings;+++

    ![](./media/image45.png)

    ![](./media/image46.png)

4.  Run below query to create vector index (DiskANN)

    ```
    USE ContosoHospitalDB;
    GO

    CREATE VECTOR INDEX IX_PatientEmbeddings
    ON dbo.PatientEmbeddings (Embedding)
    WITH (METRIC = 'cosine', TYPE = 'diskann');
    GO
    ```

    ![](./media/image47.png)

### Exercise 8: Semantic case retrieval (doctor symptom query)

This exercise helps you search for patient cases that are **similar in
meaning** to a doctor's query, using vector embeddings.

1.  Run below query for exact semantic search (VECTOR_DISTANCE). No
    manual embedding, No REST API, No 1536‑value array and SQL Server
    does everything automatically

    ```
    USE ContosoHospitalDB;
    -- (No GO here if you intend to keep @qvec in the same batch)

    -- 1. Doctor's natural-language query
    DECLARE @doctorQuery NVARCHAR(MAX) =
    N'Elderly patient with shortness of breath and cough, possible pneumonia';

    -- 2. Generate a 1536-dim embedding vector from the model
    DECLARE @qvec VECTOR(1536);
    SELECT @qvec = AI_GENERATE_EMBEDDINGS(@doctorQuery USE MODEL ClinicalEmbeddingModel);

    -- 3. (Optional) Unfiltered Top 5 semantic matches
    SELECT TOP (5)
        pn.EncounterID,
        pn.Department,
        pn.AgeGroup,
        pn.AdmissionDate,
        pn.ChiefComplaint,
        pn.Summary,
        VECTOR_DISTANCE('cosine', @qvec, pe.Embedding) AS CosineDistance
    FROM dbo.PatientEmbeddings AS pe
    JOIN dbo.PatientNotes       AS pn
    ON pn.EncounterID = pe.EncounterID
    ORDER BY CosineDistance ASC;
    ```

    ![](./media/image48.png)

2.  Run semantic search with clinical filters.this query filters
    Department = Pulmonology and Age group = 60–74

    ```
    -- Recreate the vector since we’re in a new batch (or new window)
    DECLARE @doctorQuery NVARCHAR(MAX) =
    N'Elderly patient with shortness of breath and cough, possible pneumonia';

    DECLARE @qvec VECTOR(1536);
    SELECT @qvec = AI_GENERATE_EMBEDDINGS(@doctorQuery USE MODEL ClinicalEmbeddingModel);

    -- Clinical filters
    DECLARE @dept NVARCHAR(80) = N'Pulmonology';
    DECLARE @age  NVARCHAR(50) = N'60-74';

    -- Top 5 semantic matches WITH filters
    SELECT TOP (5)
        pn.EncounterID,
        pn.Department,
        pn.AgeGroup,
        pn.AdmissionDate,
        pn.ChiefComplaint,
        pn.Summary,
        VECTOR_DISTANCE('cosine', @qvec, pe.Embedding) AS CosineDistance
    FROM dbo.PatientEmbeddings AS pe
    JOIN dbo.PatientNotes       AS pn
    ON pn.EncounterID = pe.EncounterID
    WHERE (@dept IS NULL OR pn.Department = @dept)
    AND (@age  IS NULL OR pn.AgeGroup   = @age)
    ORDER BY CosineDistance ASC;
    ```

    ![](./media/image49.png)

3.  Run below query to add clinical filter

    ```
    USE ContosoHospitalDB;

    -- Recreate the query vector (new batch / new window)
    DECLARE @doctorQuery NVARCHAR(MAX) =
    N'Elderly patient with shortness of breath and cough, possible pneumonia';

    DECLARE @qvec VECTOR(1536);
    SELECT @qvec = AI_GENERATE_EMBEDDINGS(@doctorQuery USE MODEL ClinicalEmbeddingModel);

    -- Filters
    DECLARE @dept NVARCHAR(80) = N'Pulmonology';
    DECLARE @age  NVARCHAR(50) = N'60-74';

    -- Top 5 semantic matches WITH filters (exact cosine distance)
    SELECT TOP (5)
        pn.EncounterID,
        pn.Department,
        pn.AgeGroup,
        pn.AdmissionDate,
        pn.ChiefComplaint,
        pn.Summary,
        VECTOR_DISTANCE('cosine', @qvec, pe.Embedding) AS CosineDistance
    FROM dbo.PatientEmbeddings AS pe
    JOIN dbo.PatientNotes       AS pn
    ON pn.EncounterID = pe.EncounterID
    WHERE (@dept IS NULL OR pn.Department = @dept)
    AND (@age  IS NULL OR pn.AgeGroup   = @age)
    ORDER BY CosineDistance ASC;
    ```

    ![](./media/image50.png)

4.  I’ve kept your parameters and added in‑proc embedding generation.
    This SP uses exact cosine; you can add an @useAnn BIT = 0 switch
    later if you want to go the ANN→re‑rank route.

    ```
    USE ContosoHospitalDB;
    GO

    CREATE OR ALTER PROCEDURE dbo.FindSimilarPatientCases
        @doctorQuery   NVARCHAR(MAX),
        @topK          INT = 5,
        @department    NVARCHAR(80) = NULL,
        @ageGroup      NVARCHAR(50) = NULL
    AS
    BEGIN
        SET NOCOUNT ON;

        -- Create the query vector each time (proc scope)
        DECLARE @qvec VECTOR(1536);
        SELECT @qvec = AI_GENERATE_EMBEDDINGS(@doctorQuery USE MODEL ClinicalEmbeddingModel);

        SELECT TOP (@topK)
            pn.EncounterID,
            pn.Department,
            pn.AgeGroup,
            pn.AdmissionDate,
            pn.ChiefComplaint,
            pn.DiagnosisKeywords,
            pn.Summary,
            VECTOR_DISTANCE('cosine', @qvec, pe.Embedding) AS CosineDistance
        FROM dbo.PatientEmbeddings AS pe
        JOIN dbo.PatientNotes       AS pn
        ON pn.EncounterID = pe.EncounterID
        WHERE (@department IS NULL OR pn.Department = @department)
        AND (@ageGroup   IS NULL OR pn.AgeGroup   = @ageGroup)
        ORDER BY CosineDistance ASC;
    END
    GO

    ```
    ![](./media/image51.png)

5.  Run below query to test:

    ```
    -- test 1
    EXEC dbo.FindSimilarPatientCases
    @doctorQuery = N'child with wheezing and nighttime cough, asthma flare',
    @topK = 5,
    @department = N'Pediatrics';

    -- test 2 (unfiltered)
    EXEC dbo.FindSimilarPatientCases
    @doctorQuery = N'shortness of breath and fever, rule out pneumonia',
    @topK = 5;
    ```

    ![](./media/image52.png)

### Exercise 9: Compare keyword vs semantic search

1.  Run below queries and the count should match

    ```
    SELECT COUNT(\*) FROM dbo.PatientNotes;
    SELECT COUNT(\*) FROM dbo.PatientEmbeddings;
    ```

    ![](./media/image53.png)

2.  Run below query to test embedding generation:

    +++SELECT AI_GENERATE_EMBEDDINGS(N'test case' USE MODEL ClinicalEmbeddingModel);+++

    ![](./media/image54.png)

3.  Run below query -Find similar cases for fever + painful urination
    and filter to Infectious Disease

    ```
    EXEC dbo.FindSimilarPatientCases
    @doctorQuery = N'fever and painful urination, possible UTI',
    @topK = 5,
    @department = N'Infectious Disease';
    ```
    ![](./media/image55.png)

4.  Run below query with and without department filter. Observe:Mix of
    departments,Cardiology,Emergency,Internal Medicine.

    ![](./media/image56.png)

5.  Run below query With Filter (Cardiology Only). Observe: Only cardiology cases ,Possibly fewer results,Ranking slightly changes

    ```
    EXEC dbo.FindSimilarPatientCases
    @doctorQuery = N'chest pain on exertion',
    @topK = 5,
    @department = N'Cardiology';
    ```

    ![](./media/image57.png)

6.  Run below query with Hybrid search pattern

    ```
    EXEC dbo.FindSimilarPatientCases
    @doctorQuery = N'shortness of breath',
    @topK = 5;
    ```

    ![](./media/image58.png)

    ```
    EXEC dbo.FindSimilarPatientCases
    @doctorQuery = N'dyspnea',
    @topK = 5;
    ```

    ![](./media/image59.png)

## Conclusion:

This lab demonstrates how SQL Server 2025 transforms into an AI-enabled
database platform capable of storing vector embeddings, performing
semantic similarity search, and integrating directly with Azure OpenAI.
Using a realistic healthcare scenario, students build a semantic case
retrieval agent that allows doctors to query patient cases in natural
language and retrieve clinically relevant results using vector search.
