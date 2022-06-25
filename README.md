# iAppLX for service discovery with Consul

This iApp is an example of MSDA working with Consul, including an audit processor.  

## Build (requires rpmbuild)

    $ npm run build

Build output is an RPM package
## Using IAppLX from BIG-IP UI
If you are using BIG-IP, install f5-iapplx-msda-consul RPM package using iApps->Package Management LX->Import screen. To create an application, use iApps-> Templates LX -> Application Services -> Applications LX -> Create screen. Default IApp LX UI will be rendered based on the input properties specified in basic pool IAppLX.

Pool name is mandatory when creating or updating iAppLX configuration. Optionally you can add any number of pool members.

## Using IAppLX from REST API to configure BIG-IP

Run the REST API with f5-iapplx-msda-consul IAppLX package. Pass in the remote BIG-IP to be trusted when starting REST API. For more comprehensive guide, please refer to clouddocs https://clouddocs.f5.com/products/iapp/iapp-lx/tmos-14_0/iapplx_ops_tutorials/working_with_iapplx_rest.html .

Create an Application LX block with hostname, deviceGroupName, poolName, poolType and poolMembers as shown below.
Save the JSON to block.json and use it in the curl call

```json
{
  "name": "msdaconsul",
  "inputProperties": [
    {
      "id": "consulEndpoint",
      "type": "STRING",
      "value": "http://1.1.1.1:8500",
      "metaData": {
        "description": "consul endpoint list",
        "displayName": "consul endpoints",
        "isRequired": true
      }
    },
    {
      "id": "serviceName",
      "type": "STRING",
      "value": "http",
      "metaData": {
        "description": "Service name to be exposed",
        "displayName": "Service Name in registry",
        "isRequired": true
      }
    },
    {
      "id": "poolName",
      "type": "STRING",
      "value": "/Common/consulSamplePool",
      "metaData": {
        "description": "Pool Name to be created",
        "displayName": "BIG-IP Pool Name",
        "isRequired": true
      }
    },
    {
      "id": "poolType",
      "type": "STRING",
      "value": "round-robin",
      "metaData": {
        "description": "load-balancing-mode",
        "displayName": "Load Balancing Mode",
        "isRequired": true,
        "uiType": "dropdown",
        "uiHints": {
          "list": {
            "dataList": [
              "round-robin",
              "least-connections-member",
              "least-connections-node"
            ]
          }
        }
      }
    },
    {
      "id": "healthMonitor",
      "type": "STRING",
      "value": "none",
      "metaData": {
        "description": "Health Monitor",
        "displayName": "Health Monitor",
        "isRequired": true,
        "uiType": "dropdown",
        "uiHints": {
          "list": {
            "dataList": [
              "tcp",
              "udp",
              "http",
              "none"
            ]
          }
        }
      }
    }
  ],
  "dataProperties": [
    {
      "id": "pollInterval",
      "type": "NUMBER",
      "value": 30,
      "metaData": {
        "description": "Interval of polling from BIG-IP to registry, 30s by default.",
        "displayName": "Polling Invertal",
        "isRequired": false
      }
    }
  ],
  "configurationProcessorReference": {
    "link": "https://localhost/mgmt/shared/iapp/processors/msdaconsulConfig"
  },
  "auditProcessorReference": {
    "link": "https://localhost/mgmt/shared/iapp/processors/msdaconsulEnforceConfiguredAudit"
  },
  "audit": {
    "intervalSeconds": 60,
    "policy": "ENFORCE_CONFIGURED"
  },
  "configProcessorTimeoutSeconds": 30,
  "statsProcessorTimeoutSeconds": 15,
  "configProcessorAffinity": {
    "processorPolicy": "LOAD_BALANCED",
    "affinityProcessorReference": {
      "link": "https://localhost/mgmt/shared/iapp/processors/affinity/load-balanced"
    }
  },
  "state": "TEMPLATE"
}
```

Post the block REST container using curl.
```bash
curl -sk -X POST -d @block.json https://bigip_mgmt_ip:8443/mgmt/shared/iapp/blocks
```
