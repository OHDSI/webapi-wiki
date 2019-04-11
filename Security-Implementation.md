# Security implementation in WebAPI

This page describes the details of the security implementation in WebAPI.

## Dependencies

The security subsystem is built on top of Apache Shiro framework.

[Apache Shiro](http://shiro.apache.org/documentation.html) is a powerful and easy-to-use Java security framework that performs authentication, authorization, cryptography, and session management. With Shiro’s easy-to-understand API, you can quickly and easily secure any application – from the smallest mobile applications to the largest web and enterprise applications.

It also uses:  
  * [WAFFLE](https://github.com/dblock/waffle): for Windows authentication
  * [buji-pac4j](https://github.com/bujiio/buji-pac4j): for OAuth authentication
  * [Java JWT](https://github.com/jwtk/jjwt): JSON Web Token for Java and Android


## Security Layer

The `org.ohdsi.webapi.shiro.management.Security` abstraction makes it easy to maintain different behaviors of security subsystem. There are two implementations are available out of the box. These are `AtlasRegularSecurity` and `DisabledSecurity`. The first handles all the needs of ATLAS application, the latter disables security features. 

The default setting in the WebAPI pom.xml is `<security.provider>DisabledSecurity</security.provider>` which turns off security by loading the DisabledSecurity module. If you would like to enable security and load the AtlasRegularSecurity module, this can be done by adding `<security.provider>AtlasRegularSecurity</security.provider>` to the `<profile>` section of your settings.xml file as described in the [[Security Configuration]]. This does require that you rebuild the .war file and redeploy the application.

## Path-based security

The security subsystem is set up by assigning filters to URLs. Before a method underneath the URL is invoked, filters apply some logic. For example, attempt to authenticate user and check if user has permission required to access the method. 

## Authentication

WebAPI itself does not authenticate users, it delegates authentication to external service - Windows Authentication or OAuth provider.

There is a set of authenticating filters which allows to maintain different authentication providers. If authentication is successful, WebAPI responds with token, which then should be used to sign subsequent requests. The token is sent in `Bearer` header or as a path parameter in case of OAuth.

To sign request put the token prefixed with `Bearer` keyword into `Authorization` header 

```
Authorization: Bearer mytokenvalue
```

The token is a [JSON Web Token](https://jwt.io/). It is used not only as a proof if identity but also contains some useful information, such as login and user’s permissions. It allows to check permissions on client side without need of further communication with WebAPI.

Here is a partial list of endpoints related to authentication:

  * `/user/login`: Windows Authentication endpoint, responds with token within ''Bearer'' header
  * `/user/oauth/google`: Google OAuth, redirects to configured endpoint, putting token as a path parameter
  * `/user/oauth/facebook`: Facebook OAuth, redirects to configured endpoint, putting token as a path parameter
  * `/user/logout`: Invalidates token, request should be signed with valid token
  * `/user/refresh`: Invalidates current token and creates new one with updated permissions and expiration time, request should be signed with valid token

### OAuth Settings

Currently supported OAuth providers are Google, Facebook and GitHub. See the [[Security Configuration]] for more information on configuring for OAuth.

### Adding a new OAuth provider

OAuth authentication is handled with buji-pac4j [OAuth clients](http://www.pac4j.org/docs/clients/oauth.html).

To add support of new OAuth service provider into WebAPI, you need to go through following steps.

  * Register application and obtain oauth api credentials
    * Refer to OAuth service provider’s documentation to register application
    * Obtain your API key and secret
    * Adjust redirect URL. This is a URL of a WebAPI endpoint which is responsible for processing provider’s response. The relative path of the endpoint is `/user/oauth/callback`. The same endpoint is used for all OAuth clients. To distinguish which client is responded add `client_name` parameter to the path. For example, `https://<server>/WebAPI/user/oauth/callback?client_name=FacebookClient` for Facebook.
  * Add oauth endpoint in WebAPI
    * Please see this [Pull Request #832](https://github.com/OHDSI/WebAPI/pull/832/files) which shows how GitHub oAuth was introduced into WebAPI.

### Basic Security Configuration

In the event that you do not have one of the supported OAuth providers available, WebAPI also supports as [[Basic Security Configuration]].

## Authorization

To access protected method, user must have an appropriate permission. WebAPI checks permissions which are built based on method’s URL and HTTP Method of the request. 

For example, request

```  
  GET: myservice/myresource/10
```

corresponds to permission

```
  myservice:myresource:10:get
```

However, user doesn’t need to have exactly this permission, because SHIRO provides wildcard permissions. It means, that permissions are multilevel and each level may be replaced with a wildcard. Levels are separated by the colon sign and the asterisk sign is used as wildcard character.

For the example above, user may access to resource if he or she has one of the following permissions (assuming that 10 is entity’s ID)

  * `*`: access to everything
  * `myservice`: access to all methods of `myservice`
  * `myservice:myresource:*:get`: access to any entity of `myresource`
  * `myservice:myresource:10:get`: access to entity with ID = 10

So that it’s easy to change level of granularity by giving permissions to users for a whole services or just for certain entities or whatever makes sense. 

For details on wildcard permissions please refer to https://shiro.apache.org/static/1.2.3/apidocs/org/apache/shiro/authz/permission/WildcardPermission.html. The list of permissions for the system are stored in the WebAPI database table `sec_permission`.

### Protect WebAPI Methods

To protect certain REST endpoint, assign appropriate filters to its URL. Please refer to the `org.ohdsi.webapi.shiro.management.FilterTemplates` for examples.


## Access Control

Permissions are not directly assigned to a user. Instead, they are grouped into roles and then roles are assigned to user. This makes access control flexible. You may think about it in terms of roles which user plays in your infrastructure. If user's responsibilities were changed, you can move he or she into another role. Or you can add permissions into role if you realized that this role should involve new activities. All you need is just update records in database by calling appropriate WebAPI method.

Here is a list of tables which security subsystem uses to handle access control:

  * `SEC_USER` - stores all registered users
  * `SEC_USER_ROLE` - offers many-to-many relation between users and roles
  * `SEC_ROLE` - stores roles
  * `SEC_ROLE_PERMISSION` - offers many-to-many relation between roles and permissions
  * `SEC_PERMISSION` - stores permissions

### Adding new users

When user logs in first time, record is created in `SEC_USER` table. Now user is registered and can be associated with roles.

There are two special roles - `public` role and the personal role. Both are assigned to every newly registered user. 

One personal role is created for each new user. The name of personal role is the same as user's login. 

This role is intended to hold permissions which are specific to certain user. For example, when user creates new entity, it may be useful to restrict access to methods which affect the entity, so that only author can change or delete it. 

This workflow works when user creates an entity (i.e. Concept Set, Cohort). Methods which affect these entities are protected. When entity is created, entity-level permissions required for these methods are created as well and then are assigned to creator's personal role.

`Public` role is intended to hold permissions which should have all users. Initially this role does not contain any permissions, but privileged users can add permissions into it, so that such permissions will be assigned to all current and future users.

### Predefined Roles

When WebAPI starts first time, it creates a set of roles and permissions. Here is a list:

  * `public`: intended to contain permissions granted to all users (empty by default)
  * `admin`: contains permissions required to manage roles and permissions and to add permissions into `public` role
  * `concept set creator`: contains permissions required to create concept sets
  * `cohort reader`: contains permissions required to read cohorts
  * `cohort creator`: contains permissions required to create cohorts
  * `Atlas users`: contains permissions for all actions across Atlas except for administration rights.

Each time when WebAPI starts, it looks for configured sources and creates appropriate permissions and roles. One role is created for each source. Roles are named like `Source user ({SourceKey})`, where `{SourceKey}` is a value of `source.source_key` column. For example `Source user (SYNPUF1K)`.

## ATLAS Settings

## Checking user’s permissions

Some UI controls may correspond to set of WebAPI calls. To ensure that enabling this UI control is safe permissions must be checked for all required methods. For example, to update Concept Set ATLAS performs two requests

```
POST: conceptset/{conceptsetId}
POST: conceptset/{conceptsetId}/items
```

where `{conceptsetId}` is ID of updated Concept Set.

To address this, `js/services/AuthAPI.js` in the [ATLAS](https://github.com/OHDSI/Atlas) contains set of `isPermitted...` methods to check all necessary permissions required to perform an atomic (in terms of ATLAS) actions. For the above example it is 

```
isPermittedUpdateConceptset(conceptsetId)
```

`AuthAPI.js` also contains `isAuthenticated()` method to check if access token is presented and not expired.

Note that these permission checks are performed against information contained in the token itself and it may be outdated at the time of checking. This means, that there is still a chance that ATLAS request will be refused by WebAPI with `Unauthorized` or `Forbidden` error. To handle such errors, you may use `handleAccessDenied(error)` method

```
  $.ajax({
    ...
    error: authApi.handleAccessDenied,
    ...
  });
```

To update the token use `refreshToken()` method. 

## Signing Requests To Protected Methods

The client application should sign request with access token to be able to call protected methods. This means that `Authorization` header must be set. To obtain valid `Authorization` header use `getAuthorizationHeader()` method of `AuthAPI.js`.

In case of JQuery AJAX call it may look like this

```
  $.ajax({
    ...
    headers: {
      Authorization: authApi.getAuthorizationHeader()
    },
    ...
    
  });
```
