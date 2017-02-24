---
title: "NodeJS中直接使用POST上传文件到阿里云"
date: 2016-04-04 10:16:00 +0800
categories: [program]
tags: [nodejs]
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

function uploadToAliyun(config, filename, filepath, callback) {
    if (!filename)
        filename = path.basename(filepath);

    var policy = {
        "expiration": (new Date(Date.now() + 360000)).toISOString(),
        "conditions": [{
            key: filename
        }]
    };
    var encodedPolicy = xbase64.encode(JSON.stringify(policy));
    var encodedSign = xbase64.encode(hmac_sha1(encodedPolicy, config.accessKeySecret));
    var formData = {
        key: filename,
        OSSAccessKeyId: config.accessKeyId,
        policy: encodedPolicy,
        Signature: encodedSign,
        success_action_status: 200,
        file: fs.createReadStream(filepath)
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
            callback(new Error('status: ' + resp.statusCode + ', error: ' + body));
    });
}

uploadToAliyun({
    url: 'bucket URL',
    accessKeyId: 'Access Key ID',
    accessKeySecret: 'Access Key Secret'
}, '', '/Users/vietor/Pictures/logo.png', function(err) {
    console.log(err);
});
```
