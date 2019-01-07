# Certificate testing - Module developers
Two types of modules can be integrated with the Core Platform:
* modules running in a **scheduled mode**: those need a regular access to the Core Platform (done through certificate)
* modules running in an **on-demand mode**: those need a non-regular connexion to the Core Platform (done by a user, through the a secured protocol)

Modules that run on-demand only need to get their frontends integrated in the Hit2Gap web portal. This is documented at:
```
https://github.com/HIT2GAP-EU-PROJECT/Modules-Documentation/blob/master/portal_integration.md
```

In what follows, we suppose your module is correctly informed in the Core Platform.
The steps to follow are as follows:

1. request a certificate to the Core Platform administrator, that will provide a XXX.pem file to you.
2. in a console, try to be authentified to the Core Platform:
```
curl -X POST -v -k https://h2g-platform-core.nobatek.com/api/v0/auth/cert --cert XXX.pem
```
You should receive an answer containing a cookie session, and information about your identification (i.e. roles, user type, and unique ID), where:
     a. roles = chuck | building_manager | module_data_provider | module_data_processor
     b. user type = human | machine
c. extract the cookie session and test it agains the testing APIs
```
curl --header "Cookie: session=YOUR_COOKIE" https://h2g-platform-core.nobatek.com/api/v0/auth/demo/private/roles/0
curl --header "Cookie: session=$YOUR_COOKIE" https://h2g-platform-core.nobatek.com/api/v0/auth/demo/private/roles/1
```
