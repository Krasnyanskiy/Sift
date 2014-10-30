![Sift](https://github.com/Mojang/Sift/blob/master/logo.png) 

[![Build Status](https://travis-ci.org/Mojang/Sift.svg?branch=master)](https://travis-ci.org/Mojang/Sift)

What is sift? A __lightweight__ and __easy-to-use__ tool for accessing your clouds!

## What does it do?

Sift simply does the following steps:

- Gathers all of your instances from different cloud providers and different accounts that are configured
- Filters the servers based on the provided query (if any)
- Executes any command on the result of the previous step (if provided). The default command is `ssh`

## Features

Sift supports the following (more expected to come):

- Add as many _cloud providers_ you need!
- Add as many _accounts_ you need!
- Use our simple and easy query language to build _powerfull queries_ that can be used to filter results from all providers
- Execute any _shell commands_ on any set of servers
- Define _aliases_ for different tasks you need to do so you don't need to type out everything everytime
- Flexible _configuration_


## Supported Cloud Providers

Current cloud providers we support at the moment:

- Amazon
- Digital Ocean


## How to install

`sift` is written in `node.js`. You can install it by using `npm` like this:

```bash
npm install -g cloud-sift
```

### Build from source

If you need the latest features (could be unstable) you can build from source by running the following:

```bash
git clone git@github.com:Mojang/Sift.git
cd Sift
npm install
sudo npm link
```

## How to run

### Sample `.sift.json` file

`.sift.json` file is created in your home directory the first time you run `sift`. You can then edit the file to add more options to it!

```javascript
{
    "credentials": [
        {
            "name": "Sessions",
            "public_token": "XXXXXXXXXXXXXXXXXXX",
            "token": "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX",
            "type": "amazon"
        },
        {
            "name": "Main",
            "public_token": "XXXXXXXXXXXXXXXXXXX",
            "token": "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX",
            "regions": [
                "us-east-1"
            ],
            "type": "amazon"
        }
    ],
    "ssh_config": [
        {
          "priority": 0,
          "user": "ubuntu",
          "port": 22,
          "options": ["-o", "StrictHostKeyChecking no"]
        }
    ]
}
```

So in the above config we have defined 2 accounts namely `Sessions` and `Main` and both of them are connected to `amazon` cloud provider. `Sessions` account does not have any `regions` so `sift` will consider all available regions from the cloud provider which is `amazon` in this example.

**Note**: Names you give to different accounts have nothing to do with the real account name in the cloud providers.

### Sample usages

If you run `sift` without any argument then it will show a list of all instances in the configured accounts. If you run it with `-l` it will list the current available accounts:

```bash
$ sift -l 
Sessions amazon (us-east-1,us-west-2,eu-west-1,ap-northeast-1,ap-southeast-2) 
Main amazon (us-east-1) 
```

But `sift` comes with querying capability. For simple queries we have provided you with some arguments as following:

- `-n`: filtering based on the name of the instance
- `--id`: filtering based on instance id
- `-image`: filtering based on the image id
- `-hostname`: filtering based on the hostname
- `-ip`: filtering based on the Ip address

If you need to express a complete query then see next section.

### Query language

Use `-k` option together with a plugin name to get the list of supported keys that can be used in your queries.

```
$ sift -k amazon
Keys for amazon: id, name, region, hostname, account, image, ip
```

In order to use `sift` query feature you need to use `-q`:

```bash
$ sift -q 'name contains session'
```

You can also have more than one statement, and combine them with `and` / `or`:

```bash
$ sift -q '(name contains session) or (id = i-ae7fcafc)'
```

As you see the statement consists of key-value pairs. Retrieving keys is mentioned in the previous section. 
You can combine logical statement as much as you need. If you need to have whitespaces or use `sift` keywords in your values you can escape them by using double qoutes like this:

```bash
$ sift -q "(name contains 'auth session') or (id = i-ae7fcafc)"
```

#### Supported logical operators

Supported logical operators for the query system:
- `and`, `And`, `AND`, `&`
- `or`, `Or`, `OR`, `|`
- `not`, `Not`, `NOT`, `~`, `!`

#### Supported verbs

Supported verbs you can use in your key-value statements:
- `contains`, `CONTAINS`, `Contains`
- `<>`, `!=`
- `=`, `==`


### Running commands

You can run shell commands on one or many instances that matches your query:

```bash
$ sift -q 'name contains session' -c 'uptime'
```

This commands will only be executed on the instance you select but if you want to execute the command on all matching instances you can use `-A` like this:

```bash
$ sift -q 'name contains session' -c 'uptime' -A
```

### Defining Aliases

If you're running some queries every day you can define alias for them so you don't type them out every time you need them. You can use the key `alias` in your config file to define a list of aliases:

```javascript
{
   "alias":{
      "session":{
         "accounts":[
            "Sessions"
         ],
         "query":"(tag.type = Auth) AND (tag.environment = PRODUCTION)"
      },
      "session log":{
         "accounts":[
            "Sessions"
         ],
         "query":"(tag.type = Auth) AND (tag.environment = PRODUCTION)",
         "command":"tail -1000f /opt/session-logs/session.log"
      }
   }
}
```

And you can use them like this:

```bash
$ sift session
```

```bash
$ sift session log
```

#### Including alias from file

You can define your aliases in a separate file and include them in your config file as the following:

```javascript
{
    "credentials": ...
    "ssh_config": ...
    "alias_includes": ["/Users/user/Documents/my\_aliases.json"]
}
```

And `my_aliases.json` looks like this:

```javascript
{
    "session":{
     "accounts":[
        "Sessions"
     ],
     "query":"(tag.type = Auth) AND (tag.environment = PRODUCTION)"
    },
    "session log":{
     "accounts":[
        "Sessions"
     ],
     "query":"(tag.type = Auth) AND (tag.environment = PRODUCTION)",
     "command":"tail -1000f /opt/session-logs/session.log"
    }
}
```


### Reference config file

```javascript
{
   "credentials":[
      {
         "name":"Realms",
         "public_token":"XXXXXXXXXXXXXXXXXXXXXXXX",
         "token":"XXXXXXXXXXXXXXXXXXXXXXXXXX",
         "regions":[
            "us-east-1",
            "us-west-2",
            "eu-west-1",
            "ap-northeast-1",
            "ap-southeast-2"
         ],
         "type":"amazon"
      },
   ],
   "ssh_config":[
      {
         "priority":0,
         "user":"ubuntu",
         "port":22,
         "options":[
            "-o",
            "StrictHostKeyChecking no"
         ]
      }
   ],
   "alias_includes":[
      "/Users/user/Documents/my_aliases.json"
   ],
   "plugins":[
      "amazon",
      "digitalocean"
   ],
   "allowed_filters":[

   ],
   "enabled_filters":[

   ],
   "force_regions":false,
   "alias":{
        "session":{
         "accounts":[
            "Sessions"
         ],
         "query":"(tag.type = Auth) AND (tag.environment = PRODUCTION)"
        }
   },
   "auto_connect_on_one_result":true
}
```

### Contact
Got any questions? You can email us at [sift@mojang.com](mailto:sift@mojang.com)

### Authors
* [Amir Moulavi](https://twitter.com/mamirm)
* [David Marby](http://dmarby.se)

### License
Distributed under the [MIT License](https://github.com/Mojang/Sift/blob/master/LICENSE.md)


### Thanks to
* [Per Lööv](http://perloov.com) for the awesome logo