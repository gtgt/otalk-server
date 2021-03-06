# Otalk XMPP Server

Any XMPP server that supports websockets would work, but Prosody also supports
some extra features that makes Otalk nicer to use, like message archiving.

We do plan to create a packaged Docker image soon.


## Installation

1. Install Prosody. We want to use message archiving, which requires trunk for now.

        sudo apt-get install prosody-trunk

   You may need to follow the [instructions to install the Prosody PPA first](http://prosody.im/download/package_repository).

   Once a 0.10 candidate is released, we'll switch to that instead of trunk to avoid breaking changes.

2. Install additional dependencies needed by MAM and WebSocket

        sudo apt-get install lua-zlib
        sudo apt-get install lua-sec-prosody
        sudo apt-get install lua-dbi-sqlite3
        sudo apt-get install liblua5.1-bitop-dev
        sudo apt-get install liblua5.1-bitop0

3. Install the included modules

        sudo cp -r mod_carbons /usr/lib/prosody/modules
        sudo cp -r mod_mam /usr/lib/prosody/modules
        sudo cp -r mod_smacks2 /usr/lib/prosody/modules
        sudo cp -r mod_smacks3 /usr/lib/prosody/modules
        sudo cp -r mod_websocket /usr/lib/prosody/modules
        sudo cp -r mod_http_altconnect /usr/lib/prosody/modules
        sudo cp -r mod_turncredentials /usr/lib/prosody/modules

4. Configure Prosody

   First edit the included template config to replace the HOST value, and set any other desired options.

        sudo cp prosody.cfg.lua /etc/prosody/

5. Allow access to port 5281. Proxying to hide the port would be best (eg, use `wss://HOST/xmpp-websocket`).

   If you don't proxy the WS connections, be sure to visit https://HOST:5281/xmpp-websocket first so that
   any client certificate requests are fulfilled. Otherwise, connecting to otalk might fail because the
   browser closes the websocket connection if prompted for client certs.

6. Configure restund according to restund/README.md. Optional, but highly recommended.


By default, You will need to ensure that these ports are open on your server:

- 5222 (XMPP client to server connections)
- 5269 (XMPP server to server connections)
- 5280/5281 HTTP and WebSocket connection (5281 for SSL versions)
- 3478 UDP (STUN/TURN)

You should also setup DNS SRV records:

- `_xmpp-client._tcp.HOST 3600 IN SRV 0 10 5222 HOST`
- `_xmpp-server._tcp.HOST 3600 IN SRV 0 10 5269 HOST`

If you use the `mod_http_altconnect` module, Otalk will be able to auto-discover the WebSocket connection
endpoint for your server, if you make https://HOST/.well-known/host-meta served by Prosody.

One way to do this is to make Prosody act as your HTTP server. An example nginx config for doing that
is included.


## To use &yet authentication (optional)

**NOTE:** This is intended for use by otalk.im as a default authentication
method for people who already have Andyet accounts to bootstrap the
process and experience of using Otalk. If you're running your own private
XMPP server, this is not needed, and you can use any of the various
authentication backends provided by Prosody.

1. Install external auth module

        sudo cp -r mod_auth_external /usr/lib/prosody/modules

2. Modify Prosody config

        VirtualHost "HOST"
            authentication = "external"
            external_auth_command = "andyet-prosody-auth"

3. Install `andyet-prosody-auth`

        npm install -g andyet-prosody-auth

4. Create `/etc/prosody/andyet.json`. 

        {
            "bucker": {
                "file": {
                    "filename": "/var/log/prosody/auth.log"
                }
                "console": false
            },
            "andyetAuth": {
                "id": "CLIENT ID",
                "secret": "CLIENT SECRET"
            },
            "andyetAPIs": {
                "apps": "https://apps.andyet.com",
                "shippy": "https://api.andbang.com"
            }
        }

    **NOTE**: the `"console": false` configuration MUST be included for bucker so that log output does not get sent to Prosody instead of the success/failure tokens.
