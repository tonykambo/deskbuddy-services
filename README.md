#DeskBuddy Services


## Install and Configure Node-RED

Clone the Node-RED Bluemix starter

https://github.com/ibmets/node-red-bluemix-starter.git

Copy the files to the new repository making sure you don't copy the .git folder

```
cf login -a api.ng.bluemix.net
```

Create a space

```
cf create-space DeskBuddy-dev
```

Login to space

```
cf login
```

Create the Cloudant service:

```
cf create-service cloudantNoSQLDB Lite DeskBuddy.cloudantNoSQLDB
```

Modify package.json:

```
{
    "name"         : "DeskBuddy",
    "version"      : "0.6.0",
    "dependencies": {
        "when": "~3.x",
        "nano": "6.2.x",
        "bcrypt": "1.0.2",
        "cfenv":"~1.0.0",
        "express":"4.x",
        "body-parser":"1.x",
        "http-shutdown":"1.2.0",
        "node-red": "0.x",
        "node-red-bluemix-nodes":"1.x",
        "node-red-node-watson":"0.x",
        "node-red-node-openwhisk":"0.x",
        "node-red-node-cf-cloudant":"0.x",
        "node-red-contrib-scx-ibmiotapp":"0.x",
        "node-red-nodes-cf-sqldb-dashdb":"0.x"
    },
    "scripts": {
        "start": "node --max-old-space-size=384 index.js --settings ./bluemix-settings.js -v"
    },
    "engines": {
        "node": "6.x"
    }
}

```

Modify manifest.yml:

```
applications:
- memory: 512M
  services:
  - DeskBuddy.cloudantNoSQLDB
  env:
    NODE_RED_STORAGE_NAME: DeskBuddy.cloudantNoSQLDB
declared-services:
  DeskBuddy.cloudantNoSQLDB:
    label: cloudantNoSQLDB
    plan:  Lite  
```   

Create the IoT service

```
cf cs iotf-service iotf-service-free deskbuddy-iot
```

Launch Node-RED from local workstation

```
node-red ./deskbuddy.json -s ./settings.js -u ./
```
