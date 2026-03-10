# Lab 2: Developing a Knowledge-Augmented Medical Library Research Assistant with RAG and Azure SQL Database

In this lab, you take on the role of building a Medical Research
Assistant that helps users explore trusted medical research more
efficiently. Instead of manually browsing through multiple documents,
users can ask questions in natural language and receive clear,
research-backed answers. By integrating database storage, intelligent
search, and AI capabilities, you create a secure solution that ensures
responses are grounded only in verified medical research data.

**Objectives:**

By completing this lab, you will be able to:

- Create and configure an Azure SQL Database to store medical research
  data

- Import and manage structured research documents

- Generate embeddings for research content

- Configure vector and hybrid search for intelligent data retrieval

- Implement a Retrieval-Augmented Generation (RAG) architecture

- Build and test an AI-powered chat agent

- Ensure responses are grounded strictly in trusted enterprise data

## Exercise 1: Create SQL Database Server 

1.  Open a browser and go to +++https://portal.azure.com+++ and sign in with
    your Azure subscription account.
	
	Username: +++@lab.CloudPortalCredential(User1).Username+++
	
	TAP: +++@lab.CloudPortalCredential(User1).AccessToken+++

2.  Search for +++Azure SQL+++ and select it.

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image1.png)

3.  Expand **Azure SQL Database-\> SQL logical servers** from the left menu and select **Create.**

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image2.png)

4.  Enter the values below and click on **Next: Networking**

    - Subscription: **@lab.CloudSubscription.Name**

    - Resource Group: **@lab.CloudResourceGroup(ResourceGroup1).Name**

    - Server Name: +++azdbsqlserver@lab.labinstance.id+++

    - Region: **@lab.CloudResourceGroup(ResourceGroup1).Location**

    - Authentication method: Use SQL authentication

    - Server admin login: +++sqladmin+++

    - Password: +++Pa55w0rd12345+++

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image3.png)

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image4.png)

5.  Toggle **Yes** to Allow Azure services and resources to access this
    server, and then click on **Next: Security.**

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image5.png)

6.  Click on the **Configure identities** link under the Identity
    section.

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image6.png)

7.  Select **Status On** and click on Apply.

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image7.png)

8.  Click on **Review +create.**

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image8.png)

9.  Review the details and click on **Create.**

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image9.png)

10. Wait for the deployment to be completed, and then click on **Go to resource.**

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image10.png)

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image11.png)

11. Click on Networking from the left navigation under security, click on Add your
    client and then click on save.

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image12.png)

12. Click on Overview and copy the **server name** to **Notepad** to use
    in the next task.

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image13.png)

## Exercise 2: Create an Azure SQL Database

1.  Enter +++Azure SQL+++ in the search bar and select **Azure SQL database.**

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image14.png)

2.  Select **Create -\> SQL database.**

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image15.png)

3.  Select below values:

    - Name: +++ContosoMedicalResearch+++

    - Server: **azdbsqlserver@lab.labinstance.id**

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image16.png)

4. Keep the default values and then click on Next: Networking.

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image17.png)

5. Enable Add current client IP address and then click **Review+create.**

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image18.png)

6. Review the details and click on Create.

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image19.png)

7. Wait for the deployment and click on **Go to resource.**

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image20.png)

## Exercise 3: Create a database via SSMS and upload csv file.

1.  Double-click on SSMS from task bar and select **Sign in with Microsoft**

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image21.png)

2.  Select Work or School account and click **Continue**.

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image22.png)

3.  Sign in with your assigned cloud slice account.

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image23.png)

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image24.png)

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image25.png)

4.  Enter below details and click **Continue**:

    - Server name: **The Server name you copied previously**

    - Authentication : **SQL Server Authentication**

    - Username : +++sqladmin+++

    - Password : +++Pa55w0rd12345+++

    - Select **Trust Server certificate** checkbox

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image26.png)

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image27.png)

5.  Select the Copilot icon in the top right, Prompt Copilot to create a database:

	+++@workspace, create a SQL Server database names COntosoMedicalResearch with default settings+++

    **Note:** Write any prompt and send it, it will ask you to sign in first.

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image28.png)

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image29.png)

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image30.png)

9.  Right-click **DB → Tasks → Import Flat File** as shown in the image
    below.

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image31.png)

10.  Click Next on Import flat file page,

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image32.png)

11.  Browse the file library_books.csv form C:\Labfiles folder, enter the
    table name as +++MedicalResearch+++ and click **Next**.

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image33.png)

11. Preview the data and click Next.

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image34.png)

12. Click Next on Modify column.

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image35.png)

13. On the summary page, click Finish.

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image36.png)

14. Once the operation completed, close the window.

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image37.png)

15. Run below queries to verify the data in the table.

    ```
    SELECT COUNT(*) FROM dbo.MedicalResearch;
    SELECT TOP 5 Title, Category FROM dbo.MedicalResearch;
    ```

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image38.png)

## Exercise 4: Create Azure OpenAI service and deploy chat and embedding models

1.  Switch back to Azure and search for +++Azure OpenAI+++ and select it.

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image39.png)

2.  Click on Create-\> Azure OpenAI.

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image40.png)

3.  Enter below and click **Next**.

    - Subscription: **@lab.CloudSubscription.Name**

    - Resource Group: **@lab.CloudResourceGroup(ResourceGroup1).Name**

    - Region: **@lab.CloudResourceGroup(ResourceGroup1).Location**

    - Name: +++azsqlaoai@lab.labinstance.id-lab2+++

    - Pricing tier: **Standard S0**

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image41.png)

4.  Keep the default value and click Next.

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image42.png)

5.  Keep the default tag and click Next.

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image43.png)

6.  Review the details and click Create.

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image44.png)

7.  Wait for the deployment to be successful and click on **Go to resource.**

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image45.png)

8.  Expand Resource management-\> keys and endpoints from the left
    navigation menu and copy the endpoint and key value to a notepad to
    use in the next tasks.

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image46.png)

9.  Click on **Overview** and select **Go to Foundry portal.**

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image47.png)

10. Click on Deployments under Shared resource from the left navigation
    menu. Select Deploy model-\> Deploy base model.

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image48.png)

11. Search for +++text-embedding+++, select **text-embedding-3-small** model
    and click Confirm.

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image49.png)

12. Keep the default values and click Customise.

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image50.png)

13. Set Tokens per Minute Rate limit to max and click Deploy.

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image51.png)

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image52.png)

14. Click on Deployments from the left navigation menu, select **Deploy
    model-\> Deploy base model.**

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image53.png)

15. Search for +++gpt+++ models and select the **gpt-5.2-chat** model, and
    click Confirm.

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image54.png)

16. Click on **Customize** to edit deployment details.

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image55.png)

17. **Increase the Token per Minute Rate limit** and then click on Create resource and deploy.

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image56.png)

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image57.png)

## Exercise 5: Create Azure AI Search Service

1.  Switch back to the Azure portal tab. Enter +++AI search+++ in the
    search bar and select AI Search.

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image58.png)

1.  Click on Create.

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image59.png)

2.  Enter the details below and click **Review + Create**

    - Resource Group: **@lab.CloudResourceGroup(ResourceGroup1).Name**

    - Name: +++azsearchrag@lab.labinstance.id+++

    - Region: **@lab.CloudResourceGroup(ResourceGroup1).Location**

    - Pricing tier: **Standard**.

    ![A screenshot of a computer Description automatically
    generated](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image60.png)

1.  Review the details and click on Create.

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image61.png)

2.  Wait for the deployment to complete and click on **Go to resource**.

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image62.png)

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image63.png)

## Exercise 6: Create Search Index

1.  Switch back to the AI Service tab and click on **Import data(new)**

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image64.png)

2.  Select the **Azure SQL Database** tile.

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image65.png)

3.  Select the **RAG** scenario.

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image66.png)

4.  Enter below details and click **Next**.

    - Subscription: **@lab.CloudSubscription.Name**

    - Azure SQL account type: **SQL database**

    - Server: **azdbsqlserver@lab.labinstance.id**

    - Database: **ContosoMedicalResearch**

    - Table or View: **Table**

    - Schema : **dbo**

    - Table name: +++MedicalResearch+++

    - SQL server authentication password: +++Pa55w0rd12345+++

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image67.png)

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image68.png)

5.  In Vectorise your text, enter the details below and click Next.

    - Column to vectorise: **Summary**

    - Kind: Azure OpenAI

    - Subscription: **@lab.CloudSubscription.Name**

    - Azure OpenAI Service: **azsqlaoai@lab.labinstance.id-lab2**

    - Model deployment : text-embedding-3-small

    - Authentication type : API key

    - Acknowledge the service

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image69.png)

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image70.png)

6.  Keep all default values and click Next.

	![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image71.png)

7.  Review the details and click Create.

	![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image72.png)

8.  Click **Go to Search Explorer**.

	![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image73.png)

9.  Enter the prompts, check vectors and embeddings and verify vectors
    and score differences.

    +++Recent advances in oncology treatment+++

    +++Immunotherapy in lung cancer+++

    +++AI diagnostics in cardiology+++

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image74.png)

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image75.png)

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image76.png)

## Exercise 7: Build RAG in Azure AI Foundry

1.  Open a new tab and go to +++https://ai.azure.com+++ and sign in with
    your Azure subscription account.

2.  Click on Start building to navigate to the new Microsoft Foundry
    portal.

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image77.png)

3.  **Click on Drop down and select Create a new project.**

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image78.png)

4.  Expand Advanced options.

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image79.png)

5.  Enter the details below and click Create.

    - Project Name: +++Foundryproj-@lab.labinstance.id-lab2+++

    - Region: **East US 2**

    - Resource group: Resourcegroup1

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image80.png)

6.  From the top navigation menu, select **Discover.**

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image81.png)

7.  Click on **Models** from left navigation menu, search for
    +++gpt-5.2-chat+++ and select it.

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image82.png)

8.  Select **Deploy -\> Default settings**.

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image83.png)

9.  Click on Knowledge from the left navigation menu.

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image84.png)

10. Select your AI Search service and API key then click on Connect.

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image85.png)

11. Click on Create a Knowledge base.

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image86.png)

12. Select the Azure AI Search Index and then click Connect.

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image87.png)

13. Enter +++knowledgebase1+++ for your knowledge base, select your
    **search index** from the drop-down and **create**.

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image88.png)

14. Select the **model** drop-down and browse more models.

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image89.png)

15. Select the gpt-4.1 model as gpt-5.2 is unavailable, and then **Save
    knowledge base.**

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image90.png)

16. Click on Save now.

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image91.png)

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image92.png)

17. Click on **Agents** from the left navigation menu,

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image93.png)

18. Click on Create agent.

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image94.png)

19. Enter the unique agent name and click Create.

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image95.png)

20. On Playground, expand Knowledge-\> Add and select Connect to Foundry
    IQ.

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image96.png)

21. Select your knowledge base and click on Connect.

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image97.png)

22. Enter the text in the **Instructions** text box and click on
    **Save**

    ```
    POLICY:
    1) You MUST call the Azure AI Search tool for every user message.
    2) You MUST include only facts present in the tool output.
    3) If the tool returns no items, respond: 
    "I do not have enough information in the knowledge sources to answer this."
    4) Never answer from general knowledge. Never fabricate sources.

    POLICY (Grounding & Citations):
    1) You MUST invoke the Azure AI Search tool for every user message.
    2) Use ONLY facts from the tool output to craft the answer.
    3) Provide clear citations (Title or Id) inline like [1], [2], and at the end under "Sources".
    4) If the tool returns no items, respond exactly:
    "I do not have enough information in the knowledge sources to answer this."
    5) Never answer from general knowledge. Never speculate.
    Formatting:
    - Start with a concise, factual answer.
    - Then add a short bulleted rationale with specific lines from the retrieved passages.
    - End with a "Sources" list using titles or ids from the tool output.
    ```

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image98.png)

23. Expand Tools and select Add.

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image99.png)

24.  Select Azure AI Search and click on Add tool.

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image100.png)

25.  Select the Azure search connection and index, and then click on Add.

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image101.png)

26.  Select your Azure Search tool and click on Save.

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image102.png)

27. Enter the prompts below and check the response

    Ask:

    +++What are recent advancements in cancer treatment?+++

    +++List books and their authors related to AI in radiology diagnostics+++

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image103.png)

13. Approve the tool in the chat window.

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image104.png)

14. The response is generated from the knowledge base. Preview the agent
    by clicking on Preview -\> Preview agent.

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image105.png)

15. Enter the prompt:

    List books and their authors related to AI in radiology diagnostics

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image106.png)

16. Switch back to the Foundry agent tab and click on **Publish -\> Publish
    agent.**

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image107.png)

    ![](https://raw.githubusercontent.com/technofocus-pte/sqlaidevlprdepth/refs/heads/main/Lab%20Guides/Lab%202/media/image108.png)

## Conclusion

By the end of this lab, you have successfully built a Medical Research
Assistant that can understand questions, retrieve relevant research
content, and generate meaningful responses based strictly on trusted
data. You combined database storage, intelligent search, and AI-driven
chat to create a complete end-to-end solution. Most importantly, you
ensured that every response is grounded in verified medical research,
demonstrating how AI can be used responsibly and effectively in
real-world healthcare and research environments.











