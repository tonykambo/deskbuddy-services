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

Create the DashDB service:

```
cf create-service dashDB Entry dashDB-deskbuddy
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
        "node-red-contrib-ibm-watson-iot":"0.x",
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
  - dashDB-deskbuddy
  env:
    NODE_RED_STORAGE_NAME: DeskBuddy.cloudantNoSQLDB
declared-services:
  DeskBuddy.cloudantNoSQLDB:
    label: cloudantNoSQLDB
    plan:  Lite 
  dashDB-deskbuddy:
  	 label: dashdb
  	 plan: Entry 
```   

Add dashDB to local install:

```
npm install node-red-nodes-cf-sqldb-dashdb
```

Add Watson IoT to local install:

```
npm install node-red-contrib-ibm-watson-iot
```

Add IBM IoT to local install:

```
npm install node-red-contrib-scx-ibmiotapp
```

Create .cfignore with the following content:

```
node_modules
```

Create .gitignore with the following content:

```
node_modules
.config.json
```

Create the IoT service

```
cf cs iotf-service iotf-service-free deskbuddy-iot
```

Launch Node-RED from local workstation

```
node-red ./deskbuddy.json -s ./settings.js -u ./
```

Create dashDB Tables

Climate Table

```
create table climate
(
    deviceId VARCHAR (36) NOT NULL,
    dateMeasured TIMESTAMP (0) NOT NULL,
    temperature DECIMAL (19,2),
    humidity DECIMAL (19,2),
    heatIndex DECIMAL (19,2),
    PRIMARY KEY (dateMeasured)
) ORGANIZE BY ROW;
```

### Upgrading Node-RED and Node.js

```
sudo npm cache clean
sudo npm install -g --unsafe-perm node-red
sudo npm install -g n
sudo n stable
```

Change to the deskbuddy service directory to upgrade the dashdb driver.

```
npm install node-red-nodes-cf-sqldb-dashdb
``` 

## Node-RED Development

Use the IBM IoT App Node for Node-RED, not the Watson IoT node. The Watson IoT node is for devices and gateways to allow them to send events and receive commands. The IBM IoT App Node is for applications that is used to receive device events and send commands back to the device. 

## Useful SQL statements

```
drop table climate;

select * from climate;

DELETE FROM CLIMATE;

select * from climate where deviceid='nodemcu1001' order by datemeasured ASC;

insert into climate (deviceId,dateMeasured,temperature,humidity,heatIndex)
VALUES ('nodemcu1001',TIMESTAMP_FORMAT('2017-4-15 9:00:00','YYYY-MM-DD HH24:MI:SS'),19.5,45.4,19.2),
('nodemcu1001',TIMESTAMP_FORMAT('2017-4-15 10:00:00','YYYY-MM-DD HH24:MI:SS'),20.3,46.2,20.1),
('nodemcu1001',TIMESTAMP_FORMAT('2017-4-15 11:00:00','YYYY-MM-DD HH24:MI:SS'),21.9,45.2,21.3),
('nodemcu1001',TIMESTAMP_FORMAT('2017-4-15 12:00:00','YYYY-MM-DD HH24:MI:SS'),23.2,46.3,23.2),
('nodemcu1001',TIMESTAMP_FORMAT('2017-4-15 13:00:00','YYYY-MM-DD HH24:MI:SS'),28.7,42.2,26.1),
('nodemcu1001',TIMESTAMP_FORMAT('2017-4-15 14:00:00','YYYY-MM-DD HH24:MI:SS'),26.3,49.2,27.2),
('nodemcu1001',TIMESTAMP_FORMAT('2017-4-15 15:00:00','YYYY-MM-DD HH24:MI:SS'),24.5,43.2,24.5),
('nodemcu1001',TIMESTAMP_FORMAT('2017-4-15 16:00:00','YYYY-MM-DD HH24:MI:SS'),22.1,42.2,22.4);

DELETE FROM climate
WHERE temperature=22.1 OR temperature=24.5 OR temperature=26.3 OR temperature=28.7;
```