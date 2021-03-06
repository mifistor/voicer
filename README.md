voicer
======

AGI voice recognizer for Asterisk [use Yandex or Google ASR]

Call to special extension, say "Vasya" and Asterisk connect you with Vasya! Excellent!

[![Build Status](https://travis-ci.org/antirek/voicer.svg?branch=master)](https://travis-ci.org/antirek/voicer)



Workflow
========

Voicer work as AGI-server. Voicer accept request from asterisk via AGI app.
It run handler for each request. Handler command asterisk record file.
After this send file to recognition service, receive text, search by text in 
source of data for finding concordance, if source have this text it return 
channel for call, voicer set dialplan vars RECOGNITION_STATUS as SUCCESS и 
RECOGNITION_CHANNEL for finded channel.



Fast start
==========

Use voicer-app http://github.com/antirek/voicer-app



Use 
===

in your work dir


## Install ##

> $ npm install voicer [--save]


## Write app.js ##

Add to your **app.js** code

`````
var config = require('./config');
var Voicer = require('voicer');

var voicer = new Voicer(config);
voicer.start();

`````

## Start *voicer* server ##

> $ node app.js




Configuration
=============

## Config.js ##


``````
{
    server: {
        port: 3000
    },
    processing: {
        totalAttempts: 2,
        playGreeting: true,
        playBeepBeforeRecording: false
    },
    asterisk: {
        sounds: {
            onErrorBeforeFinish: 'invalid',
            onErrorBeforeRepeat: 'invalid',
            greeting: 'tt-monkeysintro'
        },
        recognitionDialplanVars: {
            result: 'RECOGNITION_RESULT',
            channel: 'RECOGNITION_CHANNEL'
        }
    },
    record: {
        directory: '/tmp',
        type: 'wav',
        duration: 2,
    },
    recognize: {
        directory: '/tmp',
        type: 'google',  // ['yandex', 'google']
        options: {
            developer_key: 'dev_key'
        }
    },
    lookup: {
        type: 'file',  // ['file', 'mongodb', 'mysql']
        options: {
            dataFile: 'data/peernames.json'
        }
    },
    logger: {
        console: {
            colorize: true
        },
        syslog: {
            host: 'localhost'
        },
        file: {
            filename: '/var/log/voicer.log',
            json: false
        }
    }
}

``````

## Asterisk ##

Write dialplan for call to AGI-server voicer like

`````
[default]
exten=1000,1,AGI(agi://localhost:3000)
exten=1000,n,GotoIf($[${RECOGNITION_RESULT}=SUCCESS]?:default,1000,4)
exten=1000,n,Dial(${RECOGNITION_CHANNEL})

`````



## Some more ##

Also you can tune additional options in *config.js*. 

Try find optimal value for duration of record.


## Errors?! ##

Bugs?! Oh, contact with me. I want to eat them.


## Links ##

Yandex API key: https://developer.tech.yandex.ru/

Google API key: https://console.developers.google.com/

Voice speed dial on Asterisk http://habrahabr.ru/post/248263/  (russian)


## Mongo ##

1. create collection in mongodb

For example:

> mongoimport --db __yourdb__ --collection __yourcollection__ --type json --file peernames.json --jsonArray

> // peernames.json sample in ./example/data 


2. set *config.js* work with mongodb

`````
lookup: {
    type: 'mongodb',
    options: {
        url: 'mongodb://localhost/yourdb',
        collection: 'yourcollection'
    }
}
`````



## MySQL ##

1. create db and table

````
CREATE DATABASE `voicer` CHARACTER SET utf8 COLLATE utf8_general_ci;
CREATE TABLE `peernames` (
	  `id` int(11) NOT NULL AUTO_INCREMENT,
	  `name` varchar(255) NOT NULL,
	  `channel` varchar(255) NOT NULL,
	  PRIMARY KEY (`id`)
	) ENGINE=InnoDB DEFAULT CHARSET=utf8;
````
and fill table data


2. set *config.js* work with mysql

````
lookup: {
    type: 'mysql',
    options: {
        host: 'locahost',
        port: 3306,
        username: 'root',
        password: '1234',
        database: 'voicer',
        table: 'peernames'
    }
}
````




Development
===========


## Test ##

> npm test


## Code coverage ##

> grunt