- Syself
  - Nirmata
  - ./Devrel
  - Revert



### Upgrading from open-source Kyverno to Nirmata Enterprise Subscription

For users having open-source Kyverno of version 1.5.0 or above installed in their cluster, execute the following command to upgrade directly to Nirmata Enterprise Subscription:

````bash
helm upgrade kyverno --namespace kyverno nirmata/kyverno --set licenseManager.licenseKey=<license key >[,licenseManager.apiKey=<api key>]
````
>Note: Replace `<license key>` and `<api key>` with your license-key and api-key respectively.

### Uninstalling the Enterprise Kyverno Chart

The below command will uninstall the kyverno deployment and remove all the Kubernetes components associated with the chart and delete the release:

````bash
helm delete -n kyverno kyverno
````
