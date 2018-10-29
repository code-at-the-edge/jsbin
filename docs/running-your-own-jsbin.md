# Run JS Bin locally

JS Bin is a [Node](http://nodejs.org) project.

JS Bin is not always easy to install in an up-to-date environment, because it depends on an older version of node.js and some of the other code has also suffered bitrot. In these update instructions, we try to convey what we learned during our installation process.

## System Requirements

While it may be possible to install JS Bin on a Windows computer, we assume you are using a *nix system. This distribution is intended for eventual deployment to a mature Linux distribution on ARM-based systems such as the Raspberry Pi.

## Installation

### Set up Node.js

Start by installing [nvm](https://github.com/creationix/nvm#installation), the node version manager. Follow the detailed instructions in the nvm README. Then follow the [usage instructions](https://github.com/creationix/nvm#installation), being sure to install **v7**:

`nvm install v7`

**It's possible other versions of node, such as lts-carbon, will still run; but v7 is the only one that is currently confirmed.**

If you are not using node for any other projects, then simply set the default node version to `7` with:

`nvm alias default v7`

Otherwise, in any shell you are using to work on this project, simply type 

```sh
nvm use
```

from the root project directory, and version 7 of node eshould be activated. 

### Clone Repo and install dependencies

Clone the project from github:

    $ git clone git@github.com:remy/jsbin.git
    $ cd jsbin
    $ npm install --dev
    $ npm install -g grunt-cli

This will install a number of files in the `node_modules` directory, then install the `grunt-cli` package to your system; the process will take a few minutes.

### Set up Mysql

You will need a working installation of mysql. On Debian-based systems just do this:

``` shell
apt-get install mysql-server mysql-client
```
You'll need to configure the server during installation, and then create a new user and password: 

``` shell
$ mysql -u root -p 
mysql> CREATE USER 'jsuser'@'localhost' IDENTIFIED BY 'jsbin';
mysql> Ctrl-D

```
This creates a new mysql accout "jsuser", with password "jsbin". If you change these values (and you probably should!) be sure to take your changes into account below. 

Now create the database and grant permissions to the new user:

``` shell
$ mysql -u root -p < ./build/full-db-v3.mysql.sql
$ mysql -u root -p
mysql> GRANT ALL PRIVILEGES ON jsbin TO 'jsuser'@'localhost'    WITH GRANT OPTION;
mysql> Ctrl-D
```
The database should now be ready to go!

## Configuration

The JS Bin configuration is stored in a JSON file. Our working config is stored in `config.local-working.json`, and we recommend starting from that file rather than from the upstream `config.default.json`. Copy the file to `config.local.json`:

    cp config.local-working.json config.local.json

There are a number of options that you can change to configure JS Bin to your needs. The property names, meaning and possible values have been listed below:

* env: `development` or `production`

This is whether you want to run in development mode (recommended) or production. Production mode requires that you build a single JavaScript file for JS Bin to run from. This is only required if you plan to run JS Bin as a service to a larger public audience. Building for production is detailed in a section later on.

* url: Object - detailed next

This is an object controlling how URLs are generated in JS Bin. If you plan to run JS Bin from a subdirectory, you must change these settings.

* url.host: The host you will access JS Bin on, eg. `kcc` is what we have used in the field. If you want to include a port number here you should do, eg. `jsbin.dev:8000` will request JS Bin over port 8000.
* url.prefix: `/` or your subdirectory that JS Bin should be accessed from. For instance, if you run JS Bin under `http://remysharp.com/jsbin` the `url.prefix` value must be `/jsbin/`.
* url.ssl: `false` or `true` whether you want to run JS Bin over SSL. It's not easy to run ssl from localhost, so we don't recommend changing this.
* url.static: `false` or a url, such as `http://static.jsbin.dev:8000/jsbin/`. This will control where static assets are served from. If the value is `false`, they will be served from the same path as dynamic content.

### Custom config file location

To start JS Bin with a config file from another location, set the `JSBIN_CONFIG` environment variable as a path to the custom file. The path should be absolute, or relative to your current working directory:

    $ JSBIN_CONFIG=~/config.local.json jsbin

### Running behind a proxy

This is unlikely to affect users of this fork.

JS Bin will run behind a proxy, indeed the production version of JS Bin is behind a proxy.

The `PORT` on the command line takes precedence over the config variable. This means in your config, you set the url to be the user facing port (typically port 80, so no port required), and then JS Bin will listen on the port you gave at the environment level.

So to proxy jsbin.com to a localhost:8000 (using something like nginx to do the proxying), the config would look like (this snippet of `config.local.json`):

```json
  "url": {
    "host": "jsbin.com",
    "prefix": "/",
    "ssl": false
  },
```

Note that in the above config, `jsbin.com` is what is used in the HTML and JavaScript, so this is the *client facing url*. Next, running JS Bin behind a proxy is as simple as:

    $ PORT=8000 JSBIN_CONFIG=~/config.local.json jsbin

Now the `jsbin` node process is listening on port 8000, but the client facing urls are all port 80.

### Building for production

JS Bin's build process uses Grunt, so assuming you've cloned a copy, you will need the dev dependancies and the grunt cli:

    $ npm install -g grunt-cli
    $ npm install --dev
    $ grunt build

This will generate the `public/js/prod/` directory and read the version in the `package.json` file to generate to build a number of files:

1. jsbin-<version>.js - the uncompressed, concatenated version of all the scripts from `/scripts.json`
2. jsbin-<version>.min.js - the production compressed version of jsbin, used in the editor
3. jsbin.map.json - the sourcemaps file (useful for debugging in live)
4. runner-<version>.js - the runner script, used to generate the output of the user's code in an iframe
5. runner-<version>.min.js - the production version of the runner

Finally, ensure either the config.local.json's `env` property is set to "production" or you can run JS Bin in production via the environment:

    $ NODE_ENV=production node .

And that's it.


## End-to-end tests
JS Bin has end-to-end tests. The tests can be found in `tests/e2e` folder. 

Running the tests (JS Bin has to be running on port http://localhost:3000): 

    $ npm run e2e
