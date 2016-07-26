---
title: "NodeJS中直接使用POST上传文件到七牛"
date: 2016-04-04 10:10:00 +0800
categories: [program]
tags: [nodejs, 七牛]
---

``` javascript
var fs = require('fs'),
    path = require('path'),
    crypto = require('crypto');
var xbase64 = require('xbase64');
var request = require('request');

function hmac_sha1(content, secret) {
    return crypto.createHmac('sha1', secret).update(content).digest();
}

function uploadToQiniu(config, filename, filepath, callback) {
    if (!filename)
        filename = path.basename(filepath);
    var putPolicy = {
        scope: config.bucket + ':' + filename,
        deadline: Math.floor(Date.now() / 1000) + 3600,
        returnBody: "{}"
    };
    var encodedPutPolicy = xbase64.urlencode(JSON.stringify(putPolicy));
    var encodedSign = xbase64.urlencode(hmac_sha1(encodedPutPolicy, config.secretKey));
    var formData = {
        key: filename,
        file: fs.createReadStream(filepath),
        token: config.accessKey + ':' + encodedSign + ':' + encodedPutPolicy
    };
    request.post({
        url: config.url,
        formData: formData
    }, function(err, resp, body) {
        if (err)
            callback(err);
        else if (resp.statusCode == 200)
            callback(null);
        else
            callback(new Error('status: ' + resp.statusCode) + ', message: ' + JSON.parse(body).error);
    });
}

uploadToQiniu({
    url: 'http://upload.qiniu.com/',
    bucket: '空间名',
    accessKey: '访问KEY (AK)',
    secretKey: '安全KEY (SK)'
}, '', 'd:\\dollar.jpg', function(err) {
    console.log(err);
});
```
