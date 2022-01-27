# Forgerock/ID.me

ID.me’s Identity Gateway platform provides a SAML 2.0 capable IDP service, which supports standardized,
signed and encrypted assertions and different attribute bundles. This functionality can be used to enable
applications to participate in a federated single sign-on (SSO) relationship with the ID.me network of
credentials.

## Overview

### Generate an ID.me client 

An ID.me account is required to begin an integration. The only account requirement is completing email verification prior to starting an integration. 

* Navigate to developer.id.me
* If you have an ID.me account, click Sign In, if you do not have an ID.me account, click Sign Up
* Complete Sign In/Up

Now that you have a valid ID.me Developer account, you can: 

* Proceed to `Start a New Integration` and click `Continue` 
* Input organization information and click `Continue`
* Once an organization is created, you can proceed to `View My Applications`
* Click `Create New`
* Input application information. Please note, Only application name, display name, and redirect URI are required fields
* Once all fields are input and formatted correctly, you may proceed to create your OAuth application by clicking `Continue`
* Record your Client ID and Client Secret

__Note: ID.me provides a sandbox environment that allows partners to verify with any data to generate a mocked response. Contact ID.me to provide the appropriate credentials and access to begin the integration. Email ID.me at partnersupport@id.me for policy information and sandbox access.__

### Social Registration Tree

There are two nodes associated with Identity Providers:

* `Select Identity Provider Node`: The "Select Identity Provider Node" prompts the user to select a social identity provider to register or log in with, or (optionally) continue on with a local registration or login flow. When a provider is selected, the flow continues on to the Social Provider Handler node.

* `Social Provider Handler Node`: The "Social Provider Handler Node" is used in combination with the Select Identity Provider node. It communicates with the selected provider and then collects the information provided after the user has authorized the service. It then takes that information and runs a transformation script to prepare it.

ForgeRock Identity Platform includes a transformation script called Normalized Profile to Managed User, which this node uses to transform the identity object gathered from the identity provider into a ForgeRock Identity Platform object.

The node then queries IDM to see if the user already exists. If the user exists, they are logged in. If the user does not exist, the user will need to be created.

### To Configure a Basic Social Registration Tree:

* In your `realm`, go to `Journeys` and click `+ New Journey`
* Create a `Name` for the Journey and set the `Identity Object` to `Alpha realm - Users`
* Click `Save`

Now that your Journey is created, decide whether users can log in with their local credentials, and add the relevant nodes to the tree:

* Social authentication trees allowing local authentication might look like the following:

* Social authentication trees enforcing social authentication login might look like the following:

__Note: Use the above images as a reference to construct your basic social registration tree. Search for each node using the `Filter Nodes` search bar. Once built, follow the below steps to finish configuring each node.__

__Click and congifure the `Page Node`:__ 

Navigate to the `Select Identity Provider` portion of the `Page Node` and check `Include local authentication`

__Click and configure the `Social Provider Handler Node`:__

In the `Transformation Script` field, select `Normalized Profile to Managed User`. This script will transform the normalized identity provider's profile object into an appropriate object that ForgeRock Identity Platform can use.

In `Client Type`, select `BROWSER` when using the ForgeRock Identity Platform UI, or the ForgeRock SDK for JavaScript.

__Click and configure the "Required Attributes Present Node" and the "Create Object Node":__

In the `Identity Resource field`, configure the relevant managed identity resource type, likely `managed/alpha_user`.

__Click and configure the `Attribute Collector Node`:__

Here we can prompt users to self-assert any additional attributes that the Identity Provider does not provide. For example: `mail`, `givenName`, and `sn` attributes.

__Click `Save`__


### Add ID.me as an Identity Provider

* In the AM Admin UI, go to `Realms` > `Realm Name` > `Services`.
* Check if `Social Identity Provider Service` appears in the list of services configured for the realm.
* If it does not, add it: Click on `Add a Service`, and select `Social Identity Provider Service` from the drop-down list.
* The service's Configuration page appears.
* Ensure that the `Enabled` switch is on.
* Go to the `Secondary Configurations` tab.
* In the `Add a Secondary Configuration` drop-down list, select ID.me as the identity provider.
* If you do not see ID.me, select the following to add a custom identity provider client: `Client Configuration for providers that implement the OpenID Connect specification`
* The new identity provider configuration page appears.

### Add Custom Transformation Script 

The Transformation Script maps ID.me attibutes to ForgeRock attributes to create user objects. This script will be used as a part of the next step. To create this script, do the following: 

* Navigate to `Scripts` on the left side of the screen, click `+ New Script`
* Provide a Name for the script, such as `ID.me Profile Transformation`
* Set `Script Type` to `Social Identity Provider Profile Transformation`
* Within the script field, beginning at line 9, add the following script. This is a sample script with a baseline set attributes. If you would like to further customize this script with additional attributes, please follow this [User Identity Attributes and Properties Reference
](https://backstage.forgerock.com/docs/idcloud/latest/identities/user-identity-properties-attributes-reference.html)
```
import static org.forgerock.json.JsonValue.field
import static org.forgerock.json.JsonValue.json
import static org.forgerock.json.JsonValue.object

return json(object(
        field("id", rawProfile.uuid),
        field("displayName", rawProfile.fname),
        field("givenName", rawProfile.fname),
        field("familyName", rawProfile.lname),
        field("email", rawProfile.email),
        field("username", rawProfile.email)))
```

### Configure the Identity Provider with ID.me Details 

Populate the configuration page with details from your ID.me instance. For more information about each option, click on the Tooltip next to each respective field. Below is guide for some of the ID.me specific fields. Do not worry if you are missing some of the details; you will be able to edit the configuration later, after saving the client profile for the first time. Save your changes to access all the configuration fields for the client. 

* `Name`: Provide your own unique identifier 
* `Auth ID Key`: Field used to identify a user by the social provider, likely `id`
* `Client ID` and `Client Secret`: Copied and pasted from your ID.me instance
* `Authentication Endpoint URL`: `https://api.idmelabs.com/oauth/authorize` if testing in sandbox OR `https://api.id.me/oauth/authorize` for a production implementation 
* `Access Token Endpoint URL`: `https://api.idmelabs.com/oauth/token` if testing in sandbox OR `https://api.id.me/oauth/token` for a prooduction implementation 
* `User Profile Service URL`: `https://api.idmelabs.com/api/public/v2/attributes.json` if testing in sandbox OR `https://api.id.me/api/public/v2/attributes.json` for a production implementation
* `Redirect URL`: This will be tenet specific, following the convention `https://[YOUR_FQDN_HERE]/am` 
* `OAuth Scopes`: Gather the scope(s) for the requested resources from your ID.me instance. If you have any questions, work with your ID.me Sales Engineer for guidance. 
* `Well Known Endpoint`: `https://api.idmelabs.com/oidc/.well-known/openid-configuration` if testing in sandbox OR `https://api.id.me/oidc/.well-known/openid-configuration` for a production implementation
* `Issuer`: `https://api.idmelabs.com/oidc` if testing in sandbox OR `https://api.id.me/oidc` for a production implementation
* `JWKS URI Endpoint`: `https://api.idmelabs.com/oidc/.well-known/jwks` if testing in sandbox OR `https://api.id.me/oidc/.well-known/jwks` for a production implementation
* `Transform Script`: Set this value to the `Custom Transformation Script` created in the previous section - likely `ID.me Profile Transformation`

## Test ID.me Configuration

Copy and paste the `Preview URL` option in the top right-hand corner of the screen and test the ID.me/ForgeRock Integration.

[Click here to see a mockup of the user experience](https://invis.io/5AUHL6DT3PG)

## What's Next?

For assistance or more information, please contact us at [partnersupport@id.me](mailto:partnersupport@id.me)
