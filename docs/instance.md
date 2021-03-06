[Table of contents](./README.md#table-of-contents)

Instances
=========

**TODO** it's still a work in progress that needs to be completed.

A single cozy-stack can manage several instances. Requests to different instances are identified through the `Host` HTTP Header, any reverse proxy placed in front of the cozy-stack should forward this header.

To simplify development, a `dev` instance name is used when no Host Header is provided. This behaviour will be kept when the stack is started in dev mode but will be blocked in production environment.

**Exemple:**

- `curl -H "Host: bob.cozycloud.cc" localhost:8080` → `localhost:5984/bob-cozycloud.cc`
- `curl -H "Host: alice.cozycloud.cc" localhost:8080` → `localhost:5984/alice-cozycloud.cc`
- `curl localhost:8080` → `localhost:5984/dev` (in dev mode only)

Creation
--------

An instance is created on the command line:

```sh
$ cozy-stack instances add <domain>
```
With some possible additional options

- `--locale <lang>`
- `--email <email>`
- `--environment <dev/test/production>`
- `--apps <app1,app2,app3>`
- `--home <cozy-home>`
- `--onboarding <cozy-onboarding>`
- `--registry https://registry.cozycloud.cc`

It registers the instance in a global couchdb database `global/instances`
```json
{
    "hostname": "example.cozycloud.cc",
    "dbprefix": "example-clozycloud-cc/",
    "fsroot": "/var/lib/cozy/example.cozycloud.cc/fs/"

}
```

and creates the proper databases ($PREFIX/$DOCTYPE) for these doctypes:

- `io.cozy.apps`
- `io.cozy.manifests`
- `io.cozy.files`
- `io.cozy.folders`
- `io.cozy.notifications`
- `io.cozy.settings`

Then, it creates the following indexes for these doctypes :

- `io.cozy.apps : ["slug"]`
- `io.cozy.manifest : ["url"]`
- `io.cozy.files & io.cozy.folders: ["FolderId", "name"], ["FolderId", "modifiedDate"]`
- **TODO :** complete this list

Then, it creates some folders:

- `/`, with the id `io.cozy.folders-root`
- `/Apps`, with the id `io.cozy.folders-apps`
- `/Documents`, with the id `io.cozy.folders-documents`
- `/Documents/Downloads`, with the id `io.cozy.folders-downloads`
- `/Documents/Pictures`, with the id `io.cozy.folders-pictures`
- `/Documents/Music`, with the id `io.cozy.folders-music`
- `/Documents/Videos`, with the id `io.cozy.folders-videos`

**The ids are forced to known values:**  even if these folders are moved or
renamed, they can still be found for the permissions.

**The names are localized:** If a locale is provided through the CLI, the folders will be created with
names in this locale. Otherwise, theses folders will be created in english and renamed to localized name the first time the locale is set (during onboarding).

Then it creates the basic settings

- `owner_email` if an email was provided through the CLI
- `locale` if a locale was provided through the CLI
- `home_app` slug of the app to use as home, `cozy-home` by default
- `onboarding_app` slug of the app to use as onboarding, `cozy-onboarding` by default
- `registry_url` URL of the registry place to fetch manifests of installable applications.

Settings are created as named id in the `$PREFIX/io.cozy.settings` database.
During onboarding, the fields will be prefilled with these value if they were provided.

Finally, default applications are installed in the following order :

- the `home` and `onboarding` applications are installed according to provided URL or cozy defaults.
- If the `apps` CLI param is given, all these apps are installed
- If the environment is set to `dev`, some devtools are installed


--------------------------------------


Renaming
--------

An instance is renamed through the command line.

```sh
$ cozy-stack instances rename <olddomain> <newdomain>
```

Renaming an instance only change the HostName in global/instances base.


---------------------------------------

Destroying
----------

An instance is destroyed through the command line.
A confirmation is asked from the CLI user unless the --yes flag is passed

```sh
$ cozy-stack instances destroy <domain>
```
