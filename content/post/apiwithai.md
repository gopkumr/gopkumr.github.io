---
title: "Spam sensitive APIs using Azure AI services and Semantic Kernel"
date: 2024-03-11T19:33:51+11:00
draft: true 
tags: ["Azure AI", "Semantic Kernel", "Azure Function", "Azure API Management", "REST"]
---

Text classification is a common task in natural language processing, where we want to assign a label or category to a piece of text based on its content. For example, we may want to classify emails as spam or not spam, or news articles as sports, politics, entertainment, etc. There are many ways to approach this problem, but in this blog post, I will show you how to use Azure AI services and Semantic Kernel to create a text classification API that can handle custom classes and data.

What are Azure AI services and Semantic Kernel?
Azure AI services are a collection of cloud-based APIs that provide various capabilities for building intelligent applications, such as computer vision, natural language processing, speech recognition, and more. You can use these services to easily add AI features to your apps without having to worry about the underlying infrastructure, scalability, or security.

Semantic Kernel is an open-source SDK that lets you easily build agents that can call your existing code. As a highly extensible SDK, you can use Semantic Kernel with models from OpenAI, Azure OpenAI, Hugging Face, and more. By combining your existing C#, Python, and Java code with these models, you can build agents that answer questions, generate content, perform tasks, and more.

In this blog post, we will use the Custom Text Classification service from Azure AI Language, which allows us to build custom models for text classification tasks. We will also use the Semantic Kernel SDK to create a C# agent that can call our custom text classification model and expose it as an API using Azure Functions.

How to create a custom text classification model?
To create a custom text classification model, we need to follow these steps:

Prepare our data: We need to have a set of text documents and their corresponding labels. The labels should be predefined by us and match our desired classes. For example, if we want to classify emails as spam or not spam, we need to have a dataset of emails and their spam status. We also need to split our data into training, validation, and test sets, and store them in CSV files.
Create a project in Language studio: Language studio is a web portal that allows us to create and manage our custom text classification projects. We need to sign in with our Azure account and create a new project. We need to specify the project name, description, type (single label or multi label), and language. We also need to upload our CSV files and map the columns to the text and label fields.
Label our data: In order to train our model, we need to label our data manually or use active learning. Active learning is a feature that suggests labels for our data based on the existing labels and the model’s predictions. We can review and accept or reject the suggested labels, or add our own labels. We need to label at least 50 documents per class to start training our model.
Train and evaluate our model: Once we have labeled enough data, we can start training our model. We can choose the training time and the number of iterations. The model will automatically optimize the hyperparameters and the architecture. We can monitor the training progress and the metrics, such as accuracy, precision, recall, and F1-score. We can also test our model on new data and see the predictions and the confidence scores.
Publish our model: After we are satisfied with our model’s performance, we can publish it to make it available for consumption. We need to provide a model name, description, and version. We can also enable authentication and encryption for our model endpoint. We will get a model ID and an API key that we can use to call our model from our code.
How to create a text classification API using Semantic Kernel and Azure Functions?
To create a text classification API using Semantic Kernel and Azure Functions, we need to follow these steps:

Create a function app project in Visual Studio: We need to have Visual Studio 2022 with the Azure development workload installed. We can create a new project and select the Azure Functions template. We need to provide a project name, location, and solution name. We also need to choose the Functions worker as .NET 8.0 Isolated (Long Term Support) and the Function as HTTP trigger.
Add the Semantic Kernel SDK and the Azure AI Language SDK to our project: We need to install the Semantic Kernel SDK and the Azure AI Language SDK as NuGet packages to our project. We can use the Package Manager Console or the Manage NuGet Packages option to do so. We need to install the following packages:
SemanticKernel
Azure.AI.Language
Create a C# agent that can call our custom text classification model: We need to create a C# class that inherits from the SemanticKernel.Agent class and implements the OnMessageAsync method. This method will be invoked when we receive an HTTP request with a text input. We need to use the Azure AI Language SDK to create a client for our custom text classification model and pass our model ID and API key. We also need to create a TextClassificationInput object with our text input and call the ClassifyTextAsync method to get the prediction result. We can then return the result as a JSON string. Here is an example of how our C# agent class can look like:
C#

using System.Threading.Tasks;
using Azure.AI.Language;
using SemanticKernel;

public class TextClassificationAgent : Agent
{
    // The model ID and API key for our custom text classification model
    private const string modelId = "your-model-id";
    private const string apiKey = "your-api-key";

    // The client for our custom text classification model
    private readonly TextClassificationClient client;

    public TextClassificationAgent()
    {
        // Create the client for our custom text classification model
        client = new TextClassificationClient(modelId, new AzureKeyCredential(apiKey));
    }

    public override async Task<string> OnMessageAsync(string message)
    {
        // Create a text classification input with our text input
        var input = new TextClassificationInput(message);

        // Call the classify text method to get the prediction result
        var result = await client.ClassifyTextAsync(input);

        // Return the result as a JSON string
        return result.ToJson();
    }
}
AI-generated code. Review and use carefully. More info on FAQ.
Create a function that can invoke our C# agent and expose it as an API: We need to create a function that can create an instance of our C# agent and invoke its OnMessageAsync method with the text input from the HTTP request. We also need to set the route for our function to match our desired API endpoint. We can use the Route attribute to do so. Here is an example of how our function can look like:
C#

using System.IO;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Azure.Functions.Worker;
using Microsoft.Azure.Functions.Worker.Http;

public static class TextClassificationFunction
{
    // The route for our function
    [Function("TextClassificationFunction")]
    [Route("api/textclassification")]
    public static async Task<IActionResult> RunAsync(
        [HttpTrigger(AuthorizationLevel.Anonymous, "post")] HttpRequestData req,
        FunctionContext executionContext)
    {
        // Read the text input from the request body
        var reader = new StreamReader(req.Body);
        var text = await reader.ReadToEndAsync();

        // Create an instance of our C# agent
        var agent = new TextClassificationAgent();

        // Invoke the agent's OnMessageAsync method with the text input
        var response = await agent.OnMessageAsync(text);

        // Return the response as an OK result
        return new OkObjectResult(response);
    }
}
AI-generated code. Review and use carefully. More info on FAQ.
Run and test our function locally: We can run and test our function locally using Visual Studio. We can use the F5 key or the Debug > Start Debugging option to start the function app. We can use a tool like Postman to send HTTP requests to our function endpoint and see the responses. We can also use the Azure Storage Emulator or Azurite to simulate the storage account that our function app requires.
Deploy our function app to Azure: We can deploy our function app to Azure using Visual Studio. We can use the Publish option to create or select an existing function app in Azure. We need to provide the app name, subscription, resource group, and region. We also need to create or select an existing storage account for our function app. We can then publish our function app and see it running in Azure.
Conclusion
In this blog post, we have seen how to use Azure AI services and Semantic Kernel to create a text classification API. We have created a custom text classification model using Language studio and a C# agent that can call our model using Semantic Kernel. We have also exposed our agent as an API using Azure Functions. We have run and tested our function locally and deployed it to Azure. We have used a simple example of classifying emails as spam or not spam, but we can use the same approach for any text classification task with custom classes and data. I hope you have found this blog post useful and interesting. If you have any questions or feedback, please feel free to leave a comment below. Thank you for reading! 😊