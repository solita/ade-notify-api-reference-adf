# Introduction
This repository contains a reference solution for notifying incoming source data files for Agile Data Engine Notify API (https://ade.document360.io/docs/notify-api) with Azure Data Factory (ADF).

**The repository is provided for reference purposes only and the solution may require modifications to fit your use case. Note that this solution is not part of the Agile Data Engine product. Please use at your own caution.**

**Contents:**
- ADF pipeline templates (.zip files)

# Prerequisites
- Follow [Microsoft documentation](https://docs.microsoft.com/en-us/azure/data-factory/) to create and configure an Azure Data Factory or use an existing one.
- Configure a [self-hosted integration runtime](https://docs.microsoft.com/en-us/azure/data-factory/create-self-hosted-integration-runtime?tabs=data-factory) that is hosted on a server with a static public IP address (or address range). Provide the IP addresses to the Agile Data Engine support team.
- Configure a Key Vault as [linked service](https://docs.microsoft.com/en-us/azure/data-factory/store-credentials-in-key-vault) to the Data Factory if one is not already configured. Add a secret with name **notify-api-key-secret** into the Key Vault. Set the value as the Notify API key secret you have received from Agile Data Engine support.

# Deployment
1. Open Azure Data Factory Studio.
2. In the Author menu, click Add new resource - Pipeline - Import from pipeline template.
3. Add pipeline template files:
    - [add_to_manifest.zip](./add_to_manifest.zip)
    - [notify_sources.zip](./notify_sources.zip)
    - [(notify_manifests.zip)](./notify_manifests.zip)

**Note that importing the notify_sources pipeline also imports its dependency pipeline notify_manifests.**

Select the Key Vault where you have stored notify-api-key-secret as the linked service.

# Configuration
## Integration runtime
Edit the settings of each **Web activity** to use self-hosted integration runtime mentioned in the prerequisites. 

## Pipeline parameters
### add_to_manifest
| Parameter  | Example value | Description |
| --- | --- | --- |
| notify_api_base_url | https://external-api.dev.datahub.s1234567.saas.agiledataengine.com/notify-api | Agile Data Engine tenant & environment specific Notify API base url. |
| notify_api_key | e8cbca20-0d78-11ed-861d-0242ac120002 | Environment specific Notify API key. Note that the pipelines fetch Notify API key secret (notify-api-key-secret) from Key Vault. |
| source_system_name | example_source | Source system name defined in the source entity, see [Agile Data Engine documentation](https://ade.document360.io/docs/notify-api). |
| source_entity_name | example_entity | Source entity name, see [Agile Data Engine documentation](https://ade.document360.io/docs/notify-api). |
| manifest_body | { "format": "CSV", "delim": "COMMA", "skiph": 1 } | Request body in the [create manifest](https://ade.document360.io/v1/docs/create-manifest) API call. Default value is *{}*, i.e. optionally you can leave this unset and configure [file format options](https://ade.document360.io/docs/opt-file-format-options) in the file load in Agile Data Engine. |
| manifest_entries_body | [ { "sourceFile": "https://myaccount.blob.core.windows.net/mycontainer/myblob.csv" } ] | Request body in the [create multiple entries](https://ade.document360.io/docs/create-multiple-entries) API call. Allows adding one or multiple entries to a manifest. |
| single_file_manifest | false | Boolean (true/false), default value is false. If set to true, the manifest is notified (closed) after the entry has been added. |

### notify_manifests
| Parameter  | Example value | Description |
| --- | --- | --- |
| notify_api_base_url | https://external-api.dev.datahub.s1234567.saas.agiledataengine.com/notify-api | Agile Data Engine tenant & environment specific Notify API base url. |
| source_system_name | example_source | Source system name defined in the source entity, see [Agile Data Engine documentation](https://ade.document360.io/docs/notify-api). |
| source_entity_name | example_entity | Source entity name, see [Agile Data Engine documentation](https://ade.document360.io/docs/notify-api). |
| notify_api_key | e8cbca20-0d78-11ed-861d-0242ac120002 | Environment specific Notify API key. Note that the pipelines fetch Notify API key secret (notify-api-key-secret) from Key Vault. |

### notify_sources
| Parameter  | Example value | Description |
| --- | --- | --- |
| notify_api_base_url | https://external-api.dev.datahub.s1234567.saas.agiledataengine.com/notify-api | Agile Data Engine tenant & environment specific Notify API base url. |
| notify_api_key | e8cbca20-0d78-11ed-861d-0242ac120002 | Environment specific Notify API key. Note that the pipelines fetch Notify API key secret (notify-api-key-secret) from Key Vault. |
| sources | [ {"source_system_name": "example_source", "source_entity_name": "example_entity"},</br>{"source_system_name": "example_source_n", "source_entity_name": "example_entity_n"} ] | Array of source objects containing values for attributes source_system_name and source_entity_name. Defines which sources will be notified by notify_manifests. |

## Triggers
Configure the following triggers as needed in your use case:
- Blob created triggers for each data source to add files to manifests.
- Schedule triggers to notify open manifests of given data sources.

See examples below.

Alternatively you can also trigger the pipelines from other ADF pipelines or outside processes that are e.g. extracting source data from source systems.

### Example: Blob created trigger for add_to_manifest

### Example: Schedule trigger for notify_sources

