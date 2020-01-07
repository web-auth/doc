# Javascript

You will interact with the authenticators through an HTML page and Javascript using the Webauthn API.

No script is provided with the library because it could become hard to manage all types of scripts and application specificity. However, you will find on this page two JS scripts: the first one for the registration of an authenticator \(Attestation Ceremony\). The other one for the user authentication \(Assertion Ceremony\).

Feel free to adapt these script for your application \(React, Vue…\).

{% code title="attestaion.js" %}
```javascript
const publicKey = "{PLACE YOUR CREDENTIAL OPTIONS HERE}";

function arrayToBase64String(a) {
    return btoa(String.fromCharCode(...a));
}

function base64url2base64(input) {
    input = input
        .replace(/=/g, "")
        .replace(/-/g, '+')
        .replace(/_/g, '/');

    const pad = input.length % 4;
    if(pad) {
        if(pad === 1) {
            throw new Error('InvalidLengthError: Input base64url string is the wrong length to determine padding');
        }
        input += new Array(5-pad).join('=');
    }

    return input;
}

publicKey.challenge = Uint8Array.from(window.atob(base64url2base64(publicKey.challenge)), function(c){return c.charCodeAt(0);});
publicKey.user.id = Uint8Array.from(window.atob(publicKey.user.id), function(c){return c.charCodeAt(0);});
if (publicKey.excludeCredentials) {
    publicKey.excludeCredentials = publicKey.excludeCredentials.map(function(data) {
        data.id = Uint8Array.from(window.atob(base64url2base64(data.id)), function(c){return c.charCodeAt(0);});
        return data;
    });
}

navigator.credentials.create({ 'publicKey': publicKey })
    .then(function(data){
        const publicKeyCredential = {
            id: data.id,
            type: data.type,
            rawId: arrayToBase64String(new Uint8Array(data.rawId)),
            response: {
                clientDataJSON: arrayToBase64String(new Uint8Array(data.response.clientDataJSON)),
                attestationObject: arrayToBase64String(new Uint8Array(data.response.attestationObject))
            }
        };
        
        //Send the response to your server
        // You can use JSON.stringify(publicKeyCredential); to have the JSON object as a string
    })
    .catch(function(error){
        alert('Open your browser console!');
        console.log('FAIL', error);
    });
```
{% endcode %}

{% code title="assertion.js" %}
```javascript

const publicKey = "{PLACE YOUR CREDENTIAL OPTIONS HERE}";
function arrayToBase64String(a) {
    return btoa(String.fromCharCode(...a));
}
function base64url2base64(input) {
    input = input
        .replace(/-/g, '+')
        .replace(/_/g, '/');
    const pad = input.length % 4;
    if(pad) {
        if(pad === 1) {
            throw new Error('InvalidLengthError: Input base64url string is the wrong length to determine padding');
        }
        input += new Array(5-pad).join('=');
    }
    return input;
}
publicKey.challenge = Uint8Array.from(window.atob(base64url2base64(publicKey.challenge)), function(c){return c.charCodeAt(0);});
if (publicKey.allowCredentials) {
    publicKey.allowCredentials = publicKey.allowCredentials.map(function(data) {
        data.id = Uint8Array.from(window.atob(base64url2base64(data.id)), function(c){return c.charCodeAt(0);});
        return data;
    });
}
navigator.credentials.get({ 'publicKey': publicKey })
    .then(function(data){
        const publicKeyCredential = {
            id: data.id,
            type: data.type,
            rawId: arrayToBase64String(new Uint8Array(data.rawId)),
            response: {
                authenticatorData: arrayToBase64String(new Uint8Array(data.response.authenticatorData)),
                clientDataJSON: arrayToBase64String(new Uint8Array(data.response.clientDataJSON)),
                signature: arrayToBase64String(new Uint8Array(data.response.signature)),
                userHandle: data.response.userHandle ? arrayToBase64String(new Uint8Array(data.response.userHandle)) : null
            }
        };

        //Send the response to your server
        // You can use JSON.stringify(publicKeyCredential); to have the JSON object as a string
    })
    .catch(function(error){
        alert('Open your browser console!');
        console.log('FAIL', error);
    });
```
{% endcode %}

