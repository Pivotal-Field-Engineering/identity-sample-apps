# Pivotal Single Sign-On Service Sample Applications

This repo holds separate sample applications for each one of the four OAuth 2.0 grant types supported by the Pivotal Single Sign-On Service. The GRANT_TYPE environment variable is already set to the relevant value mentioned below for each sample application. Each grant type maps to an Application Type as seen in the Pivotal Single Sign-On Service Dashboard.

The latest version of this repository supports Spring Boot 1.5.5+, Spring Security OAuth 2.2.0+ and SSO connector 2.1.1+. The last version to support Spring Boot 1.3 is tagged at [spring-boot/1.3](https://github.com/pivotal-cf/identity-sample-apps/releases/tag/spring-boot%2F1.3).

Application Type  | Grant Type
------------- | -------------
[Web App](/authcode)  | authorization_code
[Native Mobile App](/password)  | password
[Service-to-Service App](/client_credentials) | client_credentials
[Single Page JavaScript App](/implicit) | implicit

## <a name="step-1">Step 1</a>: Deploy Sample Application to Pivotal Cloud Foundry

Set the correct CF API target in the CF CLI and login as a Space Developer into the required Org and Space

    cf api api.<your-domain>
    
Go to your application directory and push the app.

    ./gradlew build
    cf push

**NOTE:** Your application is expected to crash on start-up until it is bound to the Single Sign-on Service using the instructions in the next section.

**NOTE:** If the PCF Routers are set up on Public IPs, you will need to update the internal_proxies variable in application.yml to your routers public IP.

## <a name="step-2">Step 2</a>: Bind the Application with the Pivotal Single Sign-On Service Instance
Follow the steps [here](http://docs.pivotal.io/p-identity/configure-apps/index.html#bind) to bind your application to the service instance. 

Restart your application after binding the service using Apps Manager or CF CLI.

## <a name="quick-start">Quick Start</a>: Authcode Sample App and Resource Server on SSO

As an alternative to Steps 1 and 2 above, you can also quickly deploy the authcode and resource server sample applications using application bootstrapping with the steps below. You can read more about these topics in the following sections.

1. First, make sure you created a [Service Plan](https://docs.pivotal.io/p-identity/manage-service-plans.html) for your Org as well as a [Service Instance](https://docs.pivotal.io/p-identity/manage-service-instances.html) named `sample-instance` for your Space, and login via CF CLI as a Space Developer into the required Org and Space.

2. Replace `manifest.yml` with `manifest.yml.quick-start` for the *authcode* and *resource-server* projects and update the `RESOURCE_URL` and `AUTH_SERVER` values in the manifest with your plan and domain values.

3. Build (`./gradlew build`) and push (`cf push`) both the *authcode* and *resource-server* projects to your Space where you are logged in as a Space Developer.
   
The sample application and resource server be available immediately bound to the SSO Service on start-up. You can then test the applications by creating test users with the `todo.read` and `todo.write` scopes for your plan using the steps [here](https://docs.pivotal.io/p-identity/configure-id-providers.html#add-to-int).

# Bootstrap Application Client Configurations for the Pivotal Single Sign-On Service Instance
Beginning in SSO 1.4.0, you can use the following values your application's manifest to bootstrap client configurations for your applications automatically when binding or rebinding your application to the service instance. These values will be automatically populated to the client configurations for your application through CF environment variables.

**NOTE:** These configurations are only applied at the initial service binding time. Subsequent `cf push` of the application will **NOT** update the configurations. You will either need to manually update the configurations via the SSO dashboard or unbind and rebind the service instance.

When you specify your own scopes and authorities, consider including openid for scopes on auth code, implicit, and password grant type applications, and uaa.resource for client credentials grant type applications, as these will not be provided if they are not specified.

The table below provides a description and the default values. Further details and examples are provided in the sample application manifests.

| Property Name | Description | Default |
| ------------- | ------------- | ------------- |
| name | Name of the application | (N/A - Required Value) |
| GRANT_TYPE | Allowed grant type for the application through the SSO service - only one grant type per application is supported by SSO | authorization_code |
| SSO_IDENTITY_PROVIDERS | Allowed identity providers for the application through the SSO service plan | uaa |
| SSO_REDIRECT_URIS | Comma separated whitelist of redirection URIs allowed for the application - Each value must start with http:// or https:// |  (Will always include the application route) |
| SSO_SCOPES | Comma separated list of scopes that belong to the application and are registered as client scopes with the SSO service. This value is ignored for client credential grant type applications. |  openid |
| SSO_AUTO_APPROVED_SCOPES | Comma separated list of scopes that the application is automatically authorized when acting on behalf of users through SSO service | <Defaults to existing scopes/authorities> |
| SSO_AUTHORITIES | Comma separated list of authorities that belong to the application and are registered as client authorities with the SSO service. Authorities are restricted to the space they were originally created. Privileged identity zone/plan administrator scopes (e.g. scim.read, idps.write) cannot be bootstrapped and must be assigned by zone/plan administrators. This value is ignored for any grant type other than client credentials. | uaa.resource |
| SSO_REQUIRED_USER_GROUPS | Comma separated list of groups a user must have in order to authenticate successfully for the application | (No value) |
| SSO_ACCESS_TOKEN_LIFETIME | Lifetime in seconds for the access token issued to the application by the SSO service | 43200 |
| SSO_REFRESH_TOKEN_LIFETIME | Lifetime in seconds for the refresh token issued to the application by the SSO service | 2592000 (not used for client credentials) |
| SSO_RESOURCES |  Resources that the application will use as scopes/authorities for the SSO service to be created during bootstrapping if they do not already exist - The input format can be referenced in the provided sample manifest. Note that currently all permissions within the same top level permission (e.g. todo.read, todo.write) must be specified in the same application manifest. Currently you cannot specify additional permissions in the same top level permission (e.g. todo.admin) in additional application manifests.| (No value) |
| SSO_ICON |  Application icon that will be displayed next to the application name on the Pivotal Account dashboard if show on home page is enabled - do not exceed 64kb | (No value) |
| SSO_LAUNCH_URL |  Application launch URL that will be used for the application on the Pivotal Account dashboard if show on home page is enabled | (Application route) |
| SSO_SHOW_ON_HOME_PAGE |  If set to true, the application will appear on the Pivotal Account dashboard with the corresponding icon and launch URL| True |

To remove any variables set through bootstrapping, you must use `cf unset-env <APP_NAME> <PROPERTY_NAME>` and rebind the application.
