BipIO <a href="https://bip.io"><img align="right" width="48" src="https://2.gravatar.com/avatar/b4063abdf67036bdff26845fe2adcf69?d=https%3A%2F%2Fidenticons.github.com%2F7bad9441c4612b497d9d071c244f21cc.png&r=x&s=140" style="float:right"/></a>
=========

Welcome to the [BipIO](https://bip.io) API Server (Sansa 0.3)

BipIO is Billion Instructions Per I/O - For People and Robots.  

Imagine you can send a single standard payload and have a limitless host of API's orchestrate at your command.  

That's what BipIO does.

[![NPM](https://nodei.co/npm/bipio.png?downloads=true)](https://nodei.co/npm/bipio/)
[![Bitdeli Badge](https://d2weczhvl823v0.cloudfront.net/bipio-server/bipio/trend.png)](https://bitdeli.com/free "Bitdeli Badge")
----


BipIO is a highly parallel nodejs based API integration framework (iPaas).  It uses [graph](http://en.wikipedia.org/wiki/Directed_graph) 
based <a href="http://en.wikipedia.org/wiki/Pipeline_(software)">pipelines</a> or 'Bips' to create ephemeral endpoints, complex workflows and message hubs with 3rd party API's and
[RPC's](http://en.wikipedia.org/wiki/Remote_procedure_call).  Bips come in a variety of flavors for performing useful work - WebHooks/Sockets, Email, and Event Triggers.

It's a RESTful JSON API that supports account level namespacing and multiple domains ([fqdn](http://en.wikipedia.org/wiki/Fully_qualified_domain_name)) per account.  Clients authenticate over HTTP Basic.

BipIO can be installed alongside your existing open source app or prototype for out-of-band message transformation, feed aggregation, queuing, social network fanout or whatever you like, even on your Rasberry Pi.

Follow <a href="https://twitter.com/bipioapp" class="twitter-follow-button" data-show-count="false">@bipioapp</a> on Twitter for regular news and updates, or `/join #bipio` on [Freenode IRC](https://freenode.net).

For a technical deep-dive, don't be afraid to use [The Wiki](https://github.com/bipio-server/bipio/wiki) - it's open to contributors

### Concept

If you're familiar with Yahoo Pipes, IFTTT, Zapier, Mulesoft - the concept is a little similar.   BipIO is a message transformation hub for connecting API's and creating an internet of things that matter to you.  Over time and as more people use the system it builds a corpus of transformation data that serve as 'hints' for use by other community members in their own integrations.  Complete configurations can also be shared openly by different users and systems allowing for the instant installation of common workflows.

Bipio is dynamic, flexible, fast, modular, opinionless and gplv3 open source.

![concept](https://bip.io/static/img/docs/bip_concept.png)



#### Bips

[Bips](https://bip.io/docs/resource/rest/bip) are graph structures which transform messages between adjacent [Channels](https://bip.io/docs/resource/rest/channel) and chain outputs to inputs indefinitely across disparate 'cloud' services. The structures also contain metadata defining the flavor, lifespan and overall characteristics of the endpoint or trigger.

Some of their characteristics include :

 - dynamic or automatically derived naming
 - pausing or self-destructing after a certain time or impressions volume
 - binding to connecting clients with soft ACLs over the course of their 'life'
 - able to be reconfigured dynamically without changing a client implementation
 - infinitely extensible, from any channel to any other channel.
 - can serve (render) protected channel content while inheriting all of the above characteristics

It's a fairly large topic, find out more in [the wiki](https://github.com/bipio-server/bipio/wiki/Bips).

#### Channels

Channels are pointers to discrete actions provided by 3rd party API's and services. They are reusable entities which perform a discrete unit of work and emit a predictable result.

The collection of channels you create becomes something like a swatch from which you can orchestrate complex API messaging patterns.  When dropped onto a Bip's graph, a channels export becomes the next adjacent channels transformed import, which can be chained indefinitely.

Channels are instantiated from service containers called Pods.  Pods only concern is providing a set of possible actions, and doing that well.

Channels can store, track, serve or transform content and messages as part of a pipeline or in autonomous isolation.  

The server ships with a few handy '[Pods](https://github.com/bipio-server/bipio/wiki/Pods)' which you can use right away - Email, Text/HTML/Markdown Templating, Flow Control, Syndication, Web Hooks, Time.  Extra Pods can be found in the [GitHub Repository](https://github.com/bipio-server/bipio) - to install it's just :

    npm install bip-pod-{pod-name}
    ./tools/pod-install.js -a {pod-name}
  
And follow the instructions, or feel free to [craft your own](https://github.com/bipio-server/bipio/wiki/Creating-Pods).


##### Simple Integrations

Here's a quick example.  Lets say I have a private email address that I want to protect or obfuscate - I could use an SMTP Bip to
create a temporary relay which will forward emails for 1 day only.  

**Here's how :**

Create an SMTP Forwarder Channel to email me with any messages it receives :
```
POST /rest/channel
{
 action : "email.smtp_forward",
 name : "Helo FuBa"
 config : {
   rcpt_to : "foo@bar.net"
 }
}

RESPONSE
{
 id : "206fe27f-5c98-11e3-8ad3-c860002bd1a4"
}
```

... I can then build the relay with a SMTP Bip having a single edge pointing to the the generated Channel ID :

```
POST /rest/bip
{
 type : "smtp",
 hub : {
   "source" : {
      edges : [ "206fe27f-5c98-11e3-8ad3-c860002bd1a4" ],
      transforms : {
        "206fe27f-5c98-11e3-8ad3-c860002bd1a4" : {
          "subject" : "[%source#subject%]",
          "body_html" : "[%source#body_html%]",
          "body_text" : "[%source#body_text%]",
          "reply_to" : "[%source#reply_to%]",
        }
      },
     _note : "^^ Transforms aren't mandatory, but here for illustration - you only need an edge"
   }
 },
 end_life : {
   imp : 0,
   time : '+1d'
 },
 note : "No name, no problem.  Let the system generate something short and unique"
}

RESPONSE
{
 name : "lcasKQosWire22"
 _repr : "lcasKQosWire22@yourdomain.net"
}
```

And thats it. There's actually a little [chrome extension](http://goo.gl/ZVIkfr) which does just this for web based email forms.

For an extra credit example, I could store attachments arriving on that email address straight to dropbox by just adding an edge - check out
how in the [cookbook](https://github.com/bipio-server/bipio/wiki/Email-Repeater,-Dropbox-Attachment-Save)



## Requirements

  - [Node.js >= 0.10.15](http://nodejs.org) **API and graph resolver**
  - [MongoDB Server](http://www.mongodb.org) **data store**
  - [RabbitMQ](http://www.rabbitmq.com) **message broker**

SMTP Bips are available out of the box with a Haraka plugin.  Configs under [bipio-contrib/haraka](https://github.com/bipio-server/bipio-contrib).

  - [Haraka](https://github.com/baudehlo/Haraka)

## Installation

#### via npm (global)

    sudo npm install -g bipio
    bipio

#### via npm (local)

    sudo npm install bipio
    cd node_modules
    npm start

#### via git

    git clone git@github.com:bipio-server/bipio.git
    cd bipio
    npm install
    node . (or `npm start`)

When setting BipIO up for the first time, the install process will enter interactive mode, saving to the path of NODE_CONFIG_DIR environment variable,if set (otherwise, just config/{environment.json}.

    export NODE_CONFIG_DIR=<path_to_your_config_directory>

Be sure to have a MongoDB server and Rabbit broker ready and available before install.  Otherwise, follow the prompts
during the install process to get a basically sane server running that you can play with.

For Ubuntu users, a sample upstart script is supplied in config/upstart_bip.conf which should be copied to 
/etc/init and reconfigured to suit your environment.

If you have a more complex deployment environment and the packaged sparse config doesn't suit, don't worry!  Set the environment variable BIPIO_SPARSE_CONFIG to the path of your preferred config file, and it will use that instead.

For a non-interactive setup (ie: make install without any user interaction) - set environment variable HEADLESS=true

## Updating

Updating BipIO via `npm` will resolve any new dependencies for you, however if you're checking out from the repository 
directly with `git pull` you may need to manually run `npm install` to download any new dependencies (bundled pods, for example).

If you're going the `git pull` route and want to save this step, create a git 'post merge' hook by copying it from `./tools` like so :

    mkdir -p .git/hooks
    cp ./tools/post-merge .git/hooks
    chmod ug+x .git/hooks/post-merge

This will automatically install any missing dependencies every time you `git pull`

### Crons

Periodic tasks will run from the server master instance automatically, you can find the config
in the `config/{environment}.json` file, keyed by 'cron'.  

* stats - network chord stats, every hour
* triggers - trigger channels, every 15 minutes
* expirer - bip expirer, every hour

To disable a cron, either remove it from config or set an empty string.

To have these crons handled by your system scheduler rather than the bipio server, disable the crons
in config as described.  Wrapper scripts can be found in ./tools for each of stats (`tools/generate-hub-stats.js`), 
triggers (`tools/bip-trigger.js`) and expirer (`tools/bip-expire.js`).

Here's some example wrappers.

#### Trigger Runner

Cron:
    */15 * * * * {username} /path/to/bipio/tools/trigger-runner.sh

trigger-runner.sh :

    #!/bin/bash
    # trigger-runner.sh
    export NODE_ENV=production
    export HOME="/path/to/bipio"
    cd $HOME (date && node ./tools/bip-trigger.js ) 2>&1 >> /path/to/bipio/logs/trigger.log

#### Expire Runner

Cron:
    0 * * * * {username} /path/to/bipio/tools/expire-runner.sh

expire-runner.sh :

    #!/bin/bash
    # expire-runner.sh
    export NODE_ENV=production
    export HOME="/path/to/bipio"
    cd $HOME (date && node ./tools/bip-expire.js ) 2>&1 >> /path/to/bipio/logs/cron_server.log

#### Stats Runner

Cron:
    */15 * * * * {username} /path/to/bipio/tools/stats-runner.sh

stats-runner.sh :

    #!/bin/bash
    # stats-runner.sh
    export NODE_ENV=production
    export HOME="/path/to/bipio"
    cd $HOME (date && node ./tools/generate-hub-stats.js ) 2>&1 >> /path/to/bipio/logs/stats.log

### Monit Config

/etc/monit/config.d/bipio.conf

    #!monit
    set logfile /var/log/monit.log

    check process node with pidfile "/var/run/bip.pid"
        start program = "/sbin/start bipio"
        stop program  = "/sbin/stop bipio"
        if failed port 5000 protocol HTTP
            request /
            with timeout 10 seconds
            then restart


## Visualization

The BipIO server software provides an orchestration API and is distributed headless.  For visual tools, sign in to [bipio](https://bip.io) to mount your local install from your browser under My Account > Mounts > Create Mount.  

![Server Mount](https://bip.io/static/img/docs/server_mount.png)

BipIO does not provide any load balancing beyond [node-cluster](http://nodejs.org/api/cluster.html).  It can provide SSL termination but this is unsuitable for a production environment.  If you need SSL termination this should ideally be delegated to the forward proxy of your choice such as Nginx, Apache, HAProxy etc.

#### Mounting Security Notes

Be sure to answer 'yes' to the SSL question during setup to install a self signed SSL certificate.  This will avoid any browser security restrictions when mounting your server via the hosted website.  You *must* visit your bipio server in a browser first and accept the self signed certificate, or the mount may not work eg : `https://localhost:5000/status`

## Developing and Contributing

A healthy contributor community is great for everyone! Take a look at the [Contribution Document](https://github.com/bipio-server/bipio/blob/master/CONTRIBUTING.md) to see how to get your changes merged in.

## Support

Please log issues to the [repository issue tracker](https://github.com/bipio-server/bipio/issues) on GitHub.  

## License

[GPLv3](http://www.gnu.org/copyleft/gpl.html)

Our open source license is the appropriate option if you are creating an open source application under a license compatible with the GNU GPLv3. 

If you'd like to integrate BipIO with your proprietary system, GPLv3 is likely incompatible.  To secure a Commercial OEM License for Bipio,
please [reach us](mailto:hello@bip.io)
