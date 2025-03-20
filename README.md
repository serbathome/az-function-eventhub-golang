# Azure Function with Event Hub Trigger in Go

This project demonstrates an Azure Function with an Event Hub trigger implemented in Go using Azure Functions Custom Handlers.

## Project Overview

This solution processes events from an Azure Event Hub using a serverless Azure Function written in Go. The application receives Event Hub messages and forwards them to a configurable endpoint.

## Architecture

- **Azure Event Hub**: Source of events that trigger the function
- **Azure Function**: Serverless compute that processes incoming events
- **Go Custom Handler**: Implementation of the function logic in Go

## Components

### Event Hub Trigger

The function is triggered when new events arrive in the specified Event Hub named "demo". Configuration for the trigger can be found in `EventHubTrigger1/function.json`.

### Custom Handler (Go)

The Go application (`handler.go`) serves as a custom handler for Azure Functions. It:

1. Listens on the port specified by `FUNCTIONS_CUSTOMHANDLER_PORT` environment variable
2. Processes HTTP requests that represent Event Hub trigger events
3. Forwards the event data to another API endpoint defined by the `CATCHER_API_ENDPOINT` environment variable
4. Returns a success response after processing

### Sample Receiver Endpoint

Below is a sample Flask application that can be used as the `CATCHER_API_ENDPOINT` to receive and process the forwarded Event Hub messages:

```python
from flask import Flask, request

app = Flask(__name__)

@app.route('/api/catchall', methods=['POST'])
def catch_all():
    data = request.get_json()
    print("Received data:", data)
    return {"status": "success"}, 200

if __name__ == '__main__':
    app.run(debug=True, host='0.0.0.0', port=5000)
```

To use this sample:
1. Save it as `app.py`
2. Install Flask: `pip install flask`
3. Run: `python app.py`
4. Set `CATCHER_API_ENDPOINT=http://localhost:5000/api/catchall` in your local environment

#### Exposing Local Endpoint with Dev Tunnel

If you need to expose your local CATCHER_API_ENDPOINT to the internet (for testing with Azure Functions in the cloud), you can use Microsoft Dev Tunnel:

1. Install the Dev Tunnel CLI:
   ```bash
   # Using npm
   npm install -g @microsoft/dev-tunnels-cli
   # Or using the Azure CLI extension
   az extension add --name tunnel
   ```

2. Create and host a tunnel:
   ```bash
   # Using Dev Tunnel CLI
   devtunnel create --allow-anonymous
   devtunnel host --port 5000
   
   # Or using Azure CLI
   az tunnel create --allow-anonymous
   az tunnel host --port 5000
   ```

3. Get the tunnel URL:
   ```bash
   devtunnel info
   # Or
   az tunnel info
   ```

4. Update your Azure Function app settings to use the tunnel URL:
   ```
   CATCHER_API_ENDPOINT=https://<tunnel-url>/api/catchall
   ```

This allows your Azure Function to send data to your local Flask application, even when the Function is running in the Azure cloud.

## Setup and Configuration

### Prerequisites

- [Azure Functions Core Tools](https://github.com/Azure/azure-functions-core-tools)
- [Go](https://golang.org/dl/) (1.16 or later recommended)
- Azure subscription with Event Hub namespace and Event Hub instance

### Environment Variables

- `FUNCTIONS_CUSTOMHANDLER_PORT`: Port for the custom handler (default: 8080)
- `CATCHER_API_ENDPOINT`: The URL to which Event Hub messages should be forwarded

### Local Settings

Create a `local.settings.json` file in the project root (this file is excluded from git):

```json
{
  "IsEncrypted": false,
  "Values": {
    "FUNCTIONS_WORKER_RUNTIME": "custom",
    "AzureWebJobsStorage": "UseDevelopmentStorage=true",
    "eh7555_RootManageSharedAccessKey_EVENTHUB": "YOUR_EVENT_HUB_CONNECTION_STRING",
    "CATCHER_API_ENDPOINT": "https://your-target-api.example.com/endpoint"
  }
}
```

## Running Locally

1. Build the Go handler:
   ```bash
   go build -o handler.exe handler.go
   ```

2. Start the Azure Functions runtime:
   ```bash
   func start
   ```

## Deployment to Azure

1. Build the Go binary for your target environment:
   ```bash
   GOOS=linux GOARCH=amd64 go build -o handler handler.go
   ```
   (Use `handler.exe` for Windows deployments)

2. Deploy to Azure using Azure Functions Core Tools:
   ```bash
   func azure functionapp publish <your-function-app-name>
   ```

3. Configure application settings in the Azure portal:
   - Set `CATCHER_API_ENDPOINT` to your target API endpoint
   - Ensure the Event Hub connection string is properly configured

## Project Structure

- `EventHubTrigger1/function.json`: Defines the Event Hub trigger binding
- `handler.go`: Go code that implements the custom handler
- `host.json`: Azure Functions host configuration
- `.vscode/`: VSCode configuration for Azure Functions development

## Troubleshooting

- Check Azure Function logs for execution errors
- Verify Event Hub connection string and configuration
- Ensure the target API endpoint (`CATCHER_API_ENDPOINT`) is accessible


