# Azure Function Deployment Guide

This repo's intent is to deploy an Azure Function App with **4 functions** (modify the functions to vary these requirements)  

## **Perform OCR**

```http
URL: https://function-app-name.azurewebsites.net/api/perform_ocr
```

Inputs: storage_account_name, blob_url  
Output: {success: True, message: "OCR done", result_blob_name, container_name}  

## **Create Excel**

```http
URL: https://function-app-name.azurewebsites.net/api/create_excel
```

Inputs: ocr_result_blob_name, storage_account_name  
Output: {success: True, message: "Excel created", result_blob_name, container_name}  

## **Parse OCR**

```http
URL: https://function-app-name.azurewebsites.net/api/parse_ocr
```

Inputs: ocr_result_blob_name, storage_account_name  
Output: {summary_report_blob_name, summary_container_name}  

## **Clean Up**

```http
URL: https://function-app-name.azurewebsites.net/api/clean_up
```

Inputs: storage_account_name, blob_url  
Output: {success: True, message}  

## Azure Portal View

![Azure Portal Function App](./images/functions.png =25%x)

The function app requires **4 containers** in the same storage account (modify the functions to vary these requirements)  
(Use a different storage account than you the one created with you Function App)  

![Azure Portal Containers](./images/containers.png =35%x)  

## Prerequisites

1. Install Azure Functions Core Tools: <https://docs.microsoft.com/en-us/azure/azure-functions/functions-run-local>  
2. Install Azure CLI: <https://docs.microsoft.com/en-us/cli/azure/install-azure-cli>  

## Local Development Setup

### 1. Create and activate virtual environment

```bash
python -m venv .venv
.venv\Scripts\Activate.ps1  # On Windows PowerShell  
# or
.venv\Scripts\activate.bat  # On Windows Command Prompt  
```

### 2. Install dependencies

```bash
pip install -r requirements.txt  
```

### 3. Configure local.settings.json

Update the `local.settings.json` file with your actual values:  

- AZURE_CLIENT_ID: Your Azure AD application client ID  
- AZURE_CLIENT_SECRET: Your Azure AD application client secret  
- AZURE_TENANT_ID: Your Azure AD tenant ID  
- SERVICE_FOR_CU: Your Content Understanding service endpoint  
- SERVICE_API_FOR_CU: Your Content Understanding service API version  

### 4. Run locally  

```bash
func start  
```

## Deployment to Azure

### 1. Login to Azure

```bash
azd login auth  
```

use the popup to select your logon and click ok. MFA may be required.

### or

```bash
az login  
```

use the popup to select your logon and click ok. MFA may be required.  
Respond to prompt on command window by selecting your subscription.  

### The 4 functions are shown below

### 2. Create a Function App (if not exists)

```bash
az functionapp create --resource-group <resource-group-name> --consumption-plan-location <location> --runtime python --runtime-version 3.9 --functions-version 4 --name <function-app-name> --storage-account <storage-account-name>  
```

### 3. Deploy the function

```bash
func azure functionapp publish function-app-name  
```

### 4. Configure application settings in Azure

```bash
az functionapp config appsettings set --name function-app-name --resource-group <resource-group-name> --settings AZURE_CLIENT_ID=<your-client-id>  
az functionapp config appsettings set --name function-app-name --resource-group <resource-group-name> --settings AZURE_CLIENT_SECRET=<your-client-secret>  
az functionapp config appsettings set --name function-app-name --resource-group <resource-group-name> --settings AZURE_TENANT_ID=<your-tenant-id>  
az functionapp config appsettings set --name function-app-name --resource-group <resource-group-name> --settings SERVICE_FOR_CU=<your-service-endpoint>  
az functionapp config appsettings set --name function-app-name --resource-group <resource-group-name> --settings SERVICE_API_FOR_CU=<your-api-version>  
... and more
```

## Usage

### HTTP Request Format

#### Perform OCR - POST

```http
POST https://function-app-name.azurewebsites.net/perform_ocr,  
Content-Type: application/json

{
    "classifier_id": "your-classifier-id",  
    "blob_url": "https://your_storage_account.blob.core.windows.net/container/file.pdf",  
    "storage_account_name": "your_storage_account"  
}
```

#### Perform OCR Response Format

```json
{
    "success": true,
    "message": "OCR processing completed successfully",  
    "result_blob_name": "Excel created",  
    "container_name": "enhanced-results"  
}
```

#### Parse OCR - POST

```http
https://function-app-name.azurewebsites.net/parse_ocr  

{
    "ocr_result_blob_name": "result_blob_name",  
    "storage_account_name": "your_storage_account"  
}
```

#### Parse OCR Response Format

```json
{
    "success": true,
    "message": "OCR processing completed successfully",  
    "summary_report_blob_name": "file.pdf_your_timestamp.json",  
    "summary_container_name": "enhanced-results"  
}
```

#### Create Excel - POST

```http
https://function-app-name.azurewebsites.net/parse_ocr  

{
    "ocr_result_blob_name": "result_blob_name",  
    "storage_account_name": "your_storage_account"  
}
```

#### Create Excel Response Format

```json
{
    "success": true,
    "message": "OCR processing completed successfully",  
    "result_blob_name": "file.pdf_your_timestamp.json",  
    "container_name": "enhanced-results"  
}
```

#### Clean Up  - POST

```http
https://function-app-name.azurewebsites.net/clean_up?blob_url=URL_to_your_PDF&storage_account_name=your_storage_account_name  

{
    "blob_url": "https://your_storage_account.blob.core.windows.net/container/file.pdf",  
    "storage_account_name": "your_storage_account"  
}
```

#### Clean Up  Response Format

```json
{
    "success": true,
    "message": "OCR processing completed successfully",  
    "result_blob_name": "file.pdf_your_timestamp.json",  
    "container_name": "enhanced-results"  
}
```

### Alternative - Using Curl

#### Perform OCR - Curl

```bash
Perform OCR:
curl -G "https://function-app-name.azurewebsites.net/api/perform_ocr" --data-urlencode "classifier_id=<your_classifier_id>" --data-urlencode "blob_url=<blob-url>" --data-urlencode "storage_account_name=<account-name>"  
Output: {success:True,message:"OCR done",result_blob_name,container_name}.  
```

#### Create Excel - Curl

```bash
curl -G "https://function-app-name.azurewebsites.net/api/create_excel" --data-urlencode "ocr_result_blob_name=<result_blob_name>" --data-urlencode "storage_account_name=<account-name>"  
Output: {success:True,message:"Excel created",result_blob_name,container_name}.  
```

#### Parse OCR - Curl

```bash
curl -G "https://function-app-name.azurewebsites.net/api/parse_ocr" --data-urlencode "ocr_result_blob_name=<result_blob_name>" --data-urlencode "storage_account_name=<account-name>"  
Output: {summary_report_blob_name,summary_container_name}.  
```

#### Clean Up - Curl

```bash
curl -G "https://function-app-name.azurewebsites.net/api/clean_up" --data-urlencode "blob_url=<blob-url>" --data-urlencode "storage_account_name=<account-name>"  
Output: {success:True,message}.  
```

Each function’s output feeds into the next step. Ensure inputs match the logic app parameters.  

### Alternative - Query Parameters

#### Perform OCR - Query Parameters

```http
https://function-app-name.azurewebsites.net/perform_ocr?classifier_id=your_classifier&blob_url=URL_to_your_PDF&storage_account_name=your_storage_account_name  
```

#### Perform OCR Response Format - Query Parameters

```json
{
    "success": true,
    "message": "OCR processing completed successfully",  
    "result_blob_name": "file.pdf_20241016_143022.json",  
    "container_name": "enhanced-results"  
}
```

#### Create Excel - Query Parameters

```http
https://function-app-name.azurewebsites.net/create_excel?ocr_result_blob_name=your_result_blob_name&storage_account_name=your_storage_account_name  
```

#### create_excel Response Format - Query Parameters

```json
{
    "success": true,  
    "message": "Excel created",  
    "result_blob_name": "Excel created",  
    "container_name": "enhanced-results"  
}
```

#### Parse OCR - Query Parameters

```http
https://function-app-name.azurewebsites.net/parse_ocr?ocr_result_blob_name=your_result_blob_name&storage_account_name=your_storage_account_name  
```

#### Parse_ OCR Response Format - Query Parameters

```json
{
    "success": true,  
    "message": "OCR processing completed successfully",  
    "result_blob_name": "file.pdf_20241016_143022.json",  
    "container_name": "enhanced-results"  
}
```

#### Clean Up - Query Parameters

```http
https://function-app-name.azurewebsites.net/clean_up?blob_url=URL_to_your_PDF&storage_account_name=your_storage_account_name
```

#### Clean Up Response Format - Query Parameters

```json
{
    "success": true,  
    "message": "Clean up completed successfully"
}
```

## Follow Best Security Practices  

You should follow best practices for your Function Apps as described on these pages

<https://learn.microsoft.com/en-us/azure/azure-functions/security-concepts>  
<https://learn.microsoft.com/en-us/azure/azure-functions/functions-identity-based-connections-tutorial-2>  

and for App Services as described on these pages

<https://learn.microsoft.com/en-us/azure/app-service/overview-security>
<https://learn.microsoft.com/en-us/azure/app-service/overview-authentication-authorization>
