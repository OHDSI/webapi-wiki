# WebAPI Security Configuration

This guide will guide through the details required to configure the WebAPI security mechanisms. 

## Overview

Security in WebAPI involves [authentication and authorization](http://www.differencebetween.net/technology/difference-between-authentication-and-authorization/). Authentication verifies you and who you say you are while authorization decided if you have permission to access a resource. The WebAPI security subsystem is built on top of [Apache Shiro](http://shiro.apache.org/documentation.html) framework to facilitate this functionality. For more details on the implementation of security in WebAPI, please refer to the [[Security Implementation]] guide.

## Authentication Configuration

By default, security is disabled for WebAPI. This guide will provide an overview to enable security and go through the various options for securing the REST endpoints. This guide will assume you have read through the [[WebAPI Installation Guide]] and are familiar with modifying your `settings.xml` to configure the application.

### Enable Security & Secure Socket Layer (SSL)

The `<security>` settings are controlled via the settings.xml (a sample settings.xml is provided in the WebAPI repository root).  The relevant security-related settings are shown below


```
<security.provider>AtlasRegularSecurity</security.provider>
<security.origin>*</security.origin>
<server.port>8080</server.port>
<security.ssl.enabled>true</security.ssl.enabled>
```

- **security.provider**: The default is `DisabledSecurity`. To enable security in the applicaiton this value is set to `AtlasRegularSecurity`. 
- **security.origin**: The default is `*` which allows any client to connect to the REST endpoints. This setting is used to narrow the scope of clients that are able to connect to the REST endpoints. You may set this to a specific client application (i.e. `http://ohdsi.org/web/atlas`) and WebAPI will only accept connections originating from this domain.
- **server.port**: This setting specifies the port to use to listen for client connections. The default is `8080` and this value is generally switched to `443` when using a secured connection.
- **security.ssl.enabled**: Set to true to enable SSL for encrypting connections to the REST endpoints. Check the [[SSL Configuration In Tomcat]] guide for more information on how to set up a server to use SSL.
- **server.ssl.key-store, server.ssl.key-store-password**: Specify the location of the `server.ssl.key-store` and `server.ssl.key-store-password` for use with the [[SSL Configuration In Tomcat]].
  
### Active Directory/LDAP

Once `AtlasRegularSecurity` is enabled, you will be able to utilize Active Directory, LDAP or Kerberos authentication. 

*Note*: Confirm with @pavgra there are no other settings required here. Mainly concerned with Kerberos/LDAP configuration.

### OAuth configuration

WebAPI uses [`pac4j`](https://github.com/pac4j/pac4j) to provide support for several [OAuth](https://oauth.net/2/) authentication services. Here are the relevant settings.xml entries to utilize OAuth for authentication. It is important to note that if you plan to use OAuth with the providers below you must ensure that the machine hosting WebAPI has Internet & Firewall access to the OAuth endpoints.

```
<security.oauth.callback.ui>https://<server>/atlas/#/welcome</security.oauth.callback.ui>
<security.oauth.callback.api>https://<server>:8080/WebAPI/user/oauth/callback</security.oauth.callback.api>

<security.oauth.google.apiKey></security.oauth.google.apiKey>
<security.oauth.google.apiSecret></security.oauth.google.apiSecret>
<security.oauth.facebook.apiKey></security.oauth.facebook.apiKey>
<security.oauth.facebook.apiSecret></security.oauth.facebook.apiSecret>
<security.oauth.github.apiKey></security.oauth.github.apiKey>
<security.oauth.github.apiSecret></security.oauth.github.apiSecret>
```

- **security.oauth.callback.ui**: Specifies the callback URL to use when authentication is complete.
- **security.oauth.callback.api**: Specifies the callback URL for OAuth - this will need to point to your WebAPI instance.
- **security.oauth.`<provider>`.apiKey, security.oauth.`<provider>`.apiSecret**: Currently WebAPI supports OAuth through Google, Facebook and GitHub. The corresponding OAuth `provider` will provide the values for the API Key and API Secret for these settings.


### Basic Security Configuration

Please see the [[Basic Security Configuration]] guide for a set up that allows for a seperate credential data store.

## Authorization

Authorization is handled based on the REST endpoint URL and HTTP method of the request. This portion of the configuration is data-driven with the relative endpoint paths stored in the `sec_permission` table of the WebAPI database. For more details, please see the Authorization section of the [[Security Implementation]].

## ATLAS Role Based Security

Authorized routes can be grouped into roles in the system and applied to users. See the [[Atlas Security]] section to see how this is facilitiated through a user interface in [ATLAS](https://github.com/OHDSI/Atlas).

