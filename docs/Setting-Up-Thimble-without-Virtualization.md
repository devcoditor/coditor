Uses
-----

You can also setup Thimble and its needed components outside Vagrant and Virtualbox. This might be needed if you want to:
- Host your own instance of Thimble
- Cannot run virtualization software on your computer

Prerequisites
-----

You will need:
- Node.js 4.x or above (see note below)
  - * **Note:** The login.webmaker.org dependency needs a node version of 4.x only while all the other dependencies work with a node version of 4.x and above. We suggest installing [NVM](https://github.com/creationix/nvm) to allow the use of multiple versions of node.
- Postgresql 9.4 or above (for the publish.webmaker.org dependency)
- g++ 4.8 or above (for the login.webmaker.org dependency)

Setup
-----

In order to run Thimble, the following components are required. The following
is an abbreviated guide to getting it all set up.  Please see each server's README for more details.

**You'll need**
* Bramble
* Thimble
* Webmaker ID server
* Webmaker Login Server
* PostgreSQL Database
* Webmaker Publishing Server

## Installing the Parts

Please note: On Windows, use ``copy`` instead of ``cp``

**Bramble**
* Clone https://github.com/mozilla/brackets using ``git clone --recursive https://github.com/mozilla/brackets.git``
* Run ``npm install`` to install dependencies
* Run `npm run build` to create `/dist` extensions and third-party libs
* Run ``npm start`` to get a static server running on [http://localhost:8000/src](http://localhost:8000/src). You can try the demo version at [http://localhost:8000/src/hosted.html](http://localhost:8000/src/hosted.html)
* For more information on setting up Bramble, refer to [Bramble Setup](https://github.com/mozilla/brackets#how-to-setup-bramble-brackets-in-your-local-machine)

**Thimble**
* Fork and clone https://github.com/mozilla/thimble.mozilla.org
* Run ``cp env.dist .env`` to create an environment file
* Run ``npm install`` to install dependencies
* Run ``npm run localize`` to generate the locale files
* Run ``npm start`` to start the server
* Once everything is ready and running, Thimble will be available at [http://localhost:3500/](http://localhost:3500/)

**id.webmaker.org**
* Clone https://github.com/mozilla/id.webmaker.org
* Run ``cp sample.env .env`` to create an environment file
* Run ``npm install`` to install dependencies
* Run ``npm start`` to start the server

**login.webmaker.org**
* Clone https://github.com/mozilla/login.webmaker.org
* Run ``npm install`` to install dependencies
* Run ``cp env.sample .env`` to create an environment file
* Run ``npm start`` the server

**PostgreSQL**
* Run ``initdb -D /usr/local/var/postgres`` to initialize PostreSQL
  * If this already exists, run ``rm -rf /usr/local/var/postgres`` to remove it
* Run ``postgres -D /usr/local/var/postgres`` to start the PostgreSQL server
* Run ``createdb publish`` to create the Publish database

**publish.webmaker.org**
* These steps assume you've followed the PostgreSQL steps above, including creating the publish database.
* Clone https://github.com/mozilla/publish.webmaker.org
* Run ``npm install`` to install dependencies
* Run ``npm run env``
* Run ``npm install knex -g`` to install knex
* Run ``npm run knex`` to seed the publish database created earlier
* Run ``npm start`` to run the server

## Getting Ready to Publish
To publish locally, you'll need to do the following...

**1. Teach the ID server about the Publish server**

* Run ``createdb webmaker_oauth_test`` to create a test database
* In your id.webmaker.org folder
  * Run ``node scripts/create-tables.js``
  * Edit ``scripts/test-data.sql`` and replace it's contents with:

      ```sql
        INSERT INTO clients VALUES
          ( 'test',
            'test',
            '["password", "authorization_code"]'::jsonb,
            '["code", "token"]'::jsonb,
            'http://localhost:3500/callback' )
      ```

  * Run ``node scripts/test-data.js``
    * You'll see a ``INSERT 0 1`` message if successful

**2. Sign In**

To publish locally, you'll need an account.
* Go to [http://localhost:3000/account](http://localhost:3000/account)
* Click ``Join Webmaker`` and complete the process, you can use a fake email
* When you've created your account, click ``Set permanent password instead``
  * This lets you authenticate your account without needing email
* Go back to Thimble and Log In with your new account

## Running the parts
This is the list of commands to get each part up and running.

* Thimble ``npm start``
* Bramble ``npm start``
* PostgreSQL Database ``postgres -D /usr/local/var/postgres``
* Webmaker ID server ``npm start``
* Webmaker Login Server ``npm start``
* Webmaker Publishing Server and Published Projects Server ``npm start``