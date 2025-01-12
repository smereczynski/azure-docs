---
title: Anomaly Detector .NET client library quickstart
titleSuffix: Azure Cognitive Services
services: cognitive-services
author: mrbullwinkle
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 09/22/2020
ms.author: mbullwin
---

Get started with the Anomaly Detector client library for C#. Follow these steps to install the package start using the algorithms provided by the service. The Anomaly Detector service enables you to find abnormalities in your time series data by automatically using the best-fitting models on it, regardless of industry, scenario, or data volume.

Use the Anomaly Detector client library for C# to:

* Detect anomalies throughout your time series data set, as a batch request
* Detect the anomaly status of the latest data point in your time series
* Detect trend change points in your data set.

[Library reference documentation](https://aka.ms/anomaly-detector-dotnet-ref) | [Library source code](https://github.com/Azure/azure-sdk-for-net/tree/main/sdk/anomalydetector) | [Package (NuGet)](https://www.nuget.org/packages/Azure.AI.AnomalyDetector/3.0.0-preview.2) | [Find the code on GitHub](https://github.com/Azure-Samples/AnomalyDetector/blob/master/quickstarts/sdk/csharp-sdk-sample.cs)

## Prerequisites

* Azure subscription - [Create one for free](https://azure.microsoft.com/free/cognitive-services)
* The current version of [.NET Core](https://dotnet.microsoft.com/download/dotnet-core)
* Once you have your Azure subscription, <a href="https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesAnomalyDetector"  title="Create an Anomaly Detector resource"  target="_blank">create an Anomaly Detector resource </a> in the Azure portal to get your key and endpoint. Wait for it to deploy and click the **Go to resource** button.
    * You will need the key and endpoint from the resource you create to connect your application to the Anomaly Detector API. You'll paste your key and endpoint into the code below later in the quickstart.
    You can use the free pricing tier (`F0`) to try the service, and upgrade later to a paid tier for production.

## Setting up

[!INCLUDE [anomaly-detector-environment-variables](../environment-variables.md)]

### Create a new .NET Core application

In a console window (such as cmd, PowerShell, or Bash), use the `dotnet new` command to create a new console app with the name `anomaly-detector-quickstart`. This command creates a simple "Hello World" project with a single C# source file: *Program.cs*.

```dotnetcli
dotnet new console -n anomaly-detector-quickstart
```

Change your directory to the newly created app folder. You can build the application with:

```dotnetcli
dotnet build
```

The build output should contain no warnings or errors.

```output
...
Build succeeded.
 0 Warning(s)
 0 Error(s)
...
```

### Install the client library

Within the application directory, install the Anomaly Detector client library for .NET with the following command:

```dotnetcli
dotnet add package Microsoft.Azure.CognitiveServices.AnomalyDetector
```

From the project directory, open the *program.cs* file and add the following using `directives`:

[!code-csharp[using statements](~/samples-anomaly-detector/quickstarts/sdk/csharp-sdk-sample.cs?name=usingStatements)]

In the application's `main()` method, create variables for your resource's Azure location, and your key as an environment variable. If you created the environment variable after application is launched, the editor, IDE, or shell running it will need to be closed and reloaded to access the variable.

[!code-csharp[Main method](~/samples-anomaly-detector/quickstarts/sdk/csharp-sdk-sample.cs?name=mainMethod)]

## Object model

The Anomaly Detector client is a [AnomalyDetectorClient](/dotnet/api/microsoft.azure.cognitiveservices.anomalydetector.anomalydetectorclient) object that authenticates to Azure using [ApiKeyServiceClientCredentials](/dotnet/api/microsoft.azure.cognitiveservices.anomalydetector.apikeyserviceclientcredentials), which contains your key. The client can do anomaly detection on an entire dataset using [EntireDetectAsync()](/dotnet/api/microsoft.azure.cognitiveservices.anomalydetector.anomalydetectorclientextensions.entiredetectasync), or on the latest data point using [LastDetectAsync()](/dotnet/api/microsoft.azure.cognitiveservices.anomalydetector.anomalydetectorclientextensions.lastdetectasync). The [ChangePointDetectAsync](https://aka.ms/anomaly-detector-dotnet-ref) method detects points that mark changes in a trend.

Time series data is sent as a series of [Points](/dotnet/api/microsoft.azure.cognitiveservices.anomalydetector.models.request.series#Microsoft_Azure_CognitiveServices_AnomalyDetector_Models_Request_Series) in a [Request](/dotnet/api/microsoft.azure.cognitiveservices.anomalydetector.models.request) object. The `Request` object contains properties to describe the data ([Granularity](/dotnet/api/microsoft.azure.cognitiveservices.anomalydetector.models.request.granularity) for example), and parameters for the anomaly detection.

The Anomaly Detector response is either an [EntireDetectResponse](/dotnet/api/microsoft.azure.cognitiveservices.anomalydetector.models.entiredetectresponse), [LastDetectResponse](/dotnet/api/microsoft.azure.cognitiveservices.anomalydetector.models.lastdetectresponse), or [changePointDetectResponse](https://aka.ms/anomaly-detector-dotnet-ref) object, depending on the method used.

## Code examples

These code snippets show you how to do the following with the Anomaly Detector client library for .NET:

* [Authenticate the client](#authenticate-the-client)
* [Load a time series data set from a file](#load-time-series-data-from-a-file)
* [Detect anomalies in the entire data set](#detect-anomalies-in-the-entire-data-set)
* [Detect the anomaly status of the latest data point](#detect-the-anomaly-status-of-the-latest-data-point)
* [Detect the change points in the data set](#detect-change-points-in-the-data-set)

## Authenticate the client

In a new method, instantiate a client with your endpoint and key. Create an [ApiKeyServiceClientCredentials](/dotnet/api/microsoft.azure.cognitiveservices.anomalydetector.apikeyserviceclientcredentials) object with your key, and use it with your endpoint to create an [AnomalyDetectorClient](/dotnet/api/microsoft.azure.cognitiveservices.anomalydetector.anomalydetectorclient) object.

[!code-csharp[Client authentication function](~/samples-anomaly-detector/quickstarts/sdk/csharp-sdk-sample.cs?name=createClient)]

## Load time series data from a file

Download the example data for this quickstart from [GitHub](https://github.com/Azure-Samples/AnomalyDetector/blob/master/example-data/request-data.csv):
1. In your browser, right-click **Raw**.
2. Click **Save link as**.
3. Save the file to your application directory, as a .csv file.

This time series data is formatted as a .csv file, and will be sent to the Anomaly Detector API.

Create a new method to read in the time series data and add it to a [Request](/dotnet/api/microsoft.azure.cognitiveservices.anomalydetector.models.request) object. Call `File.ReadAllLines()` with the file path and create a list of [Point](/dotnet/api/microsoft.azure.cognitiveservices.anomalydetector.models.point) objects, and strip any new line characters. Extract the values and separate the datestamp from its numerical value, and add them to a new `Point` object.

Make a `Request` object with the series of points, and `Granularity.Daily` for the [Granularity](/dotnet/api/microsoft.azure.cognitiveservices.anomalydetector.models.granularity) (or periodicity) of the data points.

[!code-csharp[load the time series data file](~/samples-anomaly-detector/quickstarts/sdk/csharp-sdk-sample.cs?name=GetSeriesFromFile)]

## Detect anomalies in the entire data set

Create a method to call the client's [EntireDetectAsync()](/dotnet/api/microsoft.azure.cognitiveservices.anomalydetector.anomalydetectorclientextensions.entiredetectasync#Microsoft_Azure_CognitiveServices_AnomalyDetector_AnomalyDetectorClientExtensions_EntireDetectAsync_Microsoft_Azure_CognitiveServices_AnomalyDetector_IAnomalyDetectorClient_Microsoft_Azure_CognitiveServices_AnomalyDetector_Models_Request_System_Threading_CancellationToken_) method with the `Request` object and await the response as an [EntireDetectResponse](/dotnet/api/microsoft.azure.cognitiveservices.anomalydetector.models.entiredetectresponse) object. If the time series contains any anomalies, iterate through the response's [IsAnomaly](/dotnet/api/microsoft.azure.cognitiveservices.anomalydetector.models.entiredetectresponse.isanomaly) values and print any that are `true`. These values correspond to the index of anomalous data points, if any were found.

[!code-csharp[EntireDetectSampleAsync() function](~/samples-anomaly-detector/quickstarts/sdk/csharp-sdk-sample.cs?name=entireDatasetExample)]

## Detect the anomaly status of the latest data point

Create a method to call the client's [LastDetectAsync()](/dotnet/api/microsoft.azure.cognitiveservices.anomalydetector.anomalydetectorclientextensions.lastdetectasync#Microsoft_Azure_CognitiveServices_AnomalyDetector_AnomalyDetectorClientExtensions_LastDetectAsync_Microsoft_Azure_CognitiveServices_AnomalyDetector_IAnomalyDetectorClient_Microsoft_Azure_CognitiveServices_AnomalyDetector_Models_Request_System_Threading_CancellationToken_) method with the `Request` object and await the response as a [LastDetectResponse](/dotnet/api/microsoft.azure.cognitiveservices.anomalydetector.models.lastdetectresponse) object. Check the response's [IsAnomaly](/dotnet/api/microsoft.azure.cognitiveservices.anomalydetector.models.lastdetectresponse.isanomaly) attribute to determine if the latest data point sent was an anomaly or not.

[!code-csharp[LastDetectSampleAsync() function](~/samples-anomaly-detector/quickstarts/sdk/csharp-sdk-sample.cs?name=latestPointExample)]

## Detect change points in the data set

Create a method to call the client's [DetectChangePointAsync](https://aka.ms/anomaly-detector-dotnet-ref) method with the `Request` object and await the response as a [ChangePointDetectResponse](https://aka.ms/anomaly-detector-dotnet-ref) object. Check the response's IsChangePoint values and print any that are `true`. These values correspond to trend change points, if any were found.

[!code-csharp[DetectChangePoint() function](~/samples-anomaly-detector/quickstarts/sdk/csharp-sdk-sample.cs?name=changePointExample)]

## Run the application

Run the application with the `dotnet run` command from your application directory.

```dotnetcli
dotnet run
```

[!INCLUDE [anomaly-detector-next-steps](../quickstart-cleanup-next-steps.md)]
