# Sentiment-Analysis-Azure-Functions

a Serverless HTTP API with Azure Functions that takes a sentence as an input and returns the sentiment of the sentence.

## Prerequisites

- An **Azure Subscription** (e.g. [Free](https://aka.ms/azure-free-account) or [Student](https://aka.ms/azure-student-account) account)
- macOS, Windows, or Linux
- Python 3.7, 3.8, or 3.9
- [Azure Functions Core Tools](https://docs.microsoft.com/azure/azure-functions/functions-run-local#install-the-azure-functions-core-tools). If needed, there are [more ways to install here](https://github.com/Azure/azure-functions-core-tools#installing).
- [Visual Studio Code](https://code.visualstudio.com/download) with these extensions installed:
    - [Python](https://marketplace.visualstudio.com/items?itemName=ms-python.python)
    - [Azure Functions](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions)

## Create a local Azure Functions app and a new function

Azure Functions allows you to build and deploy your code as functions, without worrying about managing servers and other infrastructure. Your functions are triggered by events such as: items added on a queue, a document updated in a database, or an HTTP request.

You will build an HTTP API using an Azure Function. An Azure Functions app can contain one or more Azure Functions.

1. Open a new VS Code window.

2. Open the command palette by pressing `F1`, `Ctrl-Shift-P` (Windows or Linux), `Cmd-Shift-P` (macOS).

3. Search for and select **Azure Functions: Create new project...**.

4. Browse to a location where you want to create your project. Create a new empty folder and click **Select** to open it.

5. A series of prompts will appear. Enter the following values:

    | Prompt | Value | Description |
    | --- | --- | --- |
    | Language for your project | **Python** | |
    | Python interpreter | Select one | Azure Functions requires Python 3.7, 3.8, 3.9 |
    | Template for your first function | **HTTP trigger** | |
    | Function name | **sentiment** | |
    | Authorization level | **anonymous** | Allow function to be accessed anonymously |
    | How to open new project | **Open in current window** | |

VS Code reopens in the function app. Here are some important files and folders that you'll need to know for this lab:
- `.venv/` - A virtual environment is automatically created for you using the version of Python you selected. When running your function app, it runs in this virtual environment.
- `.vscode/` - VS Code settings, including the tasks needed to run and debug the function app.
- `sentiment/`
    - `__init__.py` - Your new function
    - `function.json` - Metadata for your function
- `requirements.txt` - Contains Python dependencies for your app

## Modify the function to perform sentiment analysis

1. From the VS Code explorer, open `requirements.txt`.

2. You will use the [VADER Sentiment Analysis](https://github.com/cjhutto/vaderSentiment) library in your app. On a new line in `requirements.txt`, add `vaderSentiment`.

    This will cause VS Code to install the package when you run the function app in the next section.

3. Open the function at `sentiment/__init__.py`.

4. Replace the body of the file with the following code that uses the VADER library to perform sentiment analysis on some input text that is passed to the function and returns the result:

    ```python
    import azure.functions as func
    from vaderSentiment.vaderSentiment import SentimentIntensityAnalyzer


    def main(req: func.HttpRequest) -> func.HttpResponse:
        analyzer = SentimentIntensityAnalyzer()
        text = req.params.get("text")
        scores = analyzer.polarity_scores(text)
        sentiment = "positive" if scores["compound"] > 0 else "negative"
        return func.HttpResponse(sentiment)
    ```

## Run and debug the app

1. Start debugging in VS Code by pressing `F5` or by selecting **Debug: Start debugging** from the command palette.

2. If you don't have the Azure Functions Core Tools installed, VS Code will prompt you to install it. Select **Install**.

    > If the installation fails, follow [these instructions](https://github.com/Azure/azure-functions-core-tools#installing) to install it.

3. Once the function app has started, open your web browser and enter the following in the address bar to run your function:

    ```
    http://localhost:7071/api/http_trigger?text=I+love+PyCon
    ```

    It should return a result of `positive`.

4. In `sentiment/__init__.py`, set a breakpoint by placing your cursor on the first line in the function and pressing `F9`.

5. Return to your browser and enter the above URL again (or you can replace the `text` query string with a sentence of your own).

    VS Code should stop on the breakpoint. You can step through the code with `F10` and inspect the values of variables.

6. Stop debugging by pressing `Shift-F5`.

## Create Azure resources for your function

Before you can deploy your function code to Azure, you need to create several supporting resources inside a resource group, such as a Function App itself, a Storage account (for storing the code), and an Application Insights instance (to provide monitoring).

1. In the command palette, search for and select **Azure Functions: Create new Function App in Azure (Advanced)**.

2. When prompted to select a subscription, select **Sign in to Azure**.

3. The browser should open to the Azure sign in page. Log in with your Azure credentials.

4. Enter the following values in the prompts:

    | Prompt | Description |
    | --- | --- |
    | Enter a globally unique name for the new function app | Type something like `yourname-sentiment-analyzer-function`
    | Select a runtime stack | Select the same version as you used to create your local function app |
    | Select a resource group | If you were provided lab credentials, you must select the existing resource group from the dropdown |
    | Select a location for new resources | Select whatever location is closest to you geographically |
    | Select a hosting plan | Select **Consumption** |
    | Select a storage account | Select **Create new storage account** and accept suggested name |
    | Select an Application Insights resource for your app | Select **Create new Application Insights resource** and accept suggested name |
    
5. You should see both a notification and a message in the **Azure: Activity Log** panel once the Function App resources are successfully created.  

## Deploy the app to Azure

You will now use VS Code to actually deploy the local function app to the Azure resources you just created.

1. In the command palette, search for and select **Azure Functions: Deploy to Function App**.

2. When it prompts you to "Select a resource", select the name of the resource created in the previous step. In the dialog box that pops up, confirm that you do intend to overwrite the previously deployed version. Since this is your first time deploying, that shouldn't be a concern.

    Watch the deployment progress in the notification.
    
3. When the deployment is complete, A prompt should appear. Click on **View output**.

4. In the output, locate the HTTP trigger URL, it should look like `https://<function-app-name>.azurewebsites.net/api/http_trigger`.

    Copy it and paste it into your browser's address bar. Append `?text=I+love+PyCon` to the end of the URL and press enter.

    You should see the sentiment returned.
