# EcoSOAS
Ecosystem on a stick

Featuring:
 - PostgreSQL as database
 - nginx as web server
 - Keycloak for user management
 - Gitea for git
 - Drone as CI/CD
 - Vaultwarden as password manager
 - Much more TBD

# Current issues
Sensitive data inside compose file

# Setup
Not everything can be set up, so here's step by step

## Variables:

KEYCLOAK_REALM_NAME — we will create our realm in Keycloak

DB_GITEA_USERNAME — PostgreSQL user for gitea
DB_GITEA_PASSWORD — PostgreSQL password for gitea
DB_GITEA_DBNAME — PostgreSQL database name for gitea

DB_KEYCLOAK_USERNAME — PostgreSQL user for Keycloak
DB_KEYCLOAK_PASSWORD — PostgreSQL password for Keycloak
DB_KEYCLOAK_DBNAME — PostgreSQL database name for Keycloak

GITEA_ORIGIN — Hostname(with port) of Gitea instance 
GITEA_KEYCLOAK_AUTH_SOURCE_NAME — Authentication source name for Keycloak in Gitea
KEYCLOAK_GITEA_CLIENTID — Client ID for Gitea in Keycloak

KEYCLOAK_GITEA_CLIENTID — Client ID for Gitea in Keycloak
KEYCLOAK_GITEA_CLIENT_SECRET — Client Secret for Gitea in Keycloak (non user-defined)

1. `docker-compose up postgres -d`
    1. Run the following statements against `postgres` db:
       - CREATE ROLE `DB_KEYCLOAK_USERNAME` NOSUPERUSER NOCREATEDB NOCREATEROLE NOINHERIT LOGIN NOREPLICATION NOBYPASSRLS PASSWORD `DB_KEYCLOAK_PASSWORD`;
       - CREATE ROLE `DB_GITEA_USERNAME` NOSUPERUSER NOCREATEDB NOCREATEROLE NOINHERIT LOGIN NOREPLICATION NOBYPASSRLS PASSWORD `DB_GITEA_PASSWORD`;
       - CREATE DATABASE `DB_KEYCLOAK_DBNAME` WITH OWNER = `DB_KEYCLOAK_USERNAME`
       - CREATE DATABASE `DB_GITEA_DBNAME` WITH OWNER = `DB_GITEA_USERNAME`
    1. Change active database to `DB_KEYCLOAK_DBNAME`
       -  ALTER SCHEMA public OWNER to `DB_KEYCLOAK_USERNAME`
    2. Change active database to `DB_GITEA_DBNAME`
       -  ALTER SCHEMA public OWNER to `DB_GITEA_USERNAME`
2. Stop the database container with `ctrl+C`
3. `docker-compose up keycloak -d`
4. Open up keycloak website (see services/keycloak/ports for port, usually localhost:8080) # TODO: this URL
    1. Create new realm (`KEYCLOAK_REALM_NAME`)
    2. In this new realm, go to Clients, add new
        1. Client type: `OpenID Connect`
        2. Client ID: `KEYCLOAK_GITEA_CLIENTID`
        3. Next
        4. Client authentication: `on`
        5. Authorization: `off` (default)
        6. Authentication flow: `Standard flow`, `Direct access grants` (default)
        7. Next
        8. Valid redirect URIs: http://`GITEA_ORIGIN`/user/oauth2/`GITEA_KEYCLOAK_AUTH_SOURCE_NAME`/callback # TODO: this URL
        9. Go to Credentials tab (it is visible only if you enabled Client authentication)
        10. Remember `Client secret` value (it is `KEYCLOAK_GITEA_CLIENT_SECRET` variable)
    1.  In Users:
        1.  Click "Add user"
        2.  Fill username and email
        3.  Click "Create"
        4.  Go to "Credentials"
        5.  Click "Set password"
        6.  Temporary: `off`
        7.  Click "Save"
5. `docker-compose up gitea -d`
6. Open up gitea website (see services/gitea/ports for port, usually localhost:3000)
    1. On the install page, disable "enable OpenID login" # TODO: find out what is real name of this setting
    2. Click install
    3. Wait
    4. It'll likely show the 500 error page, just refresh the page
    5. register admin account
    6. Go to administration (`http://localhost:3000/admin`) # TODO: this URL
    7. Go to "Authentication sources" tab, click "Add Authentication Source"
        1. Authentication Type: `OAuth2`
        2. Authentication Name: `GITEA_KEYCLOAK_AUTH_SOURCE_NAME`
        3. OAuth2 Provider: `OpenID Connect`
        4. Client ID (Key): `KEYCLOAK_GITEA_CLIENTID`
        5. Client Secret: `KEYCLOAK_GITEA_CLIENT_SECRET` that you've got from Keycloak
        6. OpenID Connect Auto Discovery URL: http://a-kuzmichev.local:8080/realms/`KEYCLOAK_REALM_NAME`/.well-known/openid-configuration # TODO: this URL
        7. [optionally] Icon URL: `http://localhost:8080/resources/y8e4y/admin/keycloak.v2/logo.svg` # TODO: this URL
        8. This Authentication Source is Activated: ✅
        9. Click "Add Authentication Source"
    8. Open new private tab on `http://localhost:3000/`, click "Sign in"
    9. Click on the picture in "Sign in With [picture]"
    10. Enter login/password of user you created in Keycloak
    11. On Gitea, choose "Register New Account" and verify that username and email are correct
    12. Click "Complete account"