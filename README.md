### hapi-auth-signature

Signature authentication scheme that wraps the [Joyent Signature Authentication Scheme](https://github.com/joyent/node-http-signature) and requires validating signed authorization header.

- `validateFunc` - (required) an api key id and secret key lookup validation function with the signature `function(keyId, callback)` where:
    - `parsedHeader` - [http-signature](https://github.com/joyent/node-http-signature) parsed header including the key id and signature.
    - `callback` - a callback function with the signature `function(err, isValid, credentials)` where:
        - `err` - an internal error.
        - `isValid` - `true` if the signature is verified, otherwise `false`.
        - `credentials` - a credentials object passed back to the application in `request.auth.credentials`. Typically, `credentials` are only
          included when `isValid` is `true`, but there are cases when the application needs to know who tried to authenticate even when it fails
          (e.g. with authentication mode `'try'`).

The validation function shown below is based on an hmac scheme with a key identifier and secret key stored in a user record. [http-signature](https://github.com/joyent/node-http-signature) supports the following algorithms:

* rsa-sha1
* rsa-sha256
* rsa-sha512
* dsa-sha1
* hmac-sha1
* hmac-sha256
* hmac-sha512


```javascript

var HttpSignature = require('http-signature');

var users = [
    {
        id: '1,
        username: 'john',
        apikeyid: '18KF2FGK6807ZQA945R2',
        secretkey: '46573e78ce9df4f2d9ae93afd5f5c281', // 'secret'
    }
];

var validate = function (parsedHeader, callback) {

    var keyId = parsedHeader.keyId;
    var credentials {};
    var secretKey;
    users.forEach(function(user, index) {
        if (user.apikeyid === keyId) {
            secretKey = user.secretkey;
            credentials = {id: user.id, username: user.username};
        }
    }

    if (!secretKey) {
        return callback(null, false);
    }

    if(HttpSignature.verifySignature(parsedHeader, user.secretkey)) {
        callback(null, true, credentials);
    } else {
        callback(null, false);
    };
};

server.pack.register(require('hapi-auth-signature'), function (err) {

    server.auth.strategy('hmac', 'signature', { validateFunc: validate });
    server.route({ method: 'GET', path: '/', config: { auth: 'hmac' } });
});

```