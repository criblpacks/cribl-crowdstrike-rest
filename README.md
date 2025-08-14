# Cribl Crowdstrike REST Collector Pack
----
## About this Pack

This Pack is designed to collect, process, and output Crowdstrike data via the Crowdstrike REST API. It currently supports the following API's:
* Host/Device Details ([falconpy link](https://www.falconpy.io/Service-Collections/Hosts.html#getdevicedetailsv2))
* Vulnerabilities ([falconpy link](https://www.falconpy.io/Service-Collections/Spotlight-Vulnerabilities.html#combinedqueryvulnerabilities))
* Endpoint Alerts ([falconpy link](https://www.falconpy.io/Service-Collections/Alerts.html#postentitiesalertsv1))

The Pack includes OCSF and Splunk output processing. OCSF data is mapped to the following Classes:
* Host/Device Details [Device Inventory Info [5001] Class](https://schema.ocsf.io/1.4.0/classes/inventory_info)
* Vulnerabilities - [Vulnerability Finding [2002] Class](https://schema.ocsf.io/1.4.0/classes/vulnerability_finding)
* Endpoint Alerts - [Detection Finding [2004] Class](https://schema.ocsf.io/1.4.0/classes/detection_finding)

Splunk data is mapped to the following sourcetypes - these are the sourcetypes used by the Crowdstrike-supported TA's:
* Host/Device Details:`sourcetype=crowdstrike:device:json`
* Vulnerabilities: `sourcetype=crowdstrike:spotlight:vulnerability:json`
* Endpoint Alerts: `sourcetype=crowdstrike:unified:alert:json`

Note: The official Crowdstrike API documentation access requires a support contract.


## Deployment
After installing the Pack, you must perform the following:

1. Add the Event Breaker Ruleset included in Appendix A below to your Stream instance under Processing > Knowledge > Event Breaker Rules.
2. Obtain a ```Client ID``` and ```Client Secret``` from your Crowdstrike Administrator. These credentials must have ```read``` access to the Hosts, Vulnerabilities, and Alerts API endpoints. 
3. Add the three Collector Sources in Appendix B to your Stream instance via Data > Sources > Collectors > REST. Configure each with the credentials from above and perform a Run > Preview to verify.
4. Schedule each Collector and enable State Tracking (with default configuration) for the Alerts and Vulnerabilities Collectors *only*. Suggested schedules for each are:
   
   *  Alerts: Every 5 minutes
   *  Devices: Once/day - this endpoint is configured to retrieve all Devices so keep that in mind.
   *  Vulnerabilities: Every hour
5. Connect the Crowdstrike Rest Collectors to the Pack. On the global Routes page, add a new route, specify a filter expression (if using the default Collector names, than something like ```__inputId.includes('in_crowdstrike')``` will work) and choose the ```cribl-crowdstrike-rest``` Pack in the Pipeline dropdown.

The following are the in-Pack configurable items - review/update them as needed. 

### Outputs

Each data type can be configured to output data in either OCSF or normalized JSON (Splunk) format. Enable *only one* format for each of the following pipelines:
* ```cribl_crowdstrike_alerts```
* ```cribl_crowdstrike_devices```
* ```cribl_crowdstrike_vulnerabilities```

### Lookups
The Pack includes a lookup called `crowdstrike_device_type_mapping.csv` that is used to generate the [OSCF Device](https://schema.ocsf.io/1.4.0/objects/device) `type` and `type_id` fields:
* `product_type_description`: A user-defined value within Crowdstrike. Add/update entries for your environment.
* `chassis_type_desc`: A standard Crowdstrike field. 

### Variables

The Pack has the following variables:
* `crowdstrike_devices_collector_id`: The Devices Cribl REST Collector ID.
* `crowdstrike_vulnerabilities_collector_id`: The Vulnerabilities Cribl REST Collector ID.
* `crowdstrike_alerts_collector_id`: The Alerts Cribl REST Collector ID.
* `crowdstrike_alerts_filter_low_severity`: Set to true (the default) to filter out low severity alerts.
* `crowdstrike_default_splunk_index`: Default index for the Splunk output - defaults to `crowdstrike`.

## Release Notes

### Version 0.1.0 - 2025-08-01

External Beta
* Contains the pipelines for processing the three Crowdstrike data types
* Supports either OCSF or Splunk output formats

## Contributing to the Pack
To contribute to the Pack, please connect with us on [Cribl Community Slack](https://cribl-community.slack.com/). You can suggest new features or offer to collaborate.

## License
This Pack uses the following license: [Apache 2.0](https://github.com/criblio/appscope/blob/master/LICENSE).


## Appendix A
Consolidated Crowdstrike API Event Breaker

```
{
  "id": "Crowdstrike API Ruleset",
  "minRawLength": 256,
  "rules": [
    {
      "condition": "_raw.includes(\"device_policies\")",
      "type": "json_array",
      "timestampAnchorRegex": "/modified_timestamp/",
      "timestamp": {
        "type": "auto",
        "length": 150
      },
      "timestampTimezone": "UTC",
      "timestampEarliest": "-420weeks",
      "timestampLatest": "+1week",
      "maxEventBytes": 134217728,
      "disabled": false,
      "parserEnabled": false,
      "shouldUseDataRaw": false,
      "jsonExtractAll": false,
      "delimiterRegex": "/\\t/",
      "fieldsLineRegex": "/^#[Ff]ields[:]?\\s+(.*)/",
      "headerLineRegex": "/^#/",
      "nullFieldVal": "-",
      "cleanFields": true,
      "eventBreakerRegex": "/[\\n\\r]+(?!\\s)/",
      "name": "Crowdstrike_Devices",
      "jsonTimeField": "modified_timestamp"
    },
    {
      "condition": "_raw.includes(\"legacy-detects\") || _raw.includes('automated_triage')",
      "type": "json_array",
      "timestampAnchorRegex": "/created_timestamp/",
      "timestamp": {
        "type": "auto",
        "length": 50
      },
      "timestampTimezone": "UTC",
      "timestampEarliest": "-420weeks",
      "timestampLatest": "+1week",
      "maxEventBytes": 134217728,
      "disabled": false,
      "parserEnabled": false,
      "shouldUseDataRaw": false,
      "jsonExtractAll": false,
      "eventBreakerRegex": "/[\\n\\r]+(?!\\s)/",
      "name": "Crowdstrike_Alerts",
      "jsonArrayField": "resources"
    },
    {
      "condition": "_raw.includes('vulnerability_id') || source.includes('vulnerabilities')",
      "type": "json_array",
      "timestampAnchorRegex": "/updated_timestamp/",
      "timestamp": {
        "type": "auto",
        "length": 150
      },
      "timestampTimezone": "UTC",
      "timestampEarliest": "-420weeks",
      "timestampLatest": "+1week",
      "maxEventBytes": 134217728,
      "disabled": false,
      "parserEnabled": false,
      "shouldUseDataRaw": false,
      "jsonExtractAll": false,
      "eventBreakerRegex": "/[\\n\\r]+(?!\\s)/",
      "jsonArrayField": "resources",
      "name": "Crowdstrike_Vulnerabilities",
      "jsonTimeField": "updated_timestamp"
    }
  ],
  "description": "Event breaker rules for Crowdstrike API data"
}
```

## Appendix B
Crowdstrike Collector Sources JSON

### Crowdstrike Alerts
```{
  "type": "collection",
  "ttl": "4h",
  "ignoreGroupJobsLimit": false,
  "removeFields": [],
  "resumeOnBoot": false,
  "schedule": {},
  "streamtags": [],
  "workerAffinity": false,
  "collector": {
    "conf": {
      "discovery": {
        "discoverType": "http",
        "discoverMethod": "get",
        "pagination": {
          "type": "none"
        },
        "enableDiscoverCode": false,
        "discoverUrl": "'https://api.us-2.crowdstrike.com/alerts/queries/alerts/v1'",
        "discoverRequestParams": [
          {
            "name": "limit",
            "value": "9999"
          },
          {
            "name": "filter",
            "value": "`updated_timestamp:>'${new Date(earliest*1000 || Date.now()-(5*60*1000)).toISOString()}'`"
          }
        ],
        "discoverRequestHeaders": [
          {
            "name": "Accept",
            "value": "application/json"
          },
          {
            "name": "Content-Type",
            "value": "application/json"
          },
          {
            "name": "User-Agent",
            "value": "Cribl"
          }
        ]
      },
      "collectMethod": "post_with_body",
      "pagination": {
        "type": "none",
        "offsetField": "offset",
        "limitField": "limit",
        "limit": 50,
        "maxPages": 50,
        "zeroIndexed": false
      },
      "authentication": "oauth",
      "timeout": 0,
      "useRoundRobinDns": false,
      "disableTimeFilter": true,
      "decodeUrl": true,
      "rejectUnauthorized": false,
      "captureHeaders": false,
      "safeHeaders": [],
      "retryRules": {
        "type": "backoff",
        "interval": 1000,
        "limit": 5,
        "multiplier": 2,
        "maxIntervalMs": 20000,
        "codes": [
          429,
          503
        ],
        "enableHeader": true,
        "retryConnectTimeout": false,
        "retryConnectReset": false,
        "retryHeaderName": "retry-after"
      },
      "__scheduling": {
        "stateTracking": {}
      },
      "loginUrl": "'https://api.us-2.crowdstrike.com/oauth2/token'",
      "authHeaderKey": "Authorization",
      "authHeaderExpr": "`Bearer ${token}`",
      "clientSecretParamName": "client_secret",
      "loginBody": "`client_id=${username}&client_secret=${password}`",
      "tokenRespAttribute": "access_token",
      "collectUrl": "'https://api.us-2.crowdstrike.com/alerts/entities/alerts/v1'",
      "credentialsSecret": "CrowdstrikeAPI",
      "authRequestHeaders": [
        {
          "name": "Content-Type",
          "value": "application/x-www-form-urlencoded"
        },
        {
          "name": "accept",
          "value": "application/json"
        },
        {
          "name": "User-agent",
          "value": "Cribl"
        }
      ],
      "collectBody": "`{\"ids\":${JSON.stringify(resources)}}`",
      "collectRequestHeaders": [],
      "authRequestParams": [
        {
          "name": "client_id",
          "value": "'changeme'"
        }
      ],
      "clientSecretParamValue": "changeme"
    },
    "destructive": false,
    "type": "rest"
  },
  "input": {
    "type": "collection",
    "staleChannelFlushMs": 10000,
    "sendToRoutes": true,
    "preprocess": {
      "disabled": true
    },
    "throttleRatePerSec": "0",
    "metadata": [],
    "breakerRulesets": [
      "Crowdstrike API Ruleset Non-Pack"
    ]
  },
  "savedState": {},
  "id": "in_crowdstrike_alerts"
}
```

### Crowdstrike Devices
```{
  "type": "collection",
  "ttl": "4h",
  "ignoreGroupJobsLimit": false,
  "removeFields": [],
  "resumeOnBoot": false,
  "schedule": {},
  "streamtags": [],
  "workerAffinity": false,
  "collector": {
    "conf": {
      "discovery": {
        "discoverType": "http",
        "discoverMethod": "get",
        "pagination": {
          "type": "response_body",
          "maxPages": 500,
          "attribute": [
            "offset"
          ],
          "lastPageExpr": "offset==\"\""
        },
        "enableDiscoverCode": false,
        "discoverUrl": "'https://api.us-2.crowdstrike.com/devices/queries/devices-scroll/v1'",
        "discoverRequestParams": [
          {
            "name": "limit",
            "value": "2"
          },
          {
            "name": "offset",
            "value": "`${offset}`"
          }
        ],
        "discoverRequestHeaders": [
          {
            "name": "accept",
            "value": "'application/json'"
          }
        ],
        "discoverDataField": "resources"
      },
      "collectMethod": "get",
      "pagination": {
        "type": "response_body",
        "maxPages": 500,
        "attribute": [
          "offset"
        ],
        "lastPageExpr": "offset==\"\""
      },
      "authentication": "oauth",
      "timeout": 0,
      "useRoundRobinDns": false,
      "disableTimeFilter": false,
      "decodeUrl": false,
      "rejectUnauthorized": false,
      "captureHeaders": true,
      "safeHeaders": [],
      "retryRules": {
        "type": "backoff",
        "interval": 1000,
        "limit": 5,
        "multiplier": 2,
        "maxIntervalMs": 20000,
        "codes": [
          429,
          503
        ],
        "enableHeader": true,
        "retryConnectTimeout": false,
        "retryConnectReset": false,
        "retryHeaderName": "retry-after"
      },
      "__scheduling": {
        "stateTracking": {}
      },
      "loginUrl": "'https://api.us-2.crowdstrike.com/oauth2/token'",
      "authHeaderKey": "Authorization",
      "authHeaderExpr": "`Bearer ${token}`",
      "clientSecretParamName": "client_secret",
      "collectUrl": "'https://api.us-2.crowdstrike.com/devices/entities/devices/v2'",
      "collectRequestParams": [
        {
          "name": "limit",
          "value": "500"
        },
        {
          "name": "offset",
          "value": "`${offset}`"
        },
        {
          "name": "ids",
          "value": "`${id}`"
        }
      ],
      "collectRequestHeaders": [
        {
          "name": "accept",
          "value": "'application/json'"
        }
      ],
      "clientSecretParamValue": "'changme'",
      "authRequestParams": [
        {
          "name": "client_id",
          "value": "'changme'"
        }
      ],
      "tokenRespAttribute": "access_token",
      "authRequestHeaders": [
        {
          "value": "application/x-www-form-urlencoded",
          "name": "Content-Type"
        },
        {
          "name": "accept",
          "value": "application/json"
        }
      ]
    },
    "destructive": false,
    "encoding": "utf8",
    "type": "rest"
  },
  "input": {
    "type": "collection",
    "staleChannelFlushMs": 10000,
    "sendToRoutes": true,
    "preprocess": {
      "disabled": true
    },
    "throttleRatePerSec": "0",
    "breakerRulesets": [
      "Crowdstrike API Ruleset Non-Pack"
    ],
    "metadata": []
  },
  "savedState": {},
  "id": "in_crowdstrike_devices"
}
```

### Crowdstrike Vulnerabilities
```{
  "type": "collection",
  "ttl": "4h",
  "ignoreGroupJobsLimit": false,
  "removeFields": [],
  "resumeOnBoot": false,
  "schedule": {},
  "streamtags": [],
  "workerAffinity": false,
  "collector": {
    "conf": {
      "discovery": {
        "discoverType": "none"
      },
      "collectMethod": "get",
      "pagination": {
        "type": "response_body",
        "maxPages": 50,
        "attribute": [
          "after"
        ],
        "lastPageExpr": "after==\"\""
      },
      "authentication": "oauth",
      "timeout": 0,
      "useRoundRobinDns": false,
      "disableTimeFilter": false,
      "decodeUrl": false,
      "rejectUnauthorized": false,
      "captureHeaders": true,
      "safeHeaders": [],
      "retryRules": {
        "type": "backoff",
        "interval": 1000,
        "limit": 5,
        "multiplier": 2,
        "maxIntervalMs": 20000,
        "codes": [
          429,
          503
        ],
        "enableHeader": true,
        "retryConnectTimeout": false,
        "retryConnectReset": false,
        "retryHeaderName": "retry-after"
      },
      "__scheduling": {
        "stateTracking": {}
      },
      "loginUrl": "'https://api.us-2.crowdstrike.com/oauth2/token'",
      "authHeaderKey": "Authorization",
      "authHeaderExpr": "`Bearer ${token}`",
      "clientSecretParamName": "client_secret",
      "collectUrl": "'https://api.us-2.crowdstrike.com/spotlight/combined/vulnerabilities/v1'",
      "collectRequestParams": [
        {
          "name": "filter",
          "value": "`updated_timestamp:>'${earliest ? C.Time.strftime(earliest,'%Y-%m-%dT%H:%M') : C.Time.strftime(Date.now()/1000 - 86400,'%Y-%m-%dT%H:%M')}'+updated_timestamp:<'${latest ? C.Time.strftime(latest,'%Y-%m-%dT%H:%M') : C.Time.strftime(Date.now()/1000,'%Y-%m-%dT%H:%M')}'+status:'open'`"
        },
        {
          "name": "limit",
          "value": "500"
        },
        {
          "name": "after",
          "value": "`${after}`"
        },
        {
          "name": "facet",
          "value": "\"cve\""
        },
        {
          "name": "facet",
          "value": "\"host_info\""
        },
        {
          "name": "facet",
          "value": "\"remediation\""
        },
        {
          "name": "facet",
          "value": "\"evaluation_logic\""
        }
      ],
      "collectRequestHeaders": [
        {
          "name": "Content-Type",
          "value": "'application/json'"
        },
        {
          "name": "accept",
          "value": "'application/json'"
        }
      ],
      "clientSecretParamValue": "'changeme'",
      "authRequestParams": [
        {
          "name": "client_id",
          "value": "'changeme'"
        }
      ],
      "tokenRespAttribute": "access_token",
      "authRequestHeaders": [
        {
          "value": "application/x-www-form-urlencoded",
          "name": "Content-Type"
        },
        {
          "name": "accept",
          "value": "application/json"
        },
        {
          "name": "user_agent",
          "value": "cribl"
        }
      ]
    },
    "destructive": false,
    "encoding": "utf8",
    "type": "rest"
  },
  "input": {
    "type": "collection",
    "staleChannelFlushMs": 10000,
    "sendToRoutes": true,
    "preprocess": {
      "disabled": true
    },
    "throttleRatePerSec": "0",
    "breakerRulesets": [
      "Crowdstrike API Ruleset Non-Pack"
    ]
  },
  "savedState": {},
  "description": "Pulling Data from the Crowdstrike Vulnerability API.  You can query using updated_timestamp (default) or created_timestamp; your choice. ",
  "id": "in_crowdstrike_vulnerabilities"
}
```