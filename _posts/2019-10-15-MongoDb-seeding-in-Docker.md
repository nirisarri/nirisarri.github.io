---
layout: post
title: MongoDb seeding in Docker
tags: 'MongoDb,Docker'
published: true
date: '2019-10-15'
---

For my last project I have been tinkering with Docker a lot, and one of the tasks I wanted to accomplish was to have a MongoDb instance in docker get seeded with data and users, as soon as the instance is created.

I have been trying several strategies. My first approach was the most naive one: after constructing the container, run manually a shell script that populated the database and created the users. It works fine, I guess. but is a manual process and is a step you must remember to run... also when moving to prod environments will not work: 

>## *You should never have Read/Write access into prod!*

The second approach I took was to create a separate container that would execute the seeding script. This worked better, as the deployment would create the container; and within this container I can execute shell scripts that do `mongoImport` or mongo queries against the database. Seeding container lingers in the application and eventually needs to be manually removed, and this was the only  drawback I found... or so I thought.

That last approach works very well for seeding the database, but when dealing with security lock-down, you must do more things in order for that to happen.

MongoDb requires you to create an admin user, then reboot the server and log in again using the newly created admin user to create more users and assign roles more limiting.

Looking for a way to circumvent this I found another strategy for seeding and user creation. It came after reading the [mongoDb documentation](https://hub.docker.com/_/mongo) for its container..
In this documentation I found that you can pass the environment variables `$MONGO_INITDB_ROOT_USERNAME` and `$MONGO_INITDB_ROOT_PASSWORD` that will be used for locking down the database (Yay! :) I have an admin user in my mongo! ... now what??). Then I also found that if I map a volume to the  `/docker-entrypoint-initdb.d` directory I can add to the mapped location any `.sh` or `.js` file and the deployment will run them at container creation time!

This is great because I don't need more containers, and is guaranteed to run once when creating the container.
So I created a `01.Seed.sh` and a `02.Users.sh` files that will in turn seed the database and then create the locked down users.  The container deployment will execute the files in alphabetical order.

My Seed file uses `mongoImport` to seed the database:
```sh
mongoimport --authenticationDatabase=admin \
   --username=$MONGO_INITDB_ROOT_USERNAME \
   --password=$MONGO_INITDB_ROOT_PASSWORD \
   --mode upsert \
   --host 127.0.0.1 \
   --db MyDatabase \
   --collection MyCollection \
   /docker-entrypoint-initdb.d/myDb_MyCollection_Data.json
```

In this script I am passing the env vars I mentioned before to actually gain admin access and do the database seeding.

For user creation I was trying first to create them via a `.js` script as it seemed more natural to me, but passing environment variables to that file is not supported. So I decided to use the `.sh` approach.

My Users file looks like this:

```javascript
#!/bin/bash
set -e

mongo <<EOF
use admin
db.createUser({
  user: '$LOCKED_USERNAME',
  pwd:  '$LOCKED_PASSWORD',
  roles: [
     { role: 'readWrite', db: 'myDb' },
     { role: 'readWrite', db: 'OtherDb' }]
})
EOF
```

This file will take a couple env. vars that I can pass along to the docker-compose file when I need to create the database; in fact I can eventually run this script only if the env. vars are set, that way I can limit the times the script gets run.
