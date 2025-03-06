# DefenderForCloudQueries
KQL Queries for Defender for Cloud (Azure thingy).

These KQL queries may help to gather information from Defender for Cloud, such as security recommendations and security scores. These may come handy when you have many subscriptions and large clouds where UI in the Defender for Cloud may not be the optimal solution. Yes, you can get CSV files downloaded straight up, but they lack for example subscription names and filtering with for example tags.

These are all executable in Azure Resource Graph Explorer, which at the time of writing does not for example allow "let xxx = <query>" to ease up the writing.

## DfCAzureContainerRegistryVulnerabilities.kql

Retuns a list of vulnerabilities detected in Azure container registries detected by Defender for Cloud. 
List contains for example following: Registry name, CVE number, description of the vulnerability, severity, "evidence" (path on some cases), rg name, subscription name, tags and many other. 
List also contains lots of details, but could be simplified by removing something like assessment keys, id:s and others if use case is to view manually something.
Defender for Cloud CWP "Containers" plan need to be enabled and Registry Access under that.

## DfCRecommendationsBySubscriptionWithTags.kql

Defender for Cloud required, basic CSPM is sufficient. Returns a list of recommendations on all the subscriptions the person running has access to. 
List includes such things as subscription names which arent visible in the CSVs taken out directly from the Defender UI.
Script by default also filters by specific tags. The example filters with TagName "Workload" and TagValue "Prod".
Remove this part if you dont want to filter by tags: 
```
    | mvexpand tags
    | extend tagKey = tostring(bag_keys(tags)[0])
    | extend tagValue = tostring(tags[tagKey])
    | where (tagKey =~ "Workload" and tagValue =~"Prod")  // or (name contains 'SomethingSomething')
```
You can also add additional subscriptions in the case you mostly have tags, but need to add specific subscriptions in addition. 
Remove the comment to do so and change "somethingsomethin" to something else :). 
```
// or (name contains 'SomethingSomething')
```

## SecureScoresBySubscriptionWithTags.kql

This returns Secure scores for all the subscriptions, but this one filters by Tags. The result contains score in percentage and also the "current" score and "max" score.
Script by default also filters by specific tags. The example filters with TagName "Workload" and TagValue "Prod".
Remove this part if you dont want to filter by tags:
```
    | mvexpand tags
    | extend tagKey = tostring(bag_keys(tags)[0])
    | extend tagValue = tostring(tags[tagKey])
    | where (tagKey =~ "Workload" and tagValue =~"Prod")  // or (name contains 'SomethingSomething')
```
You can also add additional subscriptions in the case you mostly have tags, but need to add specific subscriptions in addition. 
Remove the comment to do so and change "somethingsomethin" to something else :). 
```
// or (name contains 'SomethingSomething')
```

## Enabled-DefenderForCloud-plans.kql

With a large amount of subscriptions, it may be painfull to find out what subscriptions have Defender for Cloud plans enabled or not.
This query does just that. The results are based on the pricing information (enabled==costs, disabled=free). The names in the results are sometimes not the same as in UI, so here's a 
table explaining what name.

| Plan name in pricing tiers | Plan name in Defender UI |
| -------- | -------- |
| OpenSourceRelationalDatabases	| Databases - Sub option: Open-source relational databases |
| AppServices | App Service |
| AI | AI workloads |
| CosmosDbs | Databases - Sub option: Azure Cosmos Db |
| SqlServerVirtualMachines | Databases - Sub option: SQL servers on machines |
| StorageAccounts | Storage |
| Containers | Containers |
| SqlServers | Databases - Sub option: Azure SQL Databases |
| CloudPosture | Defender CSPM  |
| VirtualMachines | Servers |
| Api | APIs |
| KeyVaults | Key Vault |
| Arm | Resource Manager |
| ContainerRegistry | Container Registries (Deprecated) |
| Dns | DNS (Deprecated) |
| KubernetesService |  Kubernetes (deprecated) |

