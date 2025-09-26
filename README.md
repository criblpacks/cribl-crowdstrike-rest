# Cribl Crowdstrike REST Collector Pack
----

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

## About this Pack

A key benefit of the Pack (in the *very* near future!) is to provide pre-defined REST Collector Sources for all three data types. See the Deployment section for details on how to configure them. 

While this Pack does not have pre-configured Destinations, it does provide either OCSF or Splunk as an output format. 

## Deployment
Currently, this Pack only has a few configurable items. 

#### Lookups
The Pack includes a lookup called `crowdstrike_device_type_mapping.csv` that is used to generate the [OSCF Device](https://schema.ocsf.io/1.4.0/objects/device) `type` and `type_id` fields:
* `product_type_description`: A user-defined value within Crowdstrike. Add/update entries for your environment.
* `chassis_type_desc`: A standard Crowdstrike field. 

#### Variables

The Pack has the following variables:
* `crowdstrike_alerts_filter_low_severity`: Set to true (the default) to filter out low severity alerts.
* `crowdstrike_default_splunk_index`: Default index for the Splunk output - defaults to `crowdstrike`.

## Release Notes

### Version 0.1.5 - 2025-09-25

External Beta
* Adds support for the Alerts V2 API Endpoint as the Alerts V1 Endpoint reached EOL on 9/30/2025.
  
### Version 0.1.0 - 2025-08-01

External Beta
* Contains the pipelines for processing the three Crowdstrike data types
* Supports either OCSF or Splunk output formats

## Contributing to the Pack
 
This required section informs Cribl users how to contact you. Community Slack is the most common approach, but you could list other contact methods, such as an email address.

To contribute to this Pack, or to report any issues or enhancement requests, please connect with _____ _____ on [Cribl Community Slack](https://cribl-community.slack.com).

## Contact
To contact us please email <your-email@example.com>.

## License
All publicly posted Packs must include a license that allows for use by the general public, such as Apache 2.0, MIT, or GNU.
(Most Packs use the Apache 2.0 license.)

This Pack uses the following license: [`Apache 2.0`](https://github.com/criblio/appscope/blob/master/LICENSE).

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

### Crowdstrike Alerts V2
```
{
  "type": "collection",
  "ttl": "4h",
  "ignoreGroupJobsLimit": false,
  "removeFields": [],
  "resumeOnBoot": false,
  "schedule": {
    "cronSchedule": "*/5 * * * *",
    "maxConcurrentRuns": 1,
    "skippable": true,
    "run": {
      "rescheduleDroppedTasks": true,
      "maxTaskReschedule": 1,
      "logLevel": "info",
      "jobTimeout": "60m",
      "mode": "run",
      "timeRangeType": "relative",
      "timeWarning": {},
      "expression": "true",
      "minTaskSize": "1MB",
      "maxTaskSize": "10MB",
      "stateTracking": {
        "stateUpdateExpression": "__timestampExtracted !== false && {latestTime: (state.latestTime || 0) > _time ? state.latestTime : _time}",
        "stateMergeExpression": "(prevState.latestTime || 0) > newState.latestTime ? prevState : newState",
        "enabled": true
      }
    },
    "enabled": true
  },
  "streamtags": [],
  "workerAffinity": false,
  "collector": {
    "conf": {
      "discovery": {
        "discoverType": "http",
        "discoverMethod": "get",
        "pagination": {
          "type": "response_body",
          "maxPages": 50,
          "attribute": [
            "offset"
          ],
          "lastPageExpr": "offset==\"\""
        },
        "enableStrictDiscoverParsing": false,
        "enableDiscoverCode": false,
        "discoverUrl": "'https://api.us-2.crowdstrike.com/alerts/queries/alerts/v2'",
        "discoverRequestParams": [
          {
            "name": "limit",
            "value": "1000"
          },
          {
            "name": "filter",
            "value": "`updated_timestamp:>='${new Date(earliest*1000*300 || Date.now()-(60*60*1000*24*30)).toISOString()}'`"
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
        "type": "response_body",
        "maxPages": 0,
        "offsetField": "offset",
        "limitField": "limit",
        "limit": 50,
        "zeroIndexed": false,
        "attribute": [
          "offset"
        ]
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
      "collectUrl": "'https://api.us-2.crowdstrike.com/alerts/entities/alerts/v2'",
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
      "collectBody": "`{\"composite_ids\":${JSON.stringify(resources)}}`",
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
      "authRequestParams": [
        {
          "name": "client_id",
          "value": "'YOURCLIENTID'"
        }
      ],
      "clientSecretParamValue": "YOURSECRET"
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
  "id": "in_crowdstrike_alerts_v2"
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