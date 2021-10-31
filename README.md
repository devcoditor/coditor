# Visual Editor

By leveraging WASM for software containerization, we create universal binaries that work live and anywhere without modification, including operating systems like Linux, macOS, Windows, and also web browsers. Wasm automatically sandboxes applications by default for secure execution, shielding the host environment from malicious code, bugs and vulnerabilities in the software it runs. 

## Manual Installation

### Prerequisites for Manual Installation
In order for Visual Editor to be installed correctly, the following dependencies need to be installed:

- Node.js 4.x or above (see note below)
  - **Note:** The login.webmaker.org dependency needs a node version of 4.x only while all the other dependencies work with a node version of 4.x and above (Visual Editor requires node 6.11.1 or above). We suggest installing [NVM](https://github.com/creationix/nvm) to allow the use of multiple versions of node.
- [Brackets (Bramble)](#installing-brackets-bramble)
- [Webmaker ID server](#idwebmakerorg)
- [Webmaker Publishing Server](#publishwebmakerorg)
- [Postgresql 9.4 or above (for the publish.webmaker.org dependency)](#postgresql)
- g++ 4.8 or above (for the login.webmaker.org dependency)
- [Webmaker Login Server](#loginwebmakerorg)

The following is an abbreviated guide to getting it all set up. Please see each server's README for more details.

### Manually Installing the Parts
Please note: On Windows, use ``copy`` instead of ``cp``

#### Visual Editor
* Fork and clone this repository
* Run ``npm install`` to install dependencies
* Run ``npm run env`` to create an environment file
* Run ``npm start`` to start the server

#### id.webmaker.org
* Clone https://github.com/mozilla/id.webmaker.org
* Run ``cp sample.env .env`` to create an environment file
* Run ``npm install`` to install dependencies
* Run ``npm start`` to start the server

#### login.webmaker.org
* Clone https://github.com/mozilla/login.webmaker.org
* Run ``npm install`` to install dependencies
* Run ``cp env.sample .env`` to create an environment file
* Run ``npm start`` the server

#### PostgreSQL
* Run ``initdb -D /usr/local/var/postgres`` to initialize PostreSQL
  * If this already exists, run ``rm -rf /usr/local/var/postgres`` to remove it
* Run ``postgres -D /usr/local/var/postgres`` to start the PostgreSQL server
* Run ``createdb publish`` to create the Publish database

#### publish.webmaker.org
* These steps assume you've followed the PostgreSQL steps above, including creating the publish database.
* Clone https://github.com/mozilla/publish.webmaker.org
* Run ``npm install`` to install dependencies
* Run ``npm run env``
* Run ``npm run knex`` to seed the publish database created earlier
* Run ``npm start`` to run the server

Once everything is ready and running, Visual Editor will be available at [http://localhost:3500/](http://localhost:3500/)

### Getting Ready to Publish
To publish locally, you'll need to do the following...

#### Teach the ID server about the Publish server

* Run ``createdb webmaker_oauth_test`` to create a test database
* In your id.webmaker.org folder
  * Run ``node scripts/create-tables.js``
  * Edit ``scripts/test-data.sql`` and replace its contents with:

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

#### Sign In

To publish locally, you'll need an account.
* At the top right corner of the Visual Editor main page click ``Sign In`` if you have an account or click ``Create an account`` and complete the process, you can use a fake email
* When you've created your account, you will automatically be logged in
* You will be redirected to the Visual Editor main page, and you can start working!

It's that simple! You are now ready to start using Visual Editor to its full potential!

Localization
----------------------


##### Our Localization Community


Invalidating CloudFront
----------------------

To invalidate the production CloudFront distribution, make sure you have correct credentials set up in your env file. Then run `node invalidate.js`. Alternatively, if you have access to the Heroku deployments, run the invalidation as a one-off dyno with `heroku run npm run invalidate`

Concurrency
-----------

Visual Editor uses the [throng](https://www.npmjs.com/package/throng) module to leverage Node's [Cluster API](https://nodejs.org/api/cluster.html) for concurrency. To specify the number of server processes to start set `WEB_CONCURRENCY` to a positive integer value.
