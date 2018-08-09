Buildbot Setup
==============

We assume that the `git-ci` repo setup has already happened for a CI server.

Setup Buildbot
--------------

Install the most up-to-date Buildbot software, and the `txrequests` package:

```sh
pip install --user 'buildbot[bundle]'
pip install --user 'txrequests'
```

Create a new master and symlink the appropriate config:

```sh
mkdir buildbot
cd buildbot
buildbot create-master master
cd master
ln -sf ../../src/git-ci/buildbot/master.cfg
```

Fill in all the fields marked as `{###FIELD_NAME###}` in `master.cfg`.

Create a new worker.
Look at `c['workers']` in `buildbot/master.cfg` of `master/master.cfg` for `WORKER_NAME`, `WORKER_PASS`.
Look at `c['protocols']` for `PORT`.
Edit the appropriate worker information files.

```sh
cd ~/buildbot
buildbot-worker create-worker WORKER_NAME localhost:PORT WORKER_NAME WORKER_PASS
vim ~/WORKER_NAME/info/admin
vim ~/WORKER_NAME/info/host
```

Start the master then the worker:

```sh
buildbot        start ~/buildbot/master
buildbot-worker start ~/buildbot/WORKER_NAME
```

Setup nginx Reverse Proxy
-------------------------

The supplied `nginx/nginx.conf` and `nginx/sites-enabled/default` setup reverse proxy for Buildbot.
Copy them into the correct locations in `/etc/nginx`, then fill in the field `{###SERVER_NAME###}` in file `nginx/sites-enabled/default` to be the URL (or IP) the server is at.
Once setup, verify that the nginx config is good with:

```sh
sudo nginx -t
```

Given that the config is valid, enabled the nginx daemon.

### Optional

Install [certbot](https://certbot.eff.org/) and the [nginx certbot plugin](https://certbot.eff.org/lets-encrypt/ubuntuxenial-nginx.html).
Fill in the commented out certbot sections of `nginx/sites-enabled/default` using the certbot nginx plugin.
Note that you must have a domain name for this to work.

```sh
sudo certbot --nginx
```

Double-check the nginx config:

```sh
sudo nginx -t
```

It's also recommended that you set up certbot auto-renewal.

Setup Github Webhook/Status Check
---------------------------------

In Github, add the webhook with these parameters (fill in `SERVER-URL`):

Payload URL: https://SERVER-URL/change_hook/github
Secret:      contents of `c['www']['change_hook_dialects']['github']['secret']`
Events:      Pull Request (under "Let me select individual events").

Under Settings -> Branches, protect relevant branches by adding the `buildbot/PROJECT` status-based branch protection.

Github OAuth Setup
------------------

This enables logging into Buildbot via a "Login with GitHub" link (to do things like forcing rebuilds/manually triggering builds).

Setup OAuth with your user (or a dummy user for reporting CI information on repos in an organization).
`CLIENT_ID`, `CLIENT_SECRET`, and `ORGANIZATION_NAME` come from:

```
c['www'] = { 'auth'  : util.GitHubAuth(CLIENT_ID, CLIENT_SECRET)
           , 'authz' : util.Authz( roleMatchers = [ util.RolesFromGroups() ]
                                 , allowRules   = [ util.AnyControlEndpointMatcher(role = "ORGANIZATION_NAME")
                                                  ]
                                 )
           }
```

TODO
====

-   Containers
-   systemd unit
    https://github.com/buildbot/buildbot-contrib/blob/master/master/contrib/systemd/buildbot.service
-   document `certbot` setup as well
