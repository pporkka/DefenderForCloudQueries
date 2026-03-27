# DefenderForCloudQueries
KQL Queries for Defender for Cloud (Azure thingy).

These KQL queries may help to gather information from Defender for Cloud, such as security recommendations and security scores. These may come handy when you have many subscriptions and large clouds where UI in the Defender for Cloud may not be the optimal solution. Yes, you can get CSV files downloaded straight up, but they lack for example subscription names and filtering with for example tags.

These are all executable in Azure Resource Graph Explorer, which at the time of writing does not for example allow "let xxx = <query>" to ease up the writing.

Most, if not all of these, require at least "Security Reader" role for any subscription (or for tenant root management level) to access DfC settings and information.

## KQL_DefenderForStorage_Settings.kql

List of Subscriptions, storage accounts and some defender related settings. This unifies information from enabled defender for cloud plans and settings from storage accounts themselves. Utilizes "securityresources" table information "microsoft.security/defenderforstoragesettings" and "microsoft.security/pricings". 

The point of this script is to find where your antimalware settings are enabled and where they are not. 

The "Coverage" column displays the Defender for Storage enablement, and by default the antimalware is on as well when the plan is enabled. "FullyCovered" means the DfS is enabled on subscription level. If the coverage is "NotCovered", then Defender for Storage is not enabled for that subscription which of course means that antimalware can't be enabled either.

The column "enabled" is from the storage account's settings and should usually be semantically the same as "coverage", i.e. "FullyCovered" = enabled and "NotCovered" = not enabled. The subscription can have the antimalware disabled even when subscription has antimalware enabled, this you can see from override column. If the "override" = "true", then that specific storage account's setting's overrides the defender for storage subscription level settings (usually means antimalware for specific storage is disabled even if it is enabled on sub level). 

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

## DfcGitHubRecommendations.kql

Organization's github can be connected to Defender for Cloud to get unified experience to security data from github.
Prerequisites to get this working: Add your enterprise github with a connector to your Defender for Cloud. You will need some heavy (org owner) access permissions to do it. 

Some of the recommendations have subassessments (example vulnerable packages), see below for another query to get those.

All this is available in the Defender UI, but if you need a neat list, this may help to get that.

This query lists "unhealthy" recommendations from the GitHub such as:
| Recommendation | 
| -------- | 
| GitHub repositories should have secret scanning enabled |
| GitHub repositories should have code scanning enabled |
| GitHub repositories should have Dependabot scanning enabled |
| GitHub repositories should have secrets scanning findings resolved |
| GitHub repositories should have code scanning findings resolved |
| GitHub repositories should have API security testing findings resolved |
| GitHub repositories should have dependency vulnerability scanning findings resolved |
| GitHub repositories should have force pushes to default branch disabled |
| GitHub repositories should have protection policies for default branch enabled |
| GitHub repositories should not use self hosted runners |
| GitHub repositories should require minimum two-reviewer approval for code pushes. |
| GitHub organizations should have more than one person with administrator permissions |
| GitHub organizations should not make action secrets accessible to all repositories |
| GitHub organizations should have actions workflow permissions set to read-only |
| GitHub organizations should enforce multifactor authentication for outside collaborators |
| GitHub organizations should have base permissions set to no permissions or read |
| GitHub organizations should have secret scanning push protection enabled |
| GitHub organizations should block Copilot suggestions that match public code. |



## DfCGitHubSubassessments.KQL

Organization's github can be connected to Defender for Cloud to get unified experience to security data from github.

This query lists all the subassessments from the specified connector. Subassessments contain at least vulnerable packages that you should get updated with eg. dependabot.  Unfortunately the repos that I had access didn't have secret scanning enabled so I am not sure if they are supposed to be in the subassessments and if they are, this query may end up not showing them. Just use the first 3 lines of the query to get everything and go from there.

Prerequisites to get this working: Add your enterprise github with a connector to your Defender for Cloud. You will need some heavy (org owner) access permissions to do it. 

## DevOpsGH - Folder

This folder contains GitHub recommendation specific queries to really drill down with a recommendation specific information.

This is a work in progress. 


