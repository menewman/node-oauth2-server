=====================
 Model Specification
=====================

**Version >=5.x:** Callback support has been removed! Each model function supports either sync or async (``Promise`` or ``async function``) return values.

**Version <=4.x:** Each model function supports *promises*, *Node-style callbacks*, *ES6 generators* and *async*/*await* (using Babel_). Note that promise support implies support for returning plain values where asynchronism is not required.

.. _Babel: https://babeljs.io

::

  const model = {
    // We support returning promises.
    getAccessToken: function() {
      return new Promise('works!');
    },

    // Or sync-style values
    getAuthorizationCode: function() {
      return 'works!'
    },

    // Or, using generators.
    getClient: function*() {
      yield somethingAsync();
      return 'works!';
    },

    // Or, async/wait (using Babel).
    getUser: async function() {
      await somethingAsync();
      return 'works!';
    }
  };

  const OAuth2Server = require('@node-oauth/oauth2-server');
  let oauth = new OAuth2Server({model: model});

Code examples on this page use *promises*.

--------

.. _Model#generateAccessToken:

``generateAccessToken(client, user, scope)``
========================================================

Invoked to generate a new access token.

This model function is **optional**. If not implemented, a default handler is used that generates access tokens consisting of 40 characters in the range of ``a..z0..9``.

**Invoked during:**

- ``authorization_code`` grant
- ``client_credentials`` grant
- ``refresh_token`` grant
- ``password`` grant

**Arguments:**

+------------+----------+---------------------------------------------------------------------+
| Name       | Type     | Description                                                         |
+============+==========+=====================================================================+
| client     | Object   | The client the access token is generated for.                       |
+------------+----------+---------------------------------------------------------------------+
| user       | Object   | The user the access token is generated for.                         |
+------------+----------+---------------------------------------------------------------------+
| scope      | String   | The scopes associated with the access token. Can be ``null``.       |
+------------+----------+---------------------------------------------------------------------+

**Return value:**

A ``String`` to be used as access token.

:rfc:`RFC 6749 Appendix A.12 <6749#appendix-A.12>` specifies that access tokens must consist of characters inside the range ``0x20..0x7E`` (i.e. only printable US-ASCII characters).

**Remarks:**

``client`` is the object previously obtained through :ref:`Model#getClient() <Model#getClient>`.

``user`` is the user object previously obtained through :ref:`Model#getAuthorizationCode() <Model#getAuthorizationCode>` (``code.user``; authorization code grant), :ref:`Model#getUserFromClient() <Model#getUserFromClient>` (client credentials grant), :ref:`Model#getRefreshToken() <Model#getRefreshToken>` (``token.user``; refresh token grant) or :ref:`Model#getUser() <Model#getUser>` (password grant).

--------

.. _Model#generateRefreshToken:

``generateRefreshToken(client, user, scope)``
=========================================================

Invoked to generate a new refresh token.

This model function is **optional**. If not implemented, a default handler is used that generates refresh tokens consisting of 40 characters in the range of ``a..z0..9``.

**Invoked during:**

- ``authorization_code`` grant
- ``refresh_token`` grant
- ``password`` grant

**Arguments:**

+------------+----------+---------------------------------------------------------------------+
| Name       | Type     | Description                                                         |
+============+==========+=====================================================================+
| client     | Object   | The client the refresh token is generated for.                      |
+------------+----------+---------------------------------------------------------------------+
| user       | Object   | The user the refresh token is generated for.                        |
+------------+----------+---------------------------------------------------------------------+
| scope      | String   | The scopes associated with the refresh token. Can be ``null``.      |
+------------+----------+---------------------------------------------------------------------+

**Return value:**

A ``String`` to be used as refresh token.

:rfc:`RFC 6749 Appendix A.17 <6749#appendix-A.17>` specifies that refresh tokens must consist of characters inside the range ``0x20..0x7E`` (i.e. only printable US-ASCII characters).

**Remarks:**

``client`` is the object previously obtained through :ref:`Model#getClient() <Model#getClient>`.

``user`` is the user object previously obtained through :ref:`Model#getAuthorizationCode() <Model#getAuthorizationCode>` (``code.user``; authorization code grant), :ref:`Model#getRefreshToken() <Model#getRefreshToken>` (``token.user``; refresh token grant) or :ref:`Model#getUser() <Model#getUser>` (password grant).

--------

.. _Model#generateAuthorizationCode:

``generateAuthorizationCode(client, user, scope)``
=========================================

Invoked to generate a new authorization code.

This model function is **optional**. If not implemented, a default handler is used that generates authorization codes consisting of 40 characters in the range of ``a..z0..9``.

**Invoked during:**

- ``authorization_code`` grant

**Arguments:**

+------------+----------+---------------------------------------------------------------------+
| Name       | Type     | Description                                                         |
+============+==========+=====================================================================+
| client     | Object   | The client the authorization code is generated for.                 |
+------------+----------+---------------------------------------------------------------------+
| user       | Object   | The user the authorization code is generated for.                   |
+------------+----------+---------------------------------------------------------------------+
| scope      | String   | The scopes associated with the authorization code. Can be ``null``. |
+------------+----------+---------------------------------------------------------------------+

**Return value:**

A ``String`` to be used as authorization code.

:rfc:`RFC 6749 Appendix A.11 <6749#appendix-A.11>` specifies that authorization codes must consist of characters inside the range ``0x20..0x7E`` (i.e. only printable US-ASCII characters).

--------

.. _Model#getAccessToken:

``getAccessToken(accessToken)``
===========================================

Invoked to retrieve an existing access token previously saved through :ref:`Model#saveToken() <Model#saveToken>`.

This model function is **required** if :ref:`OAuth2Server#authenticate() <OAuth2Server#authenticate>` is used.

**Invoked during:**

- request authentication

**Arguments:**

+-------------+----------+---------------------------------------------------------------------+
| Name        | Type     | Description                                                         |
+=============+==========+=====================================================================+
| accessToken | String   | The access token to retrieve.                                       |
+-------------+----------+---------------------------------------------------------------------+

**Return value:**

An ``Object`` representing the access token and associated data.

+------------------------------+--------+--------------------------------------------------+
| Name                         | Type   | Description                                      |
+==============================+========+==================================================+
| token                        | Object | The return value.                                |
+------------------------------+--------+--------------------------------------------------+
| token.accessToken            | String | The access token passed to ``getAccessToken()``. |
+------------------------------+--------+--------------------------------------------------+
| token.accessTokenExpiresAt   | Date   | The expiry time of the access token.             |
+------------------------------+--------+--------------------------------------------------+
| [token.scope]                | String | The authorized scope of the access token.        |
+------------------------------+--------+--------------------------------------------------+
| token.client                 | Object | The client associated with the access token.     |
+------------------------------+--------+--------------------------------------------------+
| token.client.id              | String | A unique string identifying the client.          |
+------------------------------+--------+--------------------------------------------------+
| token.user                   | Object | The user associated with the access token.       |
+------------------------------+--------+--------------------------------------------------+

``token.client`` and ``token.user`` can carry additional properties that will be ignored by *oauth2-server*.

**Remarks:**

::

  function getAccessToken(accessToken) {
    // imaginary DB queries
    return db.queryAccessToken({access_token: accessToken})
      .then(function(token) {
        return Promise.all([
          token,
          db.queryClient({id: token.client_id}),
          db.queryUser({id: token.user_id})
        ]);
      })
      .spread(function(token, client, user) {
        return {
          accessToken: token.access_token,
          accessTokenExpiresAt: token.expires_at,
          scope: token.scope,
          client: client, // with 'id' property
          user: user
        };
      });
  }

--------

.. _Model#getRefreshToken:

``getRefreshToken(refreshToken)``
=============================================

Invoked to retrieve an existing refresh token previously saved through :ref:`Model#saveToken() <Model#saveToken>`.

This model function is **required** if the ``refresh_token`` grant is used.

**Invoked during:**

- ``refresh_token`` grant

**Arguments:**

+--------------+----------+---------------------------------------------------------------------+
| Name         | Type     | Description                                                         |
+==============+==========+=====================================================================+
| refreshToken | String   | The access token to retrieve.                                       |
+--------------+----------+---------------------------------------------------------------------+

**Return value:**

An ``Object`` representing the refresh token and associated data.

+-------------------------------+--------+----------------------------------------------------+
| Name                          | Type   | Description                                        |
+===============================+========+====================================================+
| token                         | Object | The return value.                                  |
+-------------------------------+--------+----------------------------------------------------+
| token.refreshToken            | String | The refresh token passed to ``getRefreshToken()``. |
+-------------------------------+--------+----------------------------------------------------+
| [token.refreshTokenExpiresAt] | Date   | The expiry time of the refresh token.              |
+-------------------------------+--------+----------------------------------------------------+
| [token.scope]                 | String | The authorized scope of the refresh token.         |
+-------------------------------+--------+----------------------------------------------------+
| token.client                  | Object | The client associated with the refresh token.      |
+-------------------------------+--------+----------------------------------------------------+
| token.client.id               | String | A unique string identifying the client.            |
+-------------------------------+--------+----------------------------------------------------+
| token.user                    | Object | The user associated with the refresh token.        |
+-------------------------------+--------+----------------------------------------------------+

``token.client`` and ``token.user`` can carry additional properties that will be ignored by *oauth2-server*.

**Remarks:**

::

  function getRefreshToken(refreshToken) {
    // imaginary DB queries
    return db.queryRefreshToken({refresh_token: refreshToken})
      .then(function(token) {
        return Promise.all([
          token,
          db.queryClient({id: token.client_id}),
          db.queryUser({id: token.user_id})
        ]);
      })
      .spread(function(token, client, user) {
        return {
          refreshToken: token.refresh_token,
          refreshTokenExpiresAt: token.expires_at,
          scope: token.scope,
          client: client, // with 'id' property
          user: user
        };
      });
  }

--------

.. _Model#getAuthorizationCode:

``getAuthorizationCode(authorizationCode)``
=======================================================

Invoked to retrieve an existing authorization code previously saved through :ref:`Model#saveAuthorizationCode() <Model#saveAuthorizationCode>`.

This model function is **required** if the ``authorization_code`` grant is used.

**Invoked during:**

- ``authorization_code`` grant

**Arguments:**

+-------------------+----------+---------------------------------------------------------------------+
| Name              | Type     | Description                                                         |
+===================+==========+=====================================================================+
| authorizationCode | String   | The authorization code to retrieve.                                 |
+-------------------+----------+---------------------------------------------------------------------+

**Return value:**

An ``Object`` representing the authorization code and associated data.

+------------------------+--------+--------------------------------------------------------------+
| Name                   | Type   | Description                                                  |
+========================+========+==============================================================+
| code                   | Object | The return value.                                            |
+------------------------+--------+--------------------------------------------------------------+
| code.authorizationCode | String | The authorization code passed to ``getAuthorizationCode()``. |
+------------------------+--------+--------------------------------------------------------------+
| code.expiresAt         | Date   | The expiry time of the authorization code.                   |
+------------------------+--------+--------------------------------------------------------------+
| [code.redirectUri]     | String | The redirect URI of the authorization code.                  |
+------------------------+--------+--------------------------------------------------------------+
| [code.scope]           | String | The authorized scope of the authorization code.              |
+------------------------+--------+--------------------------------------------------------------+
| code.client            | Object | The client associated with the authorization code.           |
+------------------------+--------+--------------------------------------------------------------+
| code.client.id         | String | A unique string identifying the client.                      |
+------------------------+--------+--------------------------------------------------------------+
| code.user              | Object | The user associated with the authorization code.             |
+------------------------+--------+--------------------------------------------------------------+

``code.client`` and ``code.user`` can carry additional properties that will be ignored by *oauth2-server*.

**Remarks:**

::

  function getAuthorizationCode(authorizationCode) {
    // imaginary DB queries
    return db.queryAuthorizationCode({authorization_code: authorizationCode})
      .then(function(code) {
        return Promise.all([
          code,
          db.queryClient({id: code.client_id}),
          db.queryUser({id: code.user_id})
        ]);
      })
      .spread(function(code, client, user) {
        return {
          authorizationCode: code.authorization_code,
          expiresAt: code.expires_at,
          redirectUri: code.redirect_uri,
          scope: code.scope,
          client: client, // with 'id' property
          user: user
        };
      });
  }

--------

.. _Model#getClient:

``getClient(clientId, clientSecret)``
=================================================

Invoked to retrieve a client using a client id or a client id/client secret combination, depending on the grant type.

This model function is **required** for all grant types.

**Invoked during:**

- ``authorization_code`` grant
- ``client_credentials`` grant
- ``refresh_token`` grant
- ``password`` grant

**Arguments:**

+--------------+----------+---------------------------------------------------------------------+
| Name         | Type     | Description                                                         |
+==============+==========+=====================================================================+
| clientId     | String   | The client id of the client to retrieve.                            |
+--------------+----------+---------------------------------------------------------------------+
| clientSecret | String   | The client secret of the client to retrieve. Can be ``null``.       |
+--------------+----------+---------------------------------------------------------------------+

**Return value:**

An ``Object`` representing the client and associated data, or a falsy value if no such client could be found.

+-------------------------------+---------------+--------------------------------------------------------------------------------------+
| Name                          | Type          | Description                                                                          |
+===============================+===============+======================================================================================+
| client                        | Object        | The return value.                                                                    |
+-------------------------------+---------------+--------------------------------------------------------------------------------------+
| client.id                     | String        | A unique string identifying the client.                                              |
+-------------------------------+---------------+--------------------------------------------------------------------------------------+
| [client.redirectUris]         | Array<String> | Redirect URIs allowed for the client. Required for the ``authorization_code`` grant. |
+-------------------------------+---------------+--------------------------------------------------------------------------------------+
| client.grants                 | Array<String> | Grant types allowed for the client.                                                  |
+-------------------------------+---------------+--------------------------------------------------------------------------------------+
| [client.accessTokenLifetime]  | Number        | Client-specific lifetime of generated access tokens in seconds.                      |
+-------------------------------+---------------+--------------------------------------------------------------------------------------+
| [client.refreshTokenLifetime] | Number        | Client-specific lifetime of generated refresh tokens in seconds.                     |
+-------------------------------+---------------+--------------------------------------------------------------------------------------+

The return value (``client``) can carry additional properties that will be ignored by *oauth2-server*.

**Remarks:**

::

  function getClient(clientId, clientSecret) {
    // imaginary DB query
    let params = {client_id: clientId};
    if (clientSecret) {
      params.client_secret = clientSecret;
    }
    return db.queryClient(params)
      .then(function(client) {
        return {
          id: client.id,
          redirectUris: client.redirect_uris,
          grants: client.grants
        };
      });
  }

--------

.. _Model#getUser:

``getUser(username, password)``
===========================================

Invoked to retrieve a user using a username/password combination.

This model function is **required** if the ``password`` grant is used.

**Invoked during:**

- ``password`` grant

**Arguments:**

+------------+----------+---------------------------------------------------------------------+
| Name       | Type     | Description                                                         |
+============+==========+=====================================================================+
| username   | String   | The username of the user to retrieve.                               |
+------------+----------+---------------------------------------------------------------------+
| password   | String   | The user's password.                                                |
+------------+----------+---------------------------------------------------------------------+

**Return value:**

An ``Object`` representing the user, or a falsy value if no such user could be found. The user object is completely transparent to *oauth2-server* and is simply used as input to other model functions.

**Remarks:**

::

  function getUser(username, password) {
    // imaginary DB query
    return db.queryUser({username: username, password: password});
  }

--------

.. _Model#getUserFromClient:

``getUserFromClient(client)``
=========================================

Invoked to retrieve the user associated with the specified client.

This model function is **required** if the ``client_credentials`` grant is used.

**Invoked during:**

- ``client_credentials`` grant

**Arguments:**

+------------+----------+---------------------------------------------------------------------+
| Name       | Type     | Description                                                         |
+============+==========+=====================================================================+
| client     | Object   | The client to retrieve the associated user for.                     |
+------------+----------+---------------------------------------------------------------------+
| client.id  | String   | A unique string identifying the client.                             |
+------------+----------+---------------------------------------------------------------------+

**Return value:**

An ``Object`` representing the user, or a falsy value if the client does not have an associated user. The user object is completely transparent to *oauth2-server* and is simply used as input to other model functions.

**Remarks:**

``client`` is the object previously obtained through :ref:`Model#getClient() <Model#getClient>`.

::

  function getUserFromClient(client) {
    // imaginary DB query
    return db.queryUser({id: client.user_id});
  }

--------

.. _Model#saveToken:

``saveToken(token, client, user)``
==============================================

Invoked to save an access token and optionally a refresh token, depending on the grant type.

This model function is **required** for all grant types.

**Invoked during:**

- ``authorization_code`` grant
- ``client_credentials`` grant
- ``refresh_token`` grant
- ``password`` grant

**Arguments:**

+-------------------------------+----------+---------------------------------------------------------------------+
| Name                          | Type     | Description                                                         |
+===============================+==========+=====================================================================+
| token                         | Object   | The token(s) to be saved.                                           |
+-------------------------------+----------+---------------------------------------------------------------------+
| token.accessToken             | String   | The access token to be saved.                                       |
+-------------------------------+----------+---------------------------------------------------------------------+
| token.accessTokenExpiresAt    | Date     | The expiry time of the access token.                                |
+-------------------------------+----------+---------------------------------------------------------------------+
| [token.refreshToken]          | String   | The refresh token to be saved.                                      |
+-------------------------------+----------+---------------------------------------------------------------------+
| [token.refreshTokenExpiresAt] | Date     | The expiry time of the refresh token.                               |
+-------------------------------+----------+---------------------------------------------------------------------+
| [token.scope]                 | String   | The authorized scope of the token(s).                               |
+-------------------------------+----------+---------------------------------------------------------------------+
| client                        | Object   | The client associated with the token(s).                            |
+-------------------------------+----------+---------------------------------------------------------------------+
| user                          | Object   | The user associated with the token(s).                              |
+-------------------------------+----------+---------------------------------------------------------------------+

**Return value:**

An ``Object`` representing the token(s) and associated data.

+-----------------------------+--------+----------------------------------------------+
| Name                        | Type   | Description                                  |
+=============================+========+==============================================+
| token                       | Object | The return value.                            |
+-----------------------------+--------+----------------------------------------------+
| token.accessToken           | String | The access token passed to ``saveToken()``.  |
+-----------------------------+--------+----------------------------------------------+
| token.accessTokenExpiresAt  | Date   | The expiry time of the access token.         |
+-----------------------------+--------+----------------------------------------------+
| token.refreshToken          | String | The refresh token passed to ``saveToken()``. |
+-----------------------------+--------+----------------------------------------------+
| token.refreshTokenExpiresAt | Date   | The expiry time of the refresh token.        |
+-----------------------------+--------+----------------------------------------------+
| [token.scope]               | String | The authorized scope of the access token.    |
+-----------------------------+--------+----------------------------------------------+
| token.client                | Object | The client associated with the access token. |
+-----------------------------+--------+----------------------------------------------+
| token.client.id             | String | A unique string identifying the client.      |
+-----------------------------+--------+----------------------------------------------+
| token.user                  | Object | The user associated with the access token.   |
+-----------------------------+--------+----------------------------------------------+

``token.client`` and ``token.user`` can carry additional properties that will be ignored by *oauth2-server*.

If the ``allowExtendedTokenAttributes`` server option is enabled (see :ref:`OAuth2Server#token() <OAuth2Server#token>`) any additional attributes set on the result are copied to the token response sent to the client.

**Remarks:**

::

  function saveToken(token, client, user) {
    // imaginary DB queries
    let fns = [
      db.saveAccessToken({
        access_token: token.accessToken,
        expires_at: token.accessTokenExpiresAt,
        scope: token.scope,
        client_id: client.id,
        user_id: user.id
      }),
      db.saveRefreshToken({
        refresh_token: token.refreshToken,
        expires_at: token.refreshTokenExpiresAt,
        scope: token.scope,
        client_id: client.id,
        user_id: user.id
      })
    ];
    return Promise.all(fns);
      .spread(function(accessToken, refreshToken) {
        return {
          accessToken: accessToken.access_token,
          accessTokenExpiresAt: accessToken.expires_at,
          refreshToken: refreshToken.refresh_token,
          refreshTokenExpiresAt: refreshToken.expires_at,
          scope: accessToken.scope,
          client: {id: accessToken.client_id},
          user: {id: accessToken.user_id}
        };
      });
  }

--------

.. _Model#saveAuthorizationCode:

``saveAuthorizationCode(code, client, user)``
=========================================================

Invoked to save an authorization code.

This model function is **required** if the ``authorization_code`` grant is used.

**Invoked during:**

- ``authorization_code`` grant

**Arguments:**

+----------------------------+----------+---------------------------------------------------------------------+
| Name                       | Type     | Description                                                         |
+----------------------------+----------+---------------------------------------------------------------------+
| code                       | Object   | The code to be saved.                                               |
+----------------------------+----------+---------------------------------------------------------------------+
| code.authorizationCode     | String   | The authorization code to be saved.                                 |
+----------------------------+----------+---------------------------------------------------------------------+
| code.expiresAt             | Date     | The expiry time of the authorization code.                          |
+----------------------------+----------+---------------------------------------------------------------------+
| code.redirectUri           | String   | The redirect URI associated with the authorization code.            |
+----------------------------+----------+---------------------------------------------------------------------+
| [code.scope]               | String   | The authorized scope of the authorization code.                     |
+----------------------------+----------+---------------------------------------------------------------------+
| [code.codeChallenge]       | String   | The code challenge; hash or plain. Only present in PKCE requests.   |
+----------------------------+----------+---------------------------------------------------------------------+
| [code.codeChallengeMethod] | String   | One of 'plain' or 'S256'. Only present in PKCE requests.            |
+----------------------------+----------+---------------------------------------------------------------------+
| client                     | Object   | The client associated with the authorization code.                  |
+----------------------------+----------+---------------------------------------------------------------------+
| user                       | Object   | The user associated with the authorization code.                    |
+----------------------------+----------+---------------------------------------------------------------------+

For PKCE requests, see :ref:`PKCE#authorizationRequest`.

**Return value:**

An ``Object`` representing the authorization code and associated data.

+------------------------+--------+---------------------------------------------------------------+
| Name                   | Type   | Description                                                   |
+========================+========+===============================================================+
| code                   | Object | The return value.                                             |
+------------------------+--------+---------------------------------------------------------------+
| code.authorizationCode | String | The authorization code passed to ``saveAuthorizationCode()``. |
+------------------------+--------+---------------------------------------------------------------+
| code.expiresAt         | Date   | The expiry time of the authorization code.                    |
+------------------------+--------+---------------------------------------------------------------+
| code.redirectUri       | String | The redirect URI associated with the authorization code.      |
+------------------------+--------+---------------------------------------------------------------+
| [code.scope]           | String | The authorized scope of the authorization code.               |
+------------------------+--------+---------------------------------------------------------------+
| code.client            | Object | The client associated with the authorization code.            |
+------------------------+--------+---------------------------------------------------------------+
| code.client.id         | String | A unique string identifying the client.                       |
+------------------------+--------+---------------------------------------------------------------+
| code.user              | Object | The user associated with the authorization code.              |
+------------------------+--------+---------------------------------------------------------------+

``code.client`` and ``code.user`` can carry additional properties that will be ignored by *oauth2-server*.

**Remarks:**

::

  function saveAuthorizationCode(code, client, user) {
    // imaginary DB queries
    let authCode = {
      authorization_code: code.authorizationCode,
      expires_at: code.expiresAt,
      redirect_uri: code.redirectUri,
      scope: code.scope,
      client_id: client.id,
      user_id: user.id
    };
    return db.saveAuthorizationCode(authCode)
      .then(function(authorizationCode) {
        return {
          authorizationCode: authorizationCode.authorization_code,
          expiresAt: authorizationCode.expires_at,
          redirectUri: authorizationCode.redirect_uri,
          scope: authorizationCode.scope,
          client: {id: authorizationCode.client_id},
          user: {id: authorizationCode.user_id}
        };
      });
  }

--------

.. _Model#revokeToken:

``revokeToken(token)``
==================================

Invoked to revoke a refresh token.

This model function is **required** if the ``refresh_token`` grant is used.

**Invoked during:**

- ``refresh_token`` grant

**Arguments:**

+-------------------------------+----------+---------------------------------------------------------------------+
| Name                          | Type     | Description                                                         |
+===============================+==========+=====================================================================+
| token                         | Object   | The token to be revoked.                                            |
+-------------------------------+----------+---------------------------------------------------------------------+
| token.refreshToken            | String   | The refresh token.                                                  |
+-------------------------------+----------+---------------------------------------------------------------------+
| [token.refreshTokenExpiresAt] | Date     | The expiry time of the refresh token.                               |
+-------------------------------+----------+---------------------------------------------------------------------+
| [token.scope]                 | String   | The authorized scope of the refresh token.                          |
+-------------------------------+----------+---------------------------------------------------------------------+
| token.client                  | Object   | The client associated with the refresh token.                       |
+-------------------------------+----------+---------------------------------------------------------------------+
| token.client.id               | String   | A unique string identifying the client.                             |
+-------------------------------+----------+---------------------------------------------------------------------+
| token.user                    | Object   | The user associated with the refresh token.                         |
+-------------------------------+----------+---------------------------------------------------------------------+

**Return value:**

Return ``true`` if the revocation was successful or ``false`` if the refresh token could not be found.

**Remarks:**

``token`` is the refresh token object previously obtained through :ref:`Model#getRefreshToken() <Model#getRefreshToken>`.

::

  function revokeToken(token) {
    // imaginary DB queries
    return db.deleteRefreshToken({refresh_token: token.refreshToken})
      .then(function(refreshToken) {
        return !!refreshToken;
      });
  }

--------

.. _Model#revokeAuthorizationCode:

``revokeAuthorizationCode(code)``
=============================================

Invoked to revoke an authorization code.

This model function is **required** if the ``authorization_code`` grant is used.

**Invoked during:**

- ``authorization_code`` grant

**Arguments:**

+------------------------+----------+---------------------------------------------------------------------+
| Name                   | Type     | Description                                                         |
+========================+==========+=====================================================================+
| code                   | Object   | The code to be revoked.                                             |
+------------------------+----------+---------------------------------------------------------------------+
| code.authorizationCode | String   | The authorization code.                                             |
+------------------------+----------+---------------------------------------------------------------------+
| code.expiresAt         | Date     | The expiry time of the authorization code.                          |
+------------------------+----------+---------------------------------------------------------------------+
| [code.redirectUri]     | String   | The redirect URI of the authorization code.                         |
+------------------------+----------+---------------------------------------------------------------------+
| [code.scope]           | String   | The authorized scope of the authorization code.                     |
+------------------------+----------+---------------------------------------------------------------------+
| code.client            | Object   | The client associated with the authorization code.                  |
+------------------------+----------+---------------------------------------------------------------------+
| code.client.id         | String   | A unique string identifying the client.                             |
+------------------------+----------+---------------------------------------------------------------------+
| code.user              | Object   | The user associated with the authorization code.                    |
+------------------------+----------+---------------------------------------------------------------------+

**Return value:**

Return ``true`` if the revocation was successful or ``false`` if the authorization code could not be found.

**Remarks:**

``code`` is the authorization code object previously obtained through :ref:`Model#getAuthorizationCode() <Model#getAuthorizationCode>`.

::

  function revokeAuthorizationCode(code) {
    // imaginary DB queries
    return db.deleteAuthorizationCode({authorization_code: code.authorizationCode})
      .then(function(authorizationCode) {
        return !!authorizationCode;
      });
  }

--------

.. _Model#validateScope:

``validateScope(user, client, scope)``
==================================================

Invoked to check if the requested ``scope`` is valid for a particular ``client``/``user`` combination.

This model function is **optional**. If not implemented, any scope is accepted.

**Invoked during:**

- ``authorization_code`` grant
- ``client_credentials`` grant
- ``password`` grant

**Arguments:**

+------------+----------+---------------------------------------------------------------------+
| Name       | Type     | Description                                                         |
+============+==========+=====================================================================+
| user       | Object   | The associated user.                                                |
+------------+----------+---------------------------------------------------------------------+
| client     | Object   | The associated client.                                              |
+------------+----------+---------------------------------------------------------------------+
| client.id  | Object   | A unique string identifying the client.                             |
+------------+----------+---------------------------------------------------------------------+
| scope      | String   | The scopes to validate.                                             |
+------------+----------+---------------------------------------------------------------------+

**Return value:**

Validated scopes to be used or a falsy value to reject the requested scopes.

**Remarks:**

``user`` is the user object previously obtained through :ref:`Model#getAuthorizationCode() <Model#getAuthorizationCode>` (``code.user``; authorization code grant), :ref:`Model#getUserFromClient() <Model#getUserFromClient>` (client credentials grant) or :ref:`Model#getUser() <Model#getUser>` (password grant).

``client`` is the object previously obtained through :ref:`Model#getClient <Model#getClient>` (all grants).

You can decide yourself whether you want to reject or accept partially valid scopes by simply filtering out invalid scopes and returning only the valid ones.

To reject invalid or only partially valid scopes:

::

  // list of valid scopes
  const VALID_SCOPES = ['read', 'write'];

  function validateScope(user, client, scope) {
    if (!scope.split(' ').every(s => VALID_SCOPES.indexOf(s) >= 0)) {
      return false;
    }
    return scope;
  }

To accept partially valid scopes:

::

  // list of valid scopes
  const VALID_SCOPES = ['read', 'write'];

  function validateScope(user, client, scope) {
    return scope
      .split(' ')
      .filter(s => VALID_SCOPES.indexOf(s) >= 0)
      .join(' ');
  }

Note that the example above will still reject completely invalid scopes, since ``validateScope`` returns an empty string if all scopes are filtered out.

--------

.. _Model#verifyScope:

``verifyScope(accessToken, scope)``
===============================================

Invoked during request authentication to check if the provided access token was authorized the requested scopes.

This model function is **required** if scopes are used with :ref:`OAuth2Server#authenticate() <OAuth2Server#authenticate>`
but it's never called, if you provide your own ``authenticateHandler`` to the options.

**Invoked during:**

- request authentication

**Arguments:**

+------------------------------+----------+---------------------------------------------------------------------+
| Name                         | Type     | Description                                                         |
+==============================+==========+=====================================================================+
| token                        | Object   | The access token to test against                                    |
+------------------------------+----------+---------------------------------------------------------------------+
| token.accessToken            | String   | The access token.                                                   |
+------------------------------+----------+---------------------------------------------------------------------+
| [token.accessTokenExpiresAt] | Date     | The expiry time of the access token.                                |
+------------------------------+----------+---------------------------------------------------------------------+
| [token.scope]                | String   | The authorized scope of the access token.                           |
+------------------------------+----------+---------------------------------------------------------------------+
| token.client                 | Object   | The client associated with the access token.                        |
+------------------------------+----------+---------------------------------------------------------------------+
| token.client.id              | String   | A unique string identifying the client.                             |
+------------------------------+----------+---------------------------------------------------------------------+
| token.user                   | Object   | The user associated with the access token.                          |
+------------------------------+----------+---------------------------------------------------------------------+
| scope                        | String   | The required scopes.                                                |
+------------------------------+----------+---------------------------------------------------------------------+

**Return value:**

Returns ``true`` if the access token passes, ``false`` otherwise.

**Remarks:**

``token`` is the access token object previously obtained through :ref:`Model#getAccessToken() <Model#getAccessToken>`.

``scope`` is the required scope as given to :ref:`OAuth2Server#authenticate() <OAuth2Server#authenticate>` as ``options.scope``.

::

  function verifyScope(token, scope) {
    if (!token.scope) {
      return false;
    }
    let requestedScopes = scope.split(' ');
    let authorizedScopes = token.scope.split(' ');
    return requestedScopes.every(s => authorizedScopes.indexOf(s) >= 0);
  }

--------

.. _Model#validateRedirectUri:

``validateRedirectUri(redirectUri, client)``
================================================================

Invoked to check if the provided ``redirectUri`` is valid for a particular ``client``.

This model function is **optional**. If not implemented, the ``redirectUri`` should be included in the provided ``redirectUris`` of the client.

**Invoked during:**

- ``authorization_code`` grant

**Arguments:**

+-----------------+----------+---------------------------------------------------------------------+
| Name            | Type     | Description                                                         |
+=================+==========+=====================================================================+
| redirect_uri    | String   | The redirect URI to validate.                                       |
+-----------------+----------+---------------------------------------------------------------------+
| client          | Object   | The associated client.                                              |
+-----------------+----------+---------------------------------------------------------------------+

**Return value:**

Returns ``true`` if the ``redirectUri`` is valid, ``false`` otherwise.

**Remarks:**
When implementing this method you should take care of possible security risks related to ``redirectUri``.
.. _rfc6819: https://datatracker.ietf.org/doc/html/rfc6819

Section-5.2.3.5 is implemented by default.
.. _Section-5.2.3.5: https://datatracker.ietf.org/doc/html/rfc6819#section-5.2.3.5

::

  function validateRedirectUri(redirectUri, client) {
    return client.redirectUris.includes(redirectUri);
  }
