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
* `crowdstrike_devices_collector_id`: The Devices Cribl REST Collector ID.
* `crowdstrike_vulnerabilities_collector_id`: The Vulnerabilities Cribl REST Collector ID.
* `crowdstrike_alerts_collector_id`: The Alerts Cribl REST Collector ID.
* `crowdstrike_alerts_filter_low_severity`: Set to true (the default) to filter out low severity alerts.
* `crowdstrike_default_splunk_index`: Default index for the Splunk output - defaults to `crowdstrike`.

## Upgrades

TBD - initial pipeline-only release.

## Release Notes

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