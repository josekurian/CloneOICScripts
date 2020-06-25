# OIC Environment Clone Scripts
[This image][DockerHub] provide a simplified way of running the import and export instance REST commands in OIC.
They also provide support for creating and deleting OCI instances using the OCI command line.
See the [OIC documentation] for details on import / export.
See the [OCI CLI OIC documentation] for details on creating and deleting OIC instances vi OCI CLI.
To use the create and delete scripts it is necessary to have the OCI CLI tools setup, see the [OCI documentation] for details.
If you are going to use the create script then it relies on [jq] which must be available on the system running the scripts.
The test scripts also use [jq].
See the [OIC documentation] for details.

This image is built on top of the [Python:3-slim] image.
A [github project][GitHub] contains the build instructions if a customized image is required.


## Running the Image
To run the image use the command

    docker run -v $HOME/.oci:/root/.oci:ro -v <CONFIG_DIRECTORY>:/oic/configs:ro --name CloneOICScripts -it --rm antonyjreynolds/cloneoic:0.9

This assumes the OCI config file and keys files are in $HOME/.oci and that config files with default values are in <CONFIG_DIRECTORY>.
The config files can be copied from oic_env.sh and edited to set appropriate values.
For example there may be a config file melbourne_oic_config.sh that can be set by running

    . configs/melbourne_oic_config.sh

Another file may be called ashburn_oic_config.sh and that can be selected by running

    . configs/ashburn_oic_config.sh


## Scripts

The following scripts are provided:

| Script | Description |
| ------: | ----------- |
| | CONFIG |
| [env.sh] | Used to query for parameters for scripts, also provides some common functions to retrieve and construct credentials and URLs.|
| [oic_env.sh] | Blank environment variables |
| | IMPORT / EXPORT |
| [export.sh] | Runs the OIC export environment command. |
| [exportStatus.sh] | Get the status of the export command. Takes a single parameter, the jobId returned from the export command. |
| [listBucket.sh] | List all the files in the object storage bucket. |
| [copyExport.sh] | Copy file from STORAGE bucket to REPLICA_STORAGE bucket. Takes a single parameter, the filename of the archive generated by the export command. This requires OCI CLI to have been set up. |  
| [objectStatus.sh] | Get the status of the copy . |  
| [import.sh] | Runs the import command. Takes a single parameter, the filename of the archive generated by the export command. An optional second parameter allows overriding the IMPORT_STATE. |
| [importStatus.sh] | Get the status of the import command. Takes a single parameter, the jobId returned from the import command. |
| | CREATE / DELETE |
| [createIntegration.sh] | Create a new OIC instance |
| [deleteIntegration.sh] | Delete an OIC instance |
| [getIntegration.sh] | Get OIC instance details |
| [getWorkRequest.sh] | Get status of work request |
| [listCompartments.sh] | List all OCI compartments |
| [listIntegrations.sh] | List instances in compartment |
| [listRegions.sh] | List OCI Regions |
| [testCreateDelete.sh] | Tests for creating and deleting OIC scripts |
| [testExportImport.sh] | Tests for exporting and importing OIC scripts |
| | MANAGE USERS FOR OIC |
| [getApplication.sh] | Get IDCS application associated with OIC instance |



## [env.sh] File
If they are not provided then the scripts in [env.sh] will prompt for them.
Passwords/Secrets are not echoed to the screen when entered.

### Environment Variables
The following environment variables are set in the [env.sh] file.

| Property | Meaning | Used By |
| :------: | ------- | ------- |
| STORAGE_REGION | OCI Region | [import.sh], [export.sh], [listBucket.sh], [copyExport.sh] |
| STORAGE_TENANCY | OCI Tenancy | [import.sh], [export.sh], [listBucket.sh], [copyExport.sh] |
| STORAGE_BUCKET | OCI Object Storage Bucket | [import.sh], [export.sh], [listBucket.sh], [copyExport.sh] |
| STORAGE_USER | OCI Object Storage User (prefix with oracleidentitycloudservice/ if using IDCS federated user | [import.sh], [export.sh], [listBucket.sh], [copyExport.sh] |
| STORAGE_PASSWORD | [OCI Auth Token] | [import.sh], [export.sh], [listBucket.sh], [copyExport.sh] |
| REPLICA_STORAGE_REGION | OCI Region | [copyExport.sh] |
| REPLICA_STORAGE_TENANCY | OCI Tenancy | [copyExport.sh] |
| REPLICA_STORAGE_BUCKET | OCI Object Storage Bucket | [copyExport.sh] |
| REPLICA_STORAGE_USER | OCI Object Storage User (prefix with oracleidentitycloudservice/ if using IDCS federated user | [copyExport.sh] |
| REPLICA_STORAGE_PASSWORD | [OCI Auth Token] | [copyExport.sh] |
| SRC_OIC_HOST | Base URL of OIC Instance to be exported  Should be of the form https://oicinstance.integration.ocp.oraclecloud.com. | [export.sh] [exportStatus.sh] |
| SRC_OIC_USERNAME | IDCS Username with Admin Privileges | [export.sh] [exportStatus.sh] |
| SRC_OIC_PASSWORD | IDCS Password | [export.sh] [exportStatus.sh] |
| TGT_OIC_HOST | Base URL of OIC Instance to receive import  Should be of the form https://oicinstance.integration.ocp.oraclecloud.com. | [export.sh] [exportStatus.sh] |
| TGT_OIC_USERNAME | IDCS Username with Admin Privileges | [export.sh] [exportStatus.sh] |
| TGT_OIC_PASSWORD | IDCS Password | [import.sh] [importStatus.sh] |
| IMPORT_STATE / Status of Imported integrations.  Valid Values are :  | [import.sh] |
| CURL_FLAGS | Flags passed to curl command | [import.sh], [importStatus.sh], [listBucket.sh], [export.sh], [exportStatus.sh] |
| INTEGRATION_ID | OCID of Integration Instance | [getIntegration.sh], [deleteIntegration.sh], [getIntegration.sh], [getWorkRequest.sh], [getApplication.sh] |
| COMPARTMENT_ID | OCID of Compartment | [createIntegration.sh], [listIntegrations.sh] |
| REGION | OCI Region | [createIntegration.sh], [deleteIntegration.sh], [getIntegration.sh], [getWorkRequest.sh], [listIntegrations.sh], [getApplication.sh] |
| DISPLAY_NAME | Name of New OIC Instance | [createIntegration.sh] |
| IS_BYOL | BYOL Flag for OIC (true or false) | [createIntegration.sh] |
| MESSAGE_PACKS | Number of Message Packs for OIC | [createIntegration.sh] |
| TYPE | OIC Edition (ENTERPRISE or STANDARD) | [createIntegration.sh] |
| IDCS_URL | IDCS host (https://idcs-hostname) | [createIntegration.sh], [getApplication.sh] |
| IDCS_CLIENT_ID | OAuth Client ID | [createIntegration.sh], [getApplication.sh] |
| IDCS_CLIENT_SECRET | OAuth Client Secret | [createIntegration.sh], [getApplication.sh] |
| IDCS_USERNAME | IDCS Username | [createIntegration.sh] |
| IDCS_PASSWORD | IDCS Password | [createIntegration.sh] |
| WORK_REQUEST_ID | Work Request ID from Delete & Create Operations | [getWorkRequest.sh] |


#### CURL_FLAGS Explained
The CURL_FLAGS are applied to all REST calls and allow additional information to be output.  Flags that have been verified to work are:
* -v
    * Verbose output, including SSL handshake
* -k
    * Ignore certification validation errors - should not be necessary
* -w HTTP_STATUS=%{http_code}
    * Output the HTTP Status returned from the command.  Useful in debugging.

## Example Exporting from an OIC Instance
Run the export command:
```shell script
export.sh
```
> {"jobId":"142040","location":"https://swiftobjectstorage.eu-frankfurt-1.oraclecloud.com/v1/xxxxxx/migrationBucket","status":"NOT_STARTED"}

Check the status of the export command:
```shell script
exportStatus.sh 142040
```
> {"jobId":"142040","jobType":"EXPORT","includeSecurityArtifacts":true,"overallStatus":"RUNNING","startTime":"Thu May 14 21:28:31 UTC 2020","components":[{"name":"Integration","status":"NOT_STARTED"},{"name":"Process","status":"NOT_STARTED"}]}

Check it again:
```shell script
exportStatus.sh 142040
```
> {"jobId":"142040","jobType":"EXPORT","archiveName":"Local_Suite_Instance-142040.zip","includeSecurityArtifacts":true,"overallStatus":"COMPLETED","startTime":"Thu May 14 21:28:31 UTC 2020","endTime":"Thu May 14 21:55:57 UTC 2020","components":[{"name":"Integration","status":"COMPLETED","percentage":100},{"name":"Process","status":"COMPLETED","percentage":100}]}

List files in OCI Bucket to make sure that it is there:
```shell script
listBucket.sh
```
> [{"name":"Local_Suite_Instance-142040.zip","hash":"8c1c0866ff0e7ff78441bc45cded6dfc","bytes":29688003,"last_modified":"2020-05-14T21:56:02.716000","content_type":null}]

## Example Importing to an OIC Instance
Run the import command:
```shell script
import.sh Local_Suite_Instance-142040.zip
```
> {"jobId":"1231","status":"NOT_STARTED"}

Check the status of the import command:
```shell script
importStatus.sh 1231
```
> {"jobId":"1231","jobType":"IMPORT","includeSecurityArtifacts":true,"overallStatus":"RUNNING","startTime":"Thu May 14 22:18:02 UTC 2020","mode":"ImportOnly","importScheduleParams":false,"startSchedules":false,"components":[{"name":"Integration","status":"NOT_STARTED"},{"name":"Process","status":"NOT_STARTED"}]}

Check it again:
```shell script
importStatus.sh 1231
```
> {"jobId":"1231","jobType":"IMPORT","includeSecurityArtifacts":true,"overallStatus":"RUNNING","startTime":"Thu May 14 22:18:02 UTC 2020","mode":"ImportOnly","importScheduleParams":false,"startSchedules":false,"components":[{"name":"Integration","status":"RUNNING"},{"name":"Process","status":"CANCELLED"}]}

## Example Moving Archive Between Buckets
Run the copy command:
```shell script
copyExport.sh Local_Suite_Instance-329120.zip
```
>   % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
>                                  Dload  Upload   Total   Spent    Left  Speed
> 100 7921k  100 7921k    0     0  1214k      0  0:00:06  0:00:06 --:--:-- 1550k
> 
> HTTP_STATUS=200
> 
> HTTP_STATUS=201

## Example Creating an OIC Instance
Run the create command:
```shell script
createIntegration.sh
```
> Action completed. Waiting until the work request has entered state: ('ACCEPTED',)  
>{  
>  "data": {  
>    "compartment-id": "ocid1.compartment.oc1..aaaaaaaackq6ccfxgzchfka57oifujjstupc2nftfgpi3ytpufuwrmijr6ea",  
>    "id": "ocid1.integrationworkrequest.oc1.ap-melbourne-1.amaaaaaajfhkgfyan2mcuu2zhlb5ircql4orxpmbkzf2wkuujqzojpdhtsda",  
>    "operation-type": "CREATE_INTEGRATION_INSTANCE",  
>    "percent-complete": 0.0,  
>    "resources": [  
>      {  
>        "action-type": "IN_PROGRESS",  
>        "entity-type": "integrationInstance",  
>        "entity-uri": "/integrationInstances/ocid1.integrationinstance.oc1.ap-melbourne-1.amaaaaaajfhkgfya3mvzrzq7zpls23oqdphwa5yxsp2n4kxjechcyvelhdlq",  
>        "identifier": "ocid1.integrationinstance.oc1.ap-melbourne-1.amaaaaaajfhkgfya3mvzrzq7zpls23oqdphwa5yxsp2n4kxjechcyvelhdlq"  
>      }  
>    ],  
>    "status": "ACCEPTED",  
>    "time-accepted": "2020-06-19T06:47:36.505000+00:00",  
>    "time-finished": null,  
>    "time-started": null  
>  },  
>  "etag": "83d77c2e7ccc4f79251b2dfd1e9edb169d98634f5ab7a25bc41c0e77e8a6c347--gzip"
>}

Check the status of the create command:
```shell script
getWorkRequest.sh ocid1.integrationworkrequest.oc1.ap-melbourne-1.amaaaaaajfhkgfyan2mcuu2zhlb5ircql4orxpmbkzf2wkuujqzojpdhtsda  
```
> {  
>  "data": {  
>    "compartment-id": "ocid1.compartment.oc1..aaaaaaaackq6ccfxgzchfka57oifujjstupc2nftfgpi3ytpufuwrmijr6ea",  
>    "id": "ocid1.integrationworkrequest.oc1.ap-melbourne-1.amaaaaaajfhkgfyan2mcuu2zhlb5ircql4orxpmbkzf2wkuujqzojpdhtsda",  
>    "operation-type": "CREATE_INTEGRATION_INSTANCE",  
>    "percent-complete": 53.0,  
>    "resources": [  
>      {  
>        "action-type": "IN_PROGRESS",  
>        "entity-type": "integrationInstance",  
>        "entity-uri": "/integrationInstances/ocid1.integrationinstance.oc1.ap-melbourne-1.amaaaaaajfhkgfya3mvzrzq7zpls23oqdphwa5yxsp2n4kxjechcyvelhdlq",  
>        "identifier": "ocid1.integrationinstance.oc1.ap-melbourne-1.amaaaaaajfhkgfya3mvzrzq7zpls23oqdphwa5yxsp2n4kxjechcyvelhdlq"  
>      }  
>    ],  
>    "status": "IN_PROGRESS",  
>    "time-accepted": "2020-06-19T06:47:36.505000+00:00",  
>    "time-finished": null,  
>    "time-started": "2020-06-19T06:47:44.465000+00:00"  
>  },  
>  "etag": "a3dd2623edf7efc8d55596169134c5bd274050bf9d11fad4d068bbce10925374--gzip"  
>}

Check it again:
```shell script
getWorkRequest.sh ocid1.integrationworkrequest.oc1.ap-melbourne-1.amaaaaaajfhkgfyan2mcuu2zhlb5ircql4orxpmbkzf2wkuujqzojpdhtsda  
```
> {  
>  "data": {  
>    "compartment-id": "ocid1.compartment.oc1..aaaaaaaackq6ccfxgzchfka57oifujjstupc2nftfgpi3ytpufuwrmijr6ea",  
>    "id": "ocid1.integrationworkrequest.oc1.ap-melbourne-1.amaaaaaajfhkgfyan2mcuu2zhlb5ircql4orxpmbkzf2wkuujqzojpdhtsda",  
>    "operation-type": "CREATE_INTEGRATION_INSTANCE",  
>    "percent-complete": 100.0,  
>    "resources": [  
>      {  
>        "action-type": "CREATED",  
>        "entity-type": "integrationInstance",  
>        "entity-uri": "/integrationInstances/ocid1.integrationinstance.oc1.ap-melbourne-1.amaaaaaajfhkgfya3mvzrzq7zpls23oqdphwa5yxsp2n4kxjechcyvelhdlq",  
>        "identifier": "ocid1.integrationinstance.oc1.ap-melbourne-1.amaaaaaajfhkgfya3mvzrzq7zpls23oqdphwa5yxsp2n4kxjechcyvelhdlq"  
>      }  
>    ],  
>    "status": "IN_PROGRESS",  
>    "time-accepted": "2020-06-19T06:47:36.505000+00:00",  
>    "time-finished": "2020-06-19T06:52:40.527000+00:00",  
>    "time-started": "2020-06-19T06:47:44.465000+00:00"  
>  },  
>  "etag": "a3dd2623edf7efc8d55596169134c5bd274050bf9d11fad4d068bbce10925374--gzip"    
>}

List integrations in OCI Compartment to make sure that it is there:
```shell script
listIntegrations.sh
```
> +-----------+-----------------------------------------------------------------------------------------------------------+--------------------------------------------------------------------+
> | Name      | OCID                                                                                                      | URL                                                                |
> +-----------+-----------------------------------------------------------------------------------------------------------+--------------------------------------------------------------------+
> | GoldImage | ocid1.integrationinstance.oc1.ap-melbourne-1.amaaaaaajfhkgfya3mvzrzq7zpls23oqdphwa5yxsp2n4kxjechcyvelhdlq | https://goldimage-oicpm-me.integration.ocp.oraclecloud.com/ic/home |
> +-----------+-----------------------------------------------------------------------------------------------------------+--------------------------------------------------------------------+

## Example Deleting an OIC Instance
Run the delete command:
```shell script
deleteIntegration.sh ocid1.integrationinstance.oc1.ap-melbourne-1.amaaaaaajfhkgfya3mvzrzq7zpls23oqdphwa5yxsp2n4kxjechcyvelhdlq
```
> Action completed. Waiting until the work request has entered state: ('ACCEPTED',)  
>{  
>  "data": {  
>    "compartment-id": "ocid1.compartment.oc1..aaaaaaaackq6ccfxgzchfka57oifujjstupc2nftfgpi3ytpufuwrmijr6ea",  
>    "id": "ocid1.integrationworkrequest.oc1.ap-melbourne-1.amaaaaaajfhkgfyamnc34oexwqr6jzopwkz3r4zgrntahdao5tsnsdyoz2ca",  
>    "operation-type": "DELETE_INTEGRATION_INSTANCE",  
>    "percent-complete": 0.0,  
>    "resources": [  
>      {  
>        "action-type": "IN_PROGRESS",  
>        "entity-type": "integrationInstance",  
>        "entity-uri": "/integrationInstances/ocid1.integrationinstance.oc1.ap-melbourne-1.amaaaaaajfhkgfya3mvzrzq7zpls23oqdphwa5yxsp2n4kxjechcyvelhdlq",  
>        "identifier": "ocid1.integrationinstance.oc1.ap-melbourne-1.amaaaaaajfhkgfya3mvzrzq7zpls23oqdphwa5yxsp2n4kxjechcyvelhdlq"  
>      }  
>    ],  
>    "status": "ACCEPTED",  
>    "time-accepted": "2020-06-19T07:10:52.803000+00:00",  
>    "time-finished": null,  
>    "time-started": null  
>  },  
>  "etag": "50f8c6be644b026c58957de6c0fb611a4d0a859bf1cee197e49f48e878223568--gzip"
>}

Check the status of the delete command:
```shell script
getWorkRequest.sh ocid1.integrationworkrequest.oc1.ap-melbourne-1.amaaaaaajfhkgfyamnc34oexwqr6jzopwkz3r4zgrntahdao5tsnsdyoz2ca  
```
>{  
>  "data": {  
>    "compartment-id": "ocid1.compartment.oc1..aaaaaaaackq6ccfxgzchfka57oifujjstupc2nftfgpi3ytpufuwrmijr6ea",  
>    "id": "ocid1.integrationworkrequest.oc1.ap-melbourne-1.amaaaaaajfhkgfyamnc34oexwqr6jzopwkz3r4zgrntahdao5tsnsdyoz2ca",  
>    "operation-type": "DELETE_INTEGRATION_INSTANCE",  
>    "percent-complete": 10.0,  
>    "resources": [  
>      {  
>        "action-type": "IN_PROGRESS",  
>        "entity-type": "integrationInstance",  
>        "entity-uri": "/integrationInstances/ocid1.integrationinstance.oc1.ap-melbourne-1.amaaaaaajfhkgfya3mvzrzq7zpls23oqdphwa5yxsp2n4kxjechcyvelhdlq",  
>        "identifier": "ocid1.integrationinstance.oc1.ap-melbourne-1.amaaaaaajfhkgfya3mvzrzq7zpls23oqdphwa5yxsp2n4kxjechcyvelhdlq"  
>      }  
>    ],  
>    "status": "IN_PROGRESS",  
>    "time-accepted": "2020-06-19T07:10:52.803000+00:00",  
>    "time-finished": null,  
>    "time-started": "2020-06-19T07:11:04.521000+00:00"  
>  },  
>  "etag": "7cf567a1e608a48cb66bf75b75c7a07f11774751c8f13e357b30592b1fb3f7db--gzip"
>}

Check it again:
```shell script
getWorkRequest.sh ocid1.integrationworkrequest.oc1.ap-melbourne-1.amaaaaaajfhkgfyamnc34oexwqr6jzopwkz3r4zgrntahdao5tsnsdyoz2ca  
```
>{  
>  "data": {  
>    "compartment-id": "ocid1.compartment.oc1..aaaaaaaackq6ccfxgzchfka57oifujjstupc2nftfgpi3ytpufuwrmijr6ea",  
>    "id": "ocid1.integrationworkrequest.oc1.ap-melbourne-1.amaaaaaajfhkgfyamnc34oexwqr6jzopwkz3r4zgrntahdao5tsnsdyoz2ca",  
>    "operation-type": "DELETE_INTEGRATION_INSTANCE",  
>    "percent-complete": 100.0,  
>    "resources": [  
>      {  
>        "action-type": "DELETED",  
>        "entity-type": "integrationInstance",  
>        "entity-uri": "/integrationInstances/ocid1.integrationinstance.oc1.ap-melbourne-1.amaaaaaajfhkgfya3mvzrzq7zpls23oqdphwa5yxsp2n4kxjechcyvelhdlq",  
>        "identifier": "ocid1.integrationinstance.oc1.ap-melbourne-1.amaaaaaajfhkgfya3mvzrzq7zpls23oqdphwa5yxsp2n4kxjechcyvelhdlq"  
>      }  
>    ],  
>    "status": "SUCCEEDED",  
>    "time-accepted": "2020-06-19T07:10:52.803000+00:00",  
>    "time-finished": "2020-06-19T07:19:33.508000+00:00",  
>    "time-started": "2020-06-19T07:11:04.521000+00:00"  
>  },  
>  "etag": "03f1e28c46949c0f7418896e4fce116ce625df6223901d5ac840a986c1b97e0a--gzip"
>}

List integrations in OCI Compartment to make sure that it is no longer there:
```shell script
listIntegrations.sh
```
Empty Response

[OIC Documentation]: https://docs.oracle.com/en/cloud/paas/integration-cloud/integration-cloud-auton/export-integration-and-process-design-time-metadata-instances.html
[OCI Auth Token]: https://docs.cloud.oracle.com/en-us/iaas/Content/Identity/Tasks/managingcredentials.htm#Working
[OCI documentation]: https://docs.cloud.oracle.com/en-us/iaas/Content/API/SDKDocs/cliinstall.htm
[OCI CLI OIC documentation]: https://docs.cloud.oracle.com/en-us/iaas/tools/oci-cli/latest/oci_cli_docs/cmdref/integration.html
[jq]: https://stedolan.github.io/jq/download/
[env.sh]: scripts/env.sh
[oic_env.sh]: scripts/oic_env.sh
[export.sh]: scripts/export.sh
[exportStatus.sh]: scripts/exportStatus.sh
[import.sh]: scripts/import.sh
[importStatus.sh]: scripts/importStatus.sh
[listBucket.sh]: scripts/listBucket.sh
[copyExport.sh]: scripts/copyExport.sh
[copyStatus.sh]: scripts/copyStatus.sh
[objectStatus.sh]: scripts/objectStatus.sh
[createIntegration.sh]: scripts/createIntegration.sh
[deleteIntegration.sh]: scripts/deleteIntegration.sh
[getIntegration.sh]: scripts/getIntegration.sh
[getWorkRequest.sh]: scripts/getWorkRequest.sh
[listCompartments.sh]: scripts/listCompartments.sh
[listIntegrations.sh]: scripts/listIntegrations.sh
[listRegions.sh]: scripts/listRegions.sh
[getApplication.sh]: scripts/getApplication.sh
[testCreateDelete.sh]: testCreateDelete.sh
[testExportImport.sh]: testExportImport.sh

[DockerHub]: https://hub.docker.com/r/antonyjreynolds/cloneoic
[GitHub]: https://github.com/AntonyJR/CloneOICScripts

[Python:3-slim]: https://hub.docker.com/_/python
