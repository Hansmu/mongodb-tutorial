# Security

There's a checklist provided by MongoDB that you'll need to apply.

https://www.mongodb.com/docs/manual/administration/security-checklist/

## Authentication & authorization

Authentication - identify a valid user.

Authorization - identify what a user can do.

An entity in MongoDB is made up of a username, password, and roles.

Roles consist of privileges.

Privileges consist of resources (collections) and actions (insert, delete etc).

### Roles

You'd want to have a set number of roles that are used.

For example, there's usually an administrator role that should not be able to insert or fetch into collections.
Should just have access to configs, could create users etc.

A developer role should only be able to insert, update, delete, or fetch data, but not create users or change the DB config.

### Adding authentication

If you start your DB from the commandline, you can add `--auth` to it to start it with authentication.

In the shell, you can authenticate after you've connected by using `db.auth('username', 'password')` or with `mongo -u username -p password`.
However, note that you will have added access to a specific database.
So authenticating from the shell will also require you to add the DB you're authenticating against.
For that you'd add `authenticationDatabase` - `mongo -u username -p password --authenticationDatabase admin`.

When you first enable auth, you'll have no users. MongoDB has a special exception for this called the "localhost exception".

The localhost exception allows you to enable access control and then create the first user or role in the system.

To create the first user, you can do:

```bash
use admin;
```

```js
db.createUser({
    user: "username",
    pwd: "password",
    roles: [
        "userAdminAnyDatabase" // Built-in role to allow adminstrating any database in this MongoDB env
    ]
});
```

### Built-in roles and assigning

MongoDB ships with some built-in roles that'll cover most of the usual use cases that you'll have.

You could use a specific DB, and then in there add a user.

```bash
use shop;
```

```js
db.createUser({
    user: "appdev",
    pwd: "dev",
    roles: [
        "readWrite" // Built-in role to allow reads and writes
    ]
});
```

When you log in with the user, remember to use the correct DB.

When you want to modify the rights for a user, you can use updateUser.

```js
db.updateUser(
    "someUserName",
    {
        roles: [
            "readWrite",
            { role: "readWrite", db: "blog" }
        ]
    });
```

The user will exist on a specific DB where it was defined, so before updating, make sure that you're logged in with admin and on the DB that the user was created on.

Use `getUser` to see information about the user.

```js
db.getUser("someUserName");
```

## Transport encryption

https://www.mongodb.com/docs/manual/tutorial/configure-ssl/

Transportation between MongoDB and the client is happening over the wire.

If there's any sort of listening going on over the wire and no encryption is present, that means everything is readable.

So first step to enabling this is to create a private/public key value pair.

The important part when creating the pair is the common name.
When you're developing locally, it should be "localhost".
When you're deploying the app, it should be the actual server address of the MongoDB server.

You need to concatenate the certificate and private keys to a .pem file for MongoDB.

When starting up the MongoDB server, you need a couple of arguments to run it with SSL.
* `--sslMode` how to enable or disable SSL.
* `--sslPEMKeyFile` to give the route to the .pem file generated previously.
* If a certificate authority vouches for the SSL file, then `--sslCAFile` should be added as well.

## Encryption at rest

This is to enable encryption for data when it's just sitting on disk.
Enterprise allows for encryption at rest.