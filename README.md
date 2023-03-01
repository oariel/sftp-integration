# SFTP Export Integration Guide
## Setting Up SFTP Export Integration

The SFTP integration allows licensed customers to receive daily, per-facility raw predictions in json format. The payload also includes a base URL for downloading the PDF report that requires an additional secret (UI TBD).

In order to set up the integration, navigate to the `Integrations` section of MySAIVA located at [app.saiva.ai](https://app.saiva.ai "app.saiva.ai"). 
> Note: You must be an Org Administrator in order to access this function:

![Integrations](https://raw.githubusercontent.com/saivaai/sftp-integration/dev/images/Integrations.png "Integrations")

Choose SFTP Export and click on `Connect` to enter the setup screen.

Select the facilities and the quality measures to export. For authentication, paste your public key into the Public Key placeholder:

![Connect](https://raw.githubusercontent.com/saivaai/sftp-integration/dev/images/sftp-connect.png "Connect")

Click on `Connect` to save and activate your connection. It typically takes about a minute to setup the new connection depending on system load.

![Test](https://raw.githubusercontent.com/saivaai/sftp-integration/dev/images/sftp-test.png "Test")

- Verify that you can log in and view your SFTP directory using the following command line, or using your SFTP client software of choice:  
`sftp -i path_to_private_key_file username@hostname`    
- Click `Test` to test your integration. When the test completes successfully you should see sample raw facility prediction files for the most recent day in the output directory
- Use the `Enable/Disable` switch to temporarily disable your integration or to enable it
- Click `Disconnect` to delete this integration. 
> Notes: 
> - When disconnecting, all exported files will be lost. This operation is irreversible. 
> - Test will produce output files on if raw prediction data aleady exists for the selected facilities


## Availability of Raw Data
Raw predictions files for a any given day will be available by **8:00a Eastern Standard Time**

## File naming Convention
The file naming convention is as follows:

`[report type]-[client id]-[facility id]-[report date YYYYMMDD].json`

- report type - For RTH daily prediction exports use ‘rth’
- client id - The SAIVA client ID
- facility id - The EHR facility ID
- report date - report date YYYYMMDD

i.e. rth-sample-3-20231801.json

## Permissions
Consumers of raw prediction files will have write/delete access permissions to the SFTP directory. They may choose to move files around, rename, create subdirectories as they see fit. SAIVA will not perform any file manipulation other than placing the files daily at the root directory  

> Notes: If multiple consumers are reading the files, please make sure to coordinate between consumers how files are handled after reading is complete in order not to create any conflicts.

## Payload Schema and Examples
Raw predictions payload is json formatted according to the following schema:

```json
{
    "$schema": "https://github.com/saivaai/sftp-integration/tree/dev/schema",
    "$ref": "#/definitions/FacilityRiskReport",
    "definitions": {
        "FacilityRiskReport": {
            "type": "object",
            "additionalProperties": false,
            "properties": {
                "version": {
                    "type": "string"
                },
                "qm_type": {
                    "type": "string"
                },
                "client": {
                    "type": "string"
                },
                "facility_name": {
                    "type": "string"
                },
                "facility_id": {
                    "type": "integer"
                },
                "report_date": {
                    "type": "string"
                },
                "risk_list_length": {
                    "type": "integer"
                },
                "report_download_url" : {
                  "type": [ "string", "null" ]
                },
                "risk_list": {
                    "type": "array",
                    "items": {
                        "$ref": "#/definitions/RiskList"
                    }
                }
            },
            "required": [
                "client",
                "facility_id",
                "facility_name",
                "qm_type",
                "report_date",
                "risk_list",
                "risk_list_length",
                "version"
            ],
            "title": "FacilityRiskReport"
        },
        "RiskList": {
            "type": "object",
            "additionalProperties": false,
            "properties": {
                "patient_id": {
                    "type": "integer"
                },
                "patient_mrn": {
                    "type": "string"
                },
                "rank_today": {
                    "type": "integer"
                },
                "rank_yesterday": {
                    "type": [ "integer", "null" ]
                },
                "days_ranked": {
                    "type": "integer"
                },
                "payer": {
                    "type": [ "string", "null" ]
                },
                "primary_physician": {
                    "type": [ "string", "null" ]
                },
                "initial_admission_date": {
                    "type": "string"
                },
                "last_transfer_date": {
                    "type": [ "string", "null" ]
                },
                "risk_rank_reasons": {
                    "type": "array",
                    "items": {
                        "type": "string"
                    }
                }
            },
            "required": [
                "days_ranked",
                "initial_admission_date",
                "last_transfer_date",
                "patient_id",
                "patient_mrn",
                "payer",
                "primary_physician",
                "rank_today",
                "rank_yesterday",
                "risk_rank_reasons"
            ],
            "title": "RiskList"
        }
    }
}
```

Example file payload (QM = RTH, risk list length = 3):
```json
{
    "version": "v1",
    "qm_type": "RTH",
    "client": "Sample",
    "facility_name": "Lincoln Center Transitional Rehab",
    "facility_id": 3,
    "report_date": "25/01/2023",
    "report_download_url": "https://api.saiva.ai/v2/report/sample/rth/1/20230214",
    "risk_list_length": 3,
    "risk_list": [
        {
            "patient_id": 79438,
            "patient_mrn": "7188",
            "rank_today": 1,
            "rank_yesterday": null,
            "days_ranked": 3,
            "payer": "Medicaid Pending",
            "primary_physician": "SANJAY PATEL",
            "initial_admission_date": "10/26/2022",
            "last_transfer_date": "01/19/2023",
            "risk_rank_reasons": [
                "5 days since last admission",
                "Diagnosis of D631 : ANEMIA IN CHRONIC KIDNEY DISEASE",
                "Diagnosis of H409 : UNSPECIFIED GLAUCOMA",
                "Diagnosis of M6281 : MUSCLE WEAKNESS (GENERALIZED)"
            ]
        },
        {
            "patient_id": 79439,
            "patient_mrn": "7211",
            "rank_today": 2,
            "rank_yesterday": 1,
            "days_ranked": 5,
            "payer": "EVER SECURE HOR/AARP MED COMP",
            "primary_physician": "SANJAY PATEL",
            "initial_admission_date": "11/18/2022",
            "last_transfer_date": "12/01/2022",
            "risk_rank_reasons": [
                "Enoxaparin Sodium Injection Solution (Heparins And Heparinoid-Like Agents) was ordered on 01/24/2023",
                "Resident with a diagnosis of CHF has no weight readings in the past 8 days. Last recorded weight of 118.5 lbs on 01/11/2023",
                "1 day since last admission",
                "Diagnostic order in last 30 days. Last Order: X Ray of sacrum to r/o OM . 27 days ago (on 12/29/2022)",
                "Diagnosis of D631 : ANEMIA IN CHRONIC KIDNEY DISEASE",
                "Albuterol Sulfate Inhalation Aerosol ordered 1 day ago (on 01/24/2023)"

            ]
        },
        {
            "patient_id": 79440,
            "patient_mrn": "95249",
            "rank_today": 3,
            "rank_yesterday": null,
            "days_ranked": 5,
            "payer": "MCD COVENTRY",
            "primary_physician": null,
            "initial_admission_date": "10/12/2022",
            "last_transfer_date": "12/20/2022",
            "risk_rank_reasons": [
                "15 days since last admission",
                "Diagnosis of D649 : ANEMIA, UNSPECIFIED",
                "AmLODIPine Besylate ordered 15 days ago (on 01/10/2023)",
                "Diagnosis of B3749 : OTHER UROGENITAL CANDIDIASIS",
                "Diagnosis of J9601 : ACUTE RESPIRATORY FAILURE WITH HYPOXIA"
            ]
        }
    ]
}
```

## PDF Report Download
> Support for this functionality will be available mid-March '23

In order to download the PDF report for the facility, use the *report_download_url* property value in conjunction with the unique secret that is provided to you separately from the MySAIVA setup screen (UI TBD). 

For example:

If:
   report_download_url value is: `https://api.saiva.ai/v2/report/sample/rth/1/20230214`

Call:
   `GET https://api.saiva.ai/v2/report/sample/rth/1/20230214`

With the following secret value and accept mime type in the header:

    x-saiva-api-token: <API TOKEN VALUE>
    Accept: application/pdf

Using curl:

    curl GET https://api.saiva.ai/v2/report/sample/rth/1/20230214
        -H "x-saiva-api-token: <API TOKEN VALUE>"
        -H "Accept: application/pdf"
    
A successful call to the above call will redirect to download using HTTPS the report file from a temporary URL that will expire after 5 minutes. 
## Get Help
For questions and support issues please contact support@saiva.ai
