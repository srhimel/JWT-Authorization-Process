# JWT Authentication steps

JWT or JSON WEB Token is a way to secure your data, so 
that no one get it without a proper authorization. 

### step 1: ClientSide(create and store the idToken)
get the id token on AuthStateChange and store it in localStorage  
``` js
getIdToken(user)
    .then(idToken => localStorage.setItem('idToken', idToken));
```

### step 2: ClientSide(pass the idToken to database)
send the id Token from localStorage, on fetch header so that you can 
make the data accessible only when it comes with it's unique idToken.
- on headers, give it a name authorization and on property add Bearer before the idToken
``` js
{
    headers: {
        'authorization': `Bearer ${localStorage.getItem('idToken')}`
}
```

### step 3: ServerSide(Add the Firebase Admin SDK to your server AKA firebase server environment)

install firebase admin
``` cmd
npm install firebase-admin --save
```
### step 4: ServerSide(Initialize the SDK )

To generate a private key file for your service account:

1. In the Firebase console, open Settings > Service Accounts.
2. Click Generate New Private Key, then confirm by clicking Generate Key.
3. Securely store the JSON file containing the key.

Now store the user-login----.json file in your backend root folder and 
initialize it 
```js
const admin = require("firebase-admin");
const serviceAccount = require("./user-login----.json.json");
// --== or you can put your credential on .env.local like Jhankar vai did ==-- \\
const serviceAccount = JSON.parse(process.env.FIREBASE_SERVICE_ACCOUNT);

admin.initializeApp({
  credential: admin.credential.cert(serviceAccount)
});
```


### step 5: ServerSide(get the idToken and decode the user information)
on server side catch the id token from req.headers, 

decode it and pass it using next() method
```js
async function verifyToken(req, res, next) {
    if (req.headers?.authorization?.startsWith('Bearer ')) {
        const idToken = req.headers.authorization.split('Bearer ')[1];
        try {
            const decodedUser = await admin.auth().verifyIdToken(idToken);
            req.decodedUserEmail = decodedUser.email;
        }
        catch {

        }
    }
    next();
}
```

### Finally: (match the idToken info with the params/query info)
here is an example

```js
 app.get('/orders', verifyToken, async (req, res) => {
    const email = req.query.email;
    if (req.decodedUserEmail === email) {
        const query = { email: email };
        const cursor = orderCollection.find(query);
        const orders = await cursor.toArray();
        res.json(orders);
    }
    else {
        res.status(401).json({ message: 'User not authorized' })
    }

});

// another

app.put('/users/admin', verifyToken, async (req, res) => {
    const user = req.body;
    const requester = req.decodeEmail;
    if (requester) {
        const requesterAccount = await userCollection.findOne({ email: requester });
        if (requesterAccount.role === 'admin') {
            const query = { email: user.email }
            const update = { $set: { role: 'admin' } }
            const result = await userCollection.updateOne(query, update)
            res.json(result);
        }
    }
});
```

#### I tried my best to explain it in a minimal way, may be it's not a perfect example but i tried anyway.