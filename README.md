# Minimal single-machine deployment shell script

These scripts can spawn a machine and deploy a web application on it, along
with dependencies like a runtime (e.g. Node.js), a reverse proxy (including
https termination using an letsencrypt certificate) and finally also a database
(MySQL). It can also install onto an existing machine as long as it can ssh into
"root@IP".

Exactly what is installed and how it is configured depends on a deployment
config json file. A sample file is provided [here](sample-deployment-config.json).

I've used these scripts to setup a machine with haproxy, node.js, MySQL and a
small node web application. It's also possible to write the deployment config
so that it installs Traefik instead of haproxy, and it would be quite straight
forward to add support for databases other than MySQL as well.

Note that you can have one deployment config per web application you have, and
you can then run ```./deploy whatever.json MACHINE_IP``` multiple times to
install multiple web applications on the same machine. The second time you
install a web application on a machine it will take much less time because then
haproxy is already installed, all the basics have already been setup etc.

These scripts are just the minimum of what I needed to get my web app running.
I typically deploy onto a very low-end droplet at Digital Ocean because it's
cheap. There are many reasons why it's a bad idea to run your reverse proxy, app
and your database on the same host. In many cases you're better of using Ansible,
Habitat and/or Kubernetes or something (depending on what usecases you have).
That said, you are of course free to use this script yourself if you want.

# How to use

Before you begin you need to have an ssh key setup at ```~/.ssh/id_rsa*```. The
scripts will allow this key to login as both admin and also as the app user
(for deployment of application code later on).

```shell
# spawn for example 33.44.55.66 at cloud vendor
# make sure "ssh root@33.44.55.66" works
$ ./deploy your-app.json 33.44.55.66
```

Or if you want to spawn a new Digital Ocean machine; first install ```doctl```
and make sure it is available on your path, then run:

```shell
$ ./spawn-deploy your-app.json droplet-name-for-app
```

Doing the above will typically being your domain online and you can access it
via https but haproxy will send back a 503 because the app is not running yet.
To upload the web application files and start the web application I typically
write a separate little script similar to this:

```shell
#!/bin/bash
if [ ! -f "package.json" ] || [ "$(cat package.json | jq -r .name)" != "foobar" ]; then
        echo "ERROR: current directory is not a FOOBAR project root."
        exit 1
fi

npm run build
rsync -avz -e ssh --progress --exclude config.json --exclude node_modules * foobar_app_user@webapps:/opt/foobar_app_dir/
ssh -t foobar_app_user@webapps "cd /opt/foobar_app_dir ; npm install ; sudo systemctl restart foobar_app"
```

# License

MIT
