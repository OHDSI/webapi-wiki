# WebAPI Security Configuration

This guide will guide through the details required to configure the WebAPI security mechanisms.

## Overview

Security in WebAPI involves [authentication and authorization](http://www.differencebetween.net/technology/difference-between-authentication-and-authorization/). Authentication verifies you and who you say you are while authorization decides if you have permission to access a resource. The WebAPI security subsystem is built on top of [Apache Shiro](http://shiro.apache.org/documentation.html) framework to facilitate this functionality. For more details on the implementation of security in WebAPI, please refer to the [[Security Implementation]] guide.

## Authentication Configuration

By default, security is disabled for WebAPI. This guide will provide an overview of how to enable security and go through the various options for securing the REST endpoints. This guide assumes you have read through the [[WebAPI Installation Guide]] and are familiar with modifying your `settings.xml` to configure the application.

### Enable Security & Secure Socket Layer (SSL)

The `<security>` settings are controlled via the settings.xml.  The relevant security-related settings are shown below


```
<security.provider>AtlasRegularSecurity</security.provider>
<security.origin>*</security.origin>
<server.port>8080</server.port>
<security.ssl.enabled>true</security.ssl.enabled>
<security.cors.enabled>true</security.cors.enabled>
```

- **security.provider**: The default is `DisabledSecurity`. To enable security in the application this value is set to `AtlasRegularSecurity`.
- **security.origin**: Use `*` to allow any client to connect to the REST endpoints. This setting is used to narrow the scope of clients that are able to connect to the REST endpoints. You may set this to a specific client application (i.e. `http://ohdsi.org/web/atlas`) and WebAPI will only accept connections originating from this domain.
- **server.port**: This setting specifies the port to use to listen for client connections. The default is `8080` and this value is generally switched to `443` when using a secured connection.
- **security.ssl.enabled**: Set to true to enable SSL for encrypting connections to the REST endpoints. Check the [[SSL Configuration In Tomcat]] guide for more information on how to set up a server to use SSL.
- **server.ssl.key-store, server.ssl.key-store-password**: Specify the location of the `server.ssl.key-store` and `server.ssl.key-store-password` for use with the [[SSL Configuration In Tomcat]].
- **security.cors.enabled**: Set to true to enable Cross-Origin Resource Sharing [CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)
- **security.maxLoginAttempts**: Maximum number of login attempts before a lockout period is initiated.
- **security.duration.initial**: The `initial` length of lockout if there are more than `security.maxLoginAttempts` failed login attempts
- **security.duration.increment**:  The `incremental` lockout time will be added to the total lockout time for each subsequent login attempt.

Once `AtlasRegularSecurity` is enabled, you will be able to utilize the authentication mechanisms detailed below.

### Lightweight Directory Access (LDAP)

The following settings are used to control LDAP settings:

```
<security.ldap.dn>cn={0},dc=example,dc=org</security.ldap.dn>
<security.ldap.url>ldap://localhost:389</security.ldap.url>
<security.ldap.baseDn></security.ldap.baseDn>
<security.ldap.system.username></security.ldap.system.username>
<security.ldap.system.password></security.ldap.system.password>
```

- **security.ldap.dn**: The LDAP distinguished name
- **security.ldap.url**: The LDAP URL
- **security.ldap.baseDn**: The LDAP base distinguished name
- **security.ldap.system.username**: User name for accessing LDAP
- **security.ldap.system.password**: Password for the `username`

### Active Directory (AD)

The following settings are used to control Active Directory settings:

```
<security.ad.url>ldap://localhost:389</security.ad.url>
<security.ad.searchBase>CN=Users,DC=example,DC=org</security.ad.searchBase>
<security.ad.principalSuffix>@example.org</security.ad.principalSuffix>
<security.ad.system.username></security.ad.system.username>
<security.ad.system.password></security.ad.system.password>
<security.ad.searchFilter></security.ad.searchFilter>
<security.ad.ignore.partial.result.exception>true</security.ad.ignore.partial.result.exception>
<security.ad.result.count.limit>30000</security.ad.result.count.limit> <!-- 0 means no limit -->
<security.ad.default.import.group>public</security.ad.default.import.group>
```

- **security.ad.url**: The LDAP endpoint for AD
- **security.ad.searchBase**: The search base for AD
- **security.ad.principalSuffix**: The user principal name suffix
- **security.ad.system.username**: The username for accessing AD
- **security.ad.system.password**: The password for `username` above.
- **security.ad.searchFilter**: Use to configure a search filter
- **security.ad.ignore.partial.result.exception**: true/false
- **security.ad.result.count.limit**: Limit the number of AD results to 0 means no limit
- **security.ad.default.import.group**: The group to use for importing users from AD into WebAPI's configuration database.

### OpenID configuration

The following settings are used to control OpenID settings:

```
<security.oid.clientId></security.oid.clientId>
<security.oid.apiSecret></security.oid.apiSecret>
<security.oid.url></security.oid.url>
<security.oid.redirectUrl>http://localhost/index.html#/welcome/</security.oid.redirectUrl>
```

- **security.oid.clientId**: The OpenID client ID
- **security.oid.apiSecret**: The API secret key provided by the OpenID provider
- **security.oid.url**: The OpenID URL to use for authentication
- **security.oid.redirectUrl**: The callback URL to use for redirecting users to the application

### Central Authentication Security (CAS)

The following settings are used to control CAS settings:

```
<security.cas.loginUrl></security.cas.loginUrl>
<security.cas.callbackUrl></security.cas.callbackUrl>
<security.cas.serverUrl></security.cas.serverUrl>
<security.cas.cassvcs></security.cas.cassvcs>
<security.cas.casticket>casticket</security.cas.casticket>
```

- **security.cas.loginUrl**: The login URL
- **security.cas.callbackUrl**: The callback URL to call after a successful login
- **security.cas.serverUrl**: The CAS server URL
- **security.cas.cassvcs**: The CAS service
- **security.cas.casticket**: The CAS ticket

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

### Google Cloud Identity-Aware Proxy (IAP)

The following settings are used to control Google IAP settings:

```
<security.googleIap.cloudProjectId></security.googleIap.cloudProjectId>
<security.googleIap.backendServiceId></security.googleIap.backendServiceId>
```

- **security.googleIap.cloudProjectId**: The Google cloud project ID.
- **security.googleIap.backendServiceId**: The Google backend service ID.

### Basic Security Configuration

Please see the [[Basic Security Configuration]] guide for a set up that allows for a separate credential data store.

## Authorization

Authorization is handled based on the REST endpoint URL and HTTP method of the request. This portion of the configuration is data-driven with the relative endpoint paths stored in the `sec_permission` table of the WebAPI database. For more details, please see the Authorization section of the [[Security Implementation]].

## ATLAS Role Based Security

Authorized routes can be grouped into roles in the system and applied to users. See the [[Atlas Security]] section to see how this is facilitiated through a user interface in [ATLAS](https://github.com/OHDSI/Atlas).
