
# Reference Implementation for FDO-FDOps
### !This repository is just for anonymized reviewing procedures.!
## Abbreviations:
- KIP: Kernel Information Profile
- TPM: Typed PID Maker
- PID: Persistent Identifier
- FDOps: FAIR Digital Operations
- FDO: FAIR Digital Object
- DOIP: Digital Object Interface Protocol
- DTR: Data Type Registry

## To reproduce the results, carry out the following steps:
- clone this repository
- navigate to the local folder and install all pip packages in the requirements.txt with 'pip install -r requirements.txt'. Ideally, set up an virtual environment first with 'python3 -m venv your_venv' (requires that you have python3 installed)
- start the flask applications using 'python3 tpm_adapter' for the TPM_Adapter module and 'python3 operation_apis' for the implementation of the web serice operations. They will run on your localhost.
- use the TPM service which is available at https://github.com/kit-data-manager/pit-service?tab=readme-ov-file by navigating to the repository and cloning also this repository.
- handling of the TPM service is described at the external repository, but you can start the service with default settings by navigating into the cloned folder and executing './gradlew run --args="--spring.config.location=config/application.properties"'
- the TPM service should run in your terminal. You can test the availability under http://localhost:8090/actuator/info. This localhost address is also the standard configuration for this software.
- back to the application, to create sandbox PIDs for the extended FDO records, execute the following shell script: 'ingest_fdos.sh'. Insert './ingest_fdos.sh chmod +x' and then execute './ingest_fdos.sh --post --dir PATH_TO_EXTENDED_RECORDS'
- the pids in the JSON record files should be automatically updated and the records registred at the local TPM instance.
- You can now request the TPM_Adapter's /doip endpoint. A reference script is provided at test_tpm.py that sequentially performs all operations as described below for all FDO-FDOps that were previously registered. 

## FDO records
We duplicated and extended the FDO records of the PIDs available at https://zenodo.org/records/7022736. The original records are stored under original_records. The extended records are available at extended_records and were only registered locally using  sandbox PIDs, not at Handle as the original ones which can be resolved at https://hdl.handle.net/ using the PIDs in the referenced JSON files. The extended records were created using additional Kernel Information Profiles (KIPs) and Attribute Types that are registered at the ePIC testing DTR (https://dtr-test.pidconsortium.net/), namely:
- KIPs: 
    - Web API Operation KIP: https://dtr-test.pidconsortium.net/#objects/21.T11148/ea4e93d06a10e15d9cdf
    - Raster Graphic Image Type KIP: https://dtr-test.pidconsortium.net/#objects/21.T11148/0e76292794888d4f1fa7
    - Schema-based Metadata Type Profile: https://dtr-test.pidconsortium.net/#objects/21.T11148/2c3cafa4db3f3e1e51b3
- Attribute Types:
    - Operation Name: https://dtr-test.pidconsortium.net/#objects/21.T11148/90ee0a5e9d4f8a668868
    - Required Input Type: https://dtr-test.pidconsortium.net/#objects/21.T11148/2694e4a7a5a00d44e62b
    - Expected Output Type: https://dtr-test.pidconsortium.net/#objects/21.T11148/f5c452794e258e83e4ad
    - HTTP Access Protocol: https://dtr-test.pidconsortium.net/#objects/21.T11148/a1fe3f60497302ae8b04
    - ORCiD Contact: https://dtr-test.pidconsortium.net/#objects/21.T11148/df4aab1aaf6c1cd41a70
    - Has Annotation: https://dtr-test.pidconsortium.net/#objects/21.T11148/210acdb71caa2e4c55cc
    - document MIME Type: https://dtr-test.pidconsortium.net/#objects/21.T11148/77e7da3320d24e21008b
    - schema Reference: https://dtr-test.pidconsortium.net/#objects/21.T11148/49330041ca5fddf9af92
    - container MIME-Type: https://dtr-test.pidconsortium.net/#objects/21.T11148/007fb75612859ed30aa0
These attributes are present on the first level of the information record and partly contain sub-types by inheritance and compoistion, all specified in the DTR. Lower-level attribute structures are represented as string values (expected by the TPM validator) but reflect a JSON structure that is reconstructed in the TPM_Adapter module. The new types can be validated by the TPM instance, configured with the ePIC DTR by default.

## The software package comprises three main modules:
- TPM_Adapter: provides a client interface by exposing an endpoint .../doip that implements the Digital Object Interface Protocol for HTTP clients. Typically uses the interface of the Typed PID Maker (TPM) Service avialable at https://github.com/kit-data-manager/pit-service.
Implements service specific operations for FDOs, namely:
    - LIST_OPS(): lists all operations that are implemented by the service
    - LIST_FDOS(): lists all FDOs using the TPM interface
    - GET_FDO(): retrieves the information record associated with the Persistent Identifier of a FDO using the TPM interface
    - LIST_FDOPS(): lists all FAIR Digital Operations using the TPM interface
    - OP*(): performs the operation described by a FDOps for an associated FDO using the Mapping_service and Executor
- Mapping_Service(): recieves the access protocol of a FDOps record (currently only HTTP supported) and the targeted FDO record. Transfers the parameters in the access protocol into (HTTP) requests and adds them to the workflow map. Values for parameter keys are either provided in the FDOps record directly (for standard values), are
 referenced via an attribute key in the target record from where they are mapped, or are passed by the client using the PID of the parameter key and are then directly inserted. In case multiple values per attribute key are present in the target FDO record and the asArray statement for the parameter states False,
 a separate request per value is added to the map. Otherwise, the values are mapped as one list. The module also considers recursive patterns when sub-operations are described and finally sorts the workflow map according to the record description which is theb returned.
- Executor(): Recieves the workflow map containing the requests to execute the described operation(s) (currently only HTTP requests for Web APIs supported). Returns the resuls to the TPM_Adapter which sends it to the client.

## DOIP request examples for FDO-FDOps
In the following, example HTTP/DOIP requests and responses are provided for each FDOps (including service and external operations) with an example target. All requests follow a unifrom structure where the operationId query argument specifies the identifier of the FAIR Digital Operation and targetId the FAIR Digtial Object or serviceID this operation is applied to. These results can be reproduced for all reference FDO records following the instructions in the earlier section.

- **LIST_OPS (for service ops)**: Lists all operations that are implemented by the service
    *Request*:
    ```
    POST .../doip?operationId=0.DOIP/Op.LIST_Ops&targetId=service
    Content-Type: appliaction/json;charset=utf-8
    ```
    *Response*:
    ```
    HTTP/1.1 200 OK
    Content-Type: application/json;charset=utf-8
    Content-Length: ...

    {
      "available operations": [
        "0.DOIP/Op.LIST_Ops",
        "0.DOIP/Op.LIST_FDOs",
        "0.DOIP/Op.GET_FDO",
        "0.DOIP/Op.*"
        ]
    }
- **LIST_FDOS**: lists all FDOs registered in the the configured TPM instance
    *Request*:
    ```
    POST .../doip?operationId=0.DOIP/Op.LIST_FDOs&targetId=service
    Content-Type: appliaction/json;charset=utf-8
    ```
    *Response*:
    ```
    {
      "available FDOs": [
        {
          "created": "2024-04-09T10:49:01.567743Z",
          "modified": "2024-04-09T10:49:01.567743Z",
          "pid": "sandboxed/fa798f80-a6db-4ce5-aab2-03188abd80aa"
        },
        {
          "created": "2024-04-09T10:49:01.567744Z",
          "modified": "2024-04-09T10:49:01.567744Z",
          "pid": "sandboxed/abbb8d8a-5ca0-4bc2-9386-47a2aae94244"
        },
        ...
        ]
    }
    ```
- **GET_FDO**: retrieves the information record associated with the Persistent Identifier of a FDO using the TPM interface
    *Request*:
    ```
    POST .../doip?operationId=0.DOIP/Op.GET_FDO&targetId=sandboxed/PID1
    Content-Type: appliaction/json;charset=utf-8
    ```
    *Response*:
    ```
    {
    "pid": "sandboxed/8182a503-fb72-4167-a9d3-c3813cb270bc",
    "entries": {
      "21.T11148/076759916209e5d62bd5": [
        {
          "key": "21.T11148/076759916209e5d62bd5",
          "value": "21.T11148/2c3cafa4db3f3e1e51b3"
        }
      ],
      "21.T11148/1c699a5d1b4ad3ba4956": [
        {
          "key": "21.T11148/1c699a5d1b4ad3ba4956",
          "value": "21.11152/3052e4f1-4dd6-4c40-9a16-0a6b5f409e4a"
        }
      ],
      "21.T11148/2f314c8fe5fb6a0063a8": [
        {
          "key": "21.T11148/2f314c8fe5fb6a0063a8",
          "value": "https://creativecommons.org/licenses/by/4.0/"
        }
      ],
      ...
      }
    }
    ```
- **LIST_OPS (for a FDO)**: lists all FAIR Digital Operations associated with a FDO and registered at the TPM instance
    *Request*:
    ```
    POST .../doip?operationId=0.DOIP/Op.LIST_Ops&targetId=sandboxed/PID1
    Content-Type: appliaction/json;charset=utf-8
    ```
    *Response*:
    ```
    HTTP/1.1 200 OK
    Content-Type: application/json;charset=utf-8
    Content-Length: ...
    
    {
      "available FDOps": [
        {
          "name": "EVALUATE_LICENSE",
          "pid": "sandboxed/49d302da-b977-42eb-8643-e4d3a391bab3"
        },
        {
          "name": "FIND_METADATA",
          "pid": "sandboxed/547c8a14-17aa-45dc-8ac5-7642cbb5c312"
        },
        ...
      ]
    }
    ```
- **EVALUATE_LICENSE**: Evaluates the license given in a FDO record based on the criteria of open- or non-open access (using regex for input argument), with "21.T11148/916ca3badfa68b06870c" as the request argument for the required value input by the client for the query parameter. 
    *Request*:
    ```
    POST .../doip?operationId=sandboxed/PID1&targetId=sandboxed/PID2
    Content-Type: appliaction/json;charset=utf-8
    
    {
    "21.T11148/916ca3badfa68b06870c": "open source",
    }
    ```
    *Response*:
    ```
    HTTP/1.1 200 OK
    Content-Type: application/json;charset=utf-8
    Content-Length: 4

    true
    ```
- **GET_ORCID**: Retrieves the profile of an ORCiD using the official ORCiD Web API.
    *Request*:
    ```
    POST …/doip?operationId=sandboxed/PID1&targetId=sandboxed/PID2
    Content-Type: application/json;charset=utf-8
    ```
    *Response*:

    ```HTTP/1.1 200 OK
    Content-Type: application/json;charset=utf-8
    Content-Length: ...

    {
        "orcidId": "0000-0002-9082-9095",
        "personalInformation": {
            "name": {
            "givenNames": ...
        ...
    }
    ```
- **FIND_METADATA**: Retrieves the information record of FDOs that are referenced as metadata objects in the target FDO record. 
    *Request*:
    ```
    POST .../doip?operationId=sandboxed/PID1&targetId=sandboxed/PID2
    Content-Type: appliaction/json;charset=utf-8
    ```
    *Response*:
    ```
    HTTP/1.1 200 OK
    Content-Type: application/json;charset=utf-8
    Content-Length: ...

    [
        {
            "pid": "21.11152/09cb76fc-b8cb-4116-a22a-68c5bdfa77b0",
            "entries": {
                "21.T11148/b8457812905b83046284": [
                    {
                        "key": "21.T11148/b8457812905b83046284",
                        "value": "https://zenodo.org/record/6517768/files/Flug1_100_stac_spec.json?download=1"
                    }
                ],
            ...
        },
        {
            "pid": "21.11152/7b58b3b5-75eb-4417-ac4d-abe025e159f6", 
            ...
        }
    ]
    ```
- **FIND_ANNOTATION**: Retrieves the information record of FDOs that are referenced as annotation objects in the target FDO record.
    *Request*:
    ```
    POST .../doip?operationId=sandboxed/PID1&targetId=sandboxed/PID2
    Content-Type: appliaction/json;charset=utf-8
    ```
    *Response*:
    ```
    HTTP/1.1 200 OK
    Content-Type: application/json;charset=utf-8
    Content-Length: ...

    [
        {
            "pid": "21.11152/6ea60288-d895-414e-80c0-26c9fdd662b2",
            "entries": {
                 "21.T11148/b8457812905b83046284": [
                    {
                        "key": "21.T11148/b8457812905b83046284",
                        "value": "https://zenodo.org/record/6517768/files/Flug1_100-104Media_coco.json?download=1"
                    }
                ],
            ...
        }
    ]
    ```
- **VALIDATE_SCHEMA**: Recieves a JSON metadata document and corresponding JSON schema reference to validate the compliance between document and schema.
    *Request*:
    ```
    POST .../doip?operationId=sandboxed/PID1&targetId=sandboxed/PID2
    Content-Type: appliaction/json;charset=utf-8
    ```
    *Response*:
    ```
    HTTP/1.1 200 OK
    Content-Type: application/json;charset=utf-8
    Content-Length: ...

    true
    ```
- **EXPORT_IMAGES_TO_PNG**: Recieves a container file containing image data and returns a zip folder with the images in png format.
    *Request*:
    ```
    POST .../doip?operationId=sandbox/PID1&targetId=sandbox/PID2
    Content-Type: appliaction/json;charset=utf-8
    ```
    *Response*:
    ```
    HTTP/1.1 200 OK
    Content-Type: application/zip
    Content-Disposition: attachment; filename="output.zip"
    Content-Length: ...
    ETag: "abcd1234"

    <binary data>
    ```