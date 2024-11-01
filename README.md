# Fetch HealthCheckPath setting for sites belongs to a subscription
How to know if 'auto heal' is enabled for all the site within the subscription

**Approach #1 App Services/ Web Apps**

For all the web apps under the subscription, it needs only a subscription Id. Users having access issues may not be able to get the desired results.
```
_subscriptionId="<subid>"
_webApps=$(az webapp list --subscription $_subscriptionId --query "[].{name:name, resourceGroup:resourceGroup}" --output jsonc)
for item in $(echo "$_webApps" | jq -r '.[] | @base64'); do
    _jq() {  echo ${item} | base64 --decode | jq -r ${1}
           }
     _name=$(_jq '.name')
     _resourceGroup=$(_jq '.resourceGroup')
      _healthCheckPath=$(az webapp config show --subscription $_subscriptionId --resource-group $_resourceGroup --name $_name --query "healthCheckPath" --output tsv)
    echo "[{ Is_autoHealEnabled: $_healthCheckPath  } { Web App: $_name} { Resource Group: $_resourceGroup }]"
done
```

**Approach #2 App Services/ Web Apps**
{Not recommended as it may put pressure on the server and your request may be throttled }

For all the web apps and subscriptions under the account.  I would suggest using this cautiously as it will make many API requests to the backend. Users having access issues may not be able to get the desired results.

```

_subscriptionIds=$(az account subscription list --query "[].{subscriptionId:subscriptionId}" --output jsonc)
for Subs in $(echo "$_subscriptionIds" | jq -r '.[] | @base64'); do
  _jq() {  echo ${Subs} | base64 --decode | jq -r ${1}
           }
    _subId=$(_jq '.subscriptionId')
_webApps=$(az webapp list --subscription $_subId --query "[].{name:name, resourceGroup:resourceGroup}" --output jsonc)
for item in $(echo "$_webApps" | jq -r '.[] | @base64'); do
    _jq() {  echo ${item} | base64 --decode | jq -r ${1}
           }

     _name=$(_jq '.name')
     _resourceGroup=$(_jq '.resourceGroup')
     _healthCheckPath=$(az webapp config show --subscription $_subscriptionId --resource-group $_resourceGroup --name $_name --query "healthCheckPath" --output tsv)
    echo "[{ Is_autoHealEnabled: $_healthCheckPath  } { Web App: $_name} { Resource Group: $_resourceGroup }]"
done
done
```


Refer below for the **Function Apps**

For all the function apps under the subscription, it needs only a subscription Id. Users having access issues may not be able to get the desired results.
```
subscriptionId="<subid>"
webApps=$(PYTHONWARNINGS="ignore" az functionapp list --subscription $subscriptionId --query "[].{name:name, resourceGroup:resourceGroup}" --output jsonc)
for item in $(echo "$webApps" | jq -r '.[] | @base64'); do
    _jq() {  echo ${item} | base64 --decode | jq -r ${1}
           }
     _name=$(_jq '.name')
     _resourceGroup=$(_jq '.resourceGroup')
     	 _healthCheckPath=$(PYTHONWARNINGS="ignore" az functionapp show --name "$_name" --resource-group "$_resourceGroup" --query "siteConfig.healthCheckPath" -o tsv | grep -q '.' && echo "true" || echo "false")
    echo "[{ Is_HealthCheckEnabled:$_healthCheckPath  }  { Function App Name: $_name} { Resource Group: $_resourceGroup }]"
done

```

