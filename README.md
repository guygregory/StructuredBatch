# Azure OpenAI - Structured Outputs & Batch
Recently, Microsoft Learn was updated to include [additional documentation on how to use Structured Outputs with Global Batch deployments](https://learn.microsoft.com/en-us/azure/ai-services/openai/how-to/batch?tabs=structured-outputs%2Cpython-secure&pivots=programming-language-ai-studio#global-batch-support).

This allows customers and partners to combining two relatively new Azure OpenAI features, to go from this batch input...

| Custom ID | Model Name    | Prompt                                       | Required                                  |
|-----------|---------------|----------------------------------------------|-------------------------------------------|
| task-0    | gpt-4o-batch  | Provide info about the Empire State Building | BuildingName, HeightInFeet, City, Country |
| task-1    | gpt-4o-batch  | Provide info about the Shard in London       | BuildingName, HeightInFeet, City, Country |
| task-2    | gpt-4o-batch  | Provide info about the Burj Khalifa          | BuildingName, HeightInFeet, City, Country |

...to this response (reformatted as a table for easy viewing):

| Custom ID | BuildingName          | HeightinFeet   | City          | Country              |
|-----------|-----------------------|----------------|---------------|----------------------|
| task-0    | Empire State Building | 1,454          | New York City | United States        |
| task-1    | The Shard             | 1,016          | London        | United Kingdom       |
| task-2    | Burj Khalifa          | 2,717          | Dubai         | United Arab Emirates |

## Structured Outputs
[**Structured Outputs**](https://learn.microsoft.com/azure/ai-services/openai/how-to/structured-outputs) ensures model-generated outputs conform exactly to developer-provided JSON Schemas, solving challenges around generating structured data from unstructured inputs. For an example, check out my [Structured Outputs demo](https://github.com/guygregory/Art-JSON).

![image](https://github.com/user-attachments/assets/9fd0d6fa-fd58-4745-b743-55177e4d0c74)

## Batch API
[**Global Batch**](https://learn.microsoft.com/azure/ai-services/openai/how-to/batch) enables efficient large-scale processing of asynchronous requests with a separate quota, 24-hour target turnaround, and 50% lower cost than standard, by sending multiple requests in a single JSONL batch file.

# Using Structured Outputs with Global Batch deployments in Azure OpenAI Service

If you want to use both of these features at the same time, firstly, deploy gpt-4o in a [supported region](https://learn.microsoft.com/en-gb/azure/ai-services/openai/how-to/batch?tabs=standard-input%2Cpython-secure&pivots=programming-language-python#region-and-model-support) using the "Global Batch" deployment type. Be sure to use version 2024-08-06 or newer for Structured Output support. At time of writing (Oct '24), only gpt-4o version: 2024-08-06 supports structured outputs.

In this example, I've deployed gpt-4o 2024-08-06 into Sweden Central, and have named it 'gpt-4o-batch'.

![image](https://github.com/user-attachments/assets/53bc4557-f89b-498e-b075-0aa0fc9c161e)

The requests for batch are stored in a JSONL file, one request per line. Each line contains the deployment name, the system message, the user message, and the JSON Schema (download this sample JSONL file [here](https://github.com/guygregory/StructuredBatch/blob/main/batch%20-%20StructuredOutputs.jsonl)):
```
{"custom_id": "task-0", "method": "POST", "url": "/chat/completions", "body": {"model": "gpt-4o-batch", "messages": [{"role": "system", "content": "You are an AI assistant that provides facts about tall buildings."}, {"role": "user", "content": "Provide info about the Empire State Building"}], "response_format": {"type": "json_schema", "json_schema": {"name": "building_info_schema", "schema": {"type": "object", "properties": {"BuildingName": {"type": "string"}, "HeightInFeet": {"type": "string"}, "City": {"type": "string"}, "Country": {"type": "string"}}, "required": ["BuildingName", "HeightInFeet", "City", "Country"], "additionalProperties": false}, "strict": true}}}}
{"custom_id": "task-1", "method": "POST", "url": "/chat/completions", "body": {"model": "gpt-4o-batch", "messages": [{"role": "system", "content": "You are an AI assistant that provides facts about tall buildings."}, {"role": "user", "content": "Provide info about the Shard in London"}], "response_format": {"type": "json_schema", "json_schema": {"name": "building_info_schema", "schema": {"type": "object", "properties": {"BuildingName": {"type": "string"}, "HeightInFeet": {"type": "string"}, "City": {"type": "string"}, "Country": {"type": "string"}}, "required": ["BuildingName", "HeightInFeet", "City", "Country"], "additionalProperties": false}, "strict": true}}}}
{"custom_id": "task-2", "method": "POST", "url": "/chat/completions", "body": {"model": "gpt-4o-batch", "messages": [{"role": "system", "content": "You are an AI assistant that provides facts about tall buildings."}, {"role": "user", "content": "Provide info about the Burj Khalifa"}], "response_format": {"type": "json_schema", "json_schema": {"name": "building_info_schema", "schema": {"type": "object", "properties": {"BuildingName": {"type": "string"}, "HeightInFeet": {"type": "string"}, "City": {"type": "string"}, "Country": {"type": "string"}}, "required": ["BuildingName", "HeightInFeet", "City", "Country"], "additionalProperties": false}, "strict": true}}}}
```

Here's what one of these requests looks like if we reformat for easier readability:
> [!IMPORTANT]  
> In your final JSONL file, ensure that each request is on a single line to ensure compliance to the JSONL format
```
{
  "custom_id": "task-0",
  "method": "POST",
  "url": "/chat/completions",
  "body": {
    "model": "gpt-4o-batch",
    "messages": [
      {
        "role": "system",
        "content": "You are an AI assistant that provides facts about tall buildings."
      },
      {
        "role": "user",
        "content": "Provide info about the Empire State Building"
      }
    ],
    "response_format": {
      "type": "json_schema",
      "json_schema": {
        "name": "building_info_schema",
        "schema": {
          "type": "object",
          "properties": {
            "BuildingName": {
              "type": "string"
            },
            "HeightInFeet": {
              "type": "string"
            },
            "City": {
              "type": "string"
            },
            "Country": {
              "type": "string"
            }
          },
          "required": [
            "BuildingName",
            "HeightInFeet",
            "City",
            "Country"
          ],
          "additionalProperties": false
        },
        "strict": true
      }
    }
  }
}
```

# Python Notebook Example - Commentary
[This Python notebook](https://github.com/guygregory/StructuredBatch/blob/main/batch%20-%20StructuredOutputs.ipynb) walks through the steps required to upload a batch file, submit it for processing, track its progress, and retrieve structured outputs using Azure OpenAI's Batch API. It's based on the sample in the [Microsoft Learn documentation](https://learn.microsoft.com/en-gb/azure/ai-services/openai/how-to/batch?tabs=standard-input%2Cpython-key&pivots=programming-language-python).

## Step 1: Uploading the Batch File
In this section, the notebook sets up the Azure OpenAI client, using environment variables for the API endpoint and key. It then uploads a file containing batch requests, specifying its purpose as batch.
> [!IMPORTANT]  
> Ensure the API is set to 2024-10-01-preview or later, and the gpt-4o model is deployed in a supported region
```
from dotenv import load_dotenv
import os
from openai import AzureOpenAI

load_dotenv()

client = AzureOpenAI(
    azure_endpoint = os.getenv("AZURE_OPENAI_API_ENDPOINT"),
    api_key=os.getenv("AZURE_OPENAI_API_KEY"),  
    api_version="2024-10-01-preview"
)

# Upload a file with a purpose of "batch"
file = client.files.create(
  file=open("batch - StructuredOutputs.jsonl", "rb"), 
  purpose="batch"
)

print(file.model_dump_json(indent=2))
file_id = file.id
```
### Explanation:
- The client is initialized using environment variables to keep sensitive information secure.
- The batch file is opened and uploaded with the purpose batch, which is used for asynchronous processing.

**Key Output:** The file_id is captured, which will be used in subsequent steps to reference the file.

## Step 2: Tracking File Upload Status
After uploading the batch file, the next step is to track its upload status. This loop checks the file status every 15 seconds until the file is processed.
```
import time
import datetime 

status = "pending"
while status != "processed":
    time.sleep(15)
    file_response = client.files.retrieve(file_id)
    status = file_response.status
    print(f"{datetime.datetime.now()} File Id: {file_id}, Status: {status}")
```
**Explanation:** The script repeatedly checks the status of the uploaded file. The loop pauses for 15 seconds between each check until the status becomes "processed", indicating that the file is ready for use.

## Step 3: Creating a Batch Job
Once the file is processed, we can submit it as a batch job. The batch request is processed asynchronously, and the notebook tracks the job's progress.
```
batch_response = client.batches.create(
    input_file_id=file_id,
    endpoint="/chat/completions",
    completion_window="24h",
)

# Save batch ID for later use
batch_id = batch_response.id

print(batch_response.model_dump_json(indent=2))
```
**Explanation:** This section submits the batch job, which processes multiple requests in parallel using the provided batch file.
The completion_window is set to 24 hours, meaning the job is expected to complete within that time.

## Step 4: Tracking Batch Job Progress
Just like tracking the file upload, this code tracks the status of the batch job until it reaches a terminal state (completed, failed, or canceled).
```
status = "validating"
while status not in ("completed", "failed", "canceled"):
    time.sleep(60)
    batch_response = client.batches.retrieve(batch_id)
    status = batch_response.status
    print(f"{datetime.datetime.now()} Batch Id: {batch_id},  Status: {status}")
```
**Explanation:** The batch status is updated in real-time and printed in intervals of 60 seconds.
Once the batch job reaches the "completed" status, you can proceed to retrieve the output.

## Step 5: Examining Job Status Details
After the batch job is completed, the response contains various metadata and details about the processing status.
```
print(batch_response.model_dump_json(indent=2))
```
**Explanation:** This code provides a detailed view of the batch job's final status, including whether any errors occurred and how many requests were successfully completed.

## Step 6: Retrieving Batch Job Output
Once the batch job completes, this step retrieves and prints the output for each request in the batch, formatting the responses into readable JSON.
```
import json

response = client.files.content(batch_response.output_file_id)
raw_responses = response.text.strip().split('\n')  

for raw_response in raw_responses:  
    json_response = json.loads(raw_response)  
    formatted_json = json.dumps(json_response, indent=2)  
    print(formatted_json)
```
**Explanation:** The output of the batch job is stored in a file on Azure. Each response is retrieved, parsed from JSON, and formatted for readability.

## Additional Batch Commands
The notebook also provides examples of how to cancel a batch job and list all batch jobs submitted.

## Cancel a Batch Job
```
client.batches.cancel("batch_abc123")  # Set to your batch_id for the job you want to cancel
```
**Explanation:** If necessary, you can cancel a batch job that is still in progress.

## List All Batch Jobs
```
client.batches.list()
```
**Explanation:** This command lists all batch jobs submitted to your Azure OpenAI service, showing their status and other details.

## Conclusion
By following the steps in [this notebook](https://github.com/guygregory/StructuredBatch/blob/main/batch%20-%20StructuredOutputs.ipynb), users can automate batch processing of Azure OpenAI requests with structured outputs, track the status of their jobs, and retrieve the results in a scalable and efficient manner.
