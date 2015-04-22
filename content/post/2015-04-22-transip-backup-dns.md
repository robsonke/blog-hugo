---
author: Rob
date: 2015-04-22 21:55:23+00:00
layout: post
slug: transip-backup-dns
title: Backup dns entries of Transip.nl with Nodejs
categories:
- NodeJS
---

One of the worst things that can happen is that you loose sensitive data. I recently had the favor to experience that. Luckily not with something really important and I was able to restore the situation but it made me thinking: this never again. 

The issue was that due to some bug in the admin environment of Transip.nl (A Dutch dns provider), I lost the dns entries of a domain.... Once I discovered what happened, I asked Transip for a backup of older data, unfortunately there was no backup and the only way to create that, was through their SOAP api. I'm not going to write a lot of information about this, I'd just like to share the code I used.

Thanks to some work done by other guys (https://github.com/DualDev/node-transip), my job became pretty simple and with just a few lines of Javascript, I had my backup script.

In general the script just connects with the api, loops through domains and stores the dns entries as a json tree in a file. I'm running this daily so I gave all filenames a timestamp.

``` javascript
  'use strict';
  
  // based on https://github.com/DualDev/node-transip
  
  var TransIP = require('transip');
  var fs = require('fs');
  
  // this requires a config.js in the same folder as this index.js
  var config = require('./config.js');
  
  var login = config.transip.login;
  var privateKey = config.transip.privateKey;
  
  
  
  
  (function(){
    // remove possible exisitng file
    clearExistingFile(getFilename());
  
    var transipInstance = new TransIP(login, privateKey);
  
    transipInstance.domainService.getDomainNames().then(function(domains) {
      
      for(var domain in domains) {
        var domainUrl = domains[domain];
        transipInstance.domainService.getInfo(domainUrl).then(function(info) {
          // store a line with the domain name 
          var domainMessage = "DOMAIN: " + info['name'] + " ==============================";
          console.log(domainMessage);
          saveDnsEntry(domainMessage);
          console.log('Saving entries for domain: ' + info['name']);
          saveDnsEntry(info['dnsEntries']);
        });
      }
    });
  })();
  
  
  
  function saveDnsEntry(data) {
    var fs = require('fs');
    
    fs.appendFile(getFilename(), JSON.stringify(data, null, 2) + "\n", function(err) {
      if(err) { return console.log(err); }
    });
  }
  
  function clearExistingFile(filePath){
    if(fs.existsSync(filePath))
      fs.unlinkSync(filePath);
  }
  
  function getFilename(){
    var date = new Date();
    var dateStr = date.getDate() + '-' + (date.getMonth() + 1) + '-' + date.getFullYear();
    return "dns-backup-"+dateStr+".bak";
  }
```

[See the Github repo here](https://github.com/MaxxtonGroup/transip-backup) (there's some more explanation there)