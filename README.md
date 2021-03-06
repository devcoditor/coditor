# Web Maker Studio

## Setup/Installation

Web Maker Studio interacts with the Publish API (source managed in [publish.webmaker.org](https://github.com/devcoditor/publish.webmaker.org)) to store users, projects, files and other content as well as publish user projects.

For authentication and user management, Web Maker uses *Webmaker OAuth* which consists of the Webmaker ID System (source managed in [id.webmaker.org](https://github.com/devcoditor/id.webmaker.org)) and the Webmaker Login API (source managed in [login.webmaker.org](https://github.com/devcoditor/login.webmaker.org)).

All three services are bundled together using Git subtrees to be run together using Vagrant, or, they may be run separately with Web Maker [manually](#manual-installation).

**Note:** The Git subtree bundle mentioned above for use with the automated installation can be found in the `/services` folder. It contains a subtree for each of the three services. These subtrees are not automatically kept in sync with their corresponding service's parent repositories. If you need to update one of the subtrees to match the history of its parent repository, follow these instructions:

- Create a separate branch and checkout to it.
  
- Run the following to get the history of the service's repository:

  ```

  git fetch https://github.com/devcoditor/<service's repository name> <branch name>
  ```

  Replace `<service's repository name>` with the remote repository name of the service you are trying to update and `<branch name>` with the name of the branch on that repository you want to update the subtree with.<br>
  For e.g. `git fetch https://github.com/devcoditor/publish.webmaker.org master`.
  - Now to update the subtree, run:

  ```
  git subtree pull --prefix services/<service's repository name> https://github.com/devcoditor/<service's repository name> <branch name> --squash
  ```

  Replace `<service's repository name>` and `<branch name>` with the same values you used in the previous command.<br>
  For e.g. `git subtree pull --prefix services/publish.webmaker.org https://github.com/devcoditor/publish.webmaker.org master --squash`.
  - Update your remote branch with this new change.
  - Open a pull request to have the subtree update reviewed and merged.

## Manual Installation

### Prerequisites for Manual Installation

In order for Web Maker to be installed correctly, the following dependencies need to be installed:

- Node.js 4.x or above (see note below)
  - **Note:** The login.webmaker.org dependency needs a node version of 4.x only while all the other dependencies work with a node version of 4.x and above (Web Maker requires node 6.11.1 or above). We suggest installing [NVM](https://github.com/creationix/nvm) to allow the use of multiple versions of node.
- [EditorX](#installing-editorx)
- [Webmaker ID server](#idwebmakerorg)
- [Webmaker Publishing Server](#publishwebmakerorg)
- [Postgresql 9.4 or above (for the publish.webmaker.org dependency)](#postgresql)
- g++ 4.8 or above (for the login.webmaker.org dependency)
- [Webmaker Login Server](#loginwebmakerorg)

The following is an abbreviated guide to getting it all set up. Please see each server's README for more details.

### Manually Installing the Parts

Please note: On Windows, use ``copy`` instead of ``cp``

#### Installing EditorX

- Fork the [EditorX repository](https://github.com/devcoditor/editorx) and then clone it to your local machine using `git clone --recursive https://github.com/<your_username>/editorx.git` (replace `<your_username>` with your Github username for the account you forked EditorX into)
- In the cloned repository directory, run `npm install` to install the dependencies for EditorX
- Run `npm run build` to create the built editor files that will be used by Thimble
- Run `npm start` to start a server that will allow the editor to be accessed on [http://localhost:8000/src](http://localhost:8000/src)
 -- You can find out more information about setting up EditorX locally by referring to the instructions [here](https://github.com/devcoditor/editorx#how-to-setup-editorx-in-your-local-machine)

#### Web Maker

- Fork and clone this repository

- Run ``npm install`` to install dependencies

- Run ``npm run env`` to create an environment file

- Run ``npm start`` to start the server

#### id.webmaker.org

- Clone <https://github.com/devcoditor/id.webmaker.org>

- Run ``cp sample.env .env`` to create an environment file
- Run ``npm install`` to install dependencies
- Run ``npm start`` to start the server

#### login.webmaker.org

- Clone <https://github.com/devcoditor/login.webmaker.org>

- Run ``npm install`` to install dependencies
- Run ``cp env.sample .env`` to create an environment file
- Run ``npm start`` the server

#### PostgreSQL

- Run ``initdb -D /usr/local/var/postgres`` to initialize PostreSQL
  - If this already exists, run ``rm -rf /usr/local/var/postgres`` to remove it

- Run ``postgres -D /usr/local/var/postgres`` to start the PostgreSQL server
- Run ``createdb publish`` to create the Publish database

#### publish.webmaker.org

- These steps assume you've followed the PostgreSQL steps above, including creating the publish database.

- Clone <https://github.com/devcoditor/publish.webmaker.org>
- Run ``npm install`` to install dependencies
- Run ``npm run env``
- Run ``npm run knex`` to seed the publish database created earlier
- Run ``npm start`` to run the server

Once everything is ready and running, Web Maker will be available at [http://localhost:3500/](http://localhost:3500/)

### Getting Ready to Publish

To publish locally, you'll need to do the following...

#### Teach the ID server about the Publish server

- Run ``createdb webmaker_oauth_test`` to create a test database

- In your id.webmaker.org folder
  - Run ``node scripts/create-tables.js``
  - Edit ``scripts/test-data.sql`` and replace its contents with:

      ```sql
        INSERT INTO clients VALUES
          ( 'test',
            'test',
            '["password", "authorization_code"]'::jsonb,
            '["code", "token"]'::jsonb,
            'http://localhost:3500/callback' )
      ```

  - Run ``node scripts/test-data.js``
    - You'll see a ``INSERT 0 1`` message if successful

#### Sign In

To publish locally, you'll need an account.

- At the top right corner of the Web Maker main page click ``Sign In`` if you have an account or click ``Create an account`` and complete the process, you can use a fake email

- When you've created your account, you will automatically be logged in

- You will be redirected to the Web Maker main page, and you can start working!

It's that simple! You are now ready to start using Web Maker to its full potential!

#### Invalidating CloudFront

To invalidate the production CloudFront distribution, make sure you have correct credentials set up in your env file. Then run `node invalidate.js`. Alternatively, if you have access to the Heroku deployments, run the invalidation as a one-off dyno with `heroku run npm run invalidate`

#### Concurrency

Visual Editor uses the [throng](https://www.npmjs.com/package/throng) module to leverage Node's [Cluster API](https://nodejs.org/api/cluster.html) for concurrency. To specify the number of server processes to start set `WEB_CONCURRENCY` to a positive integer value.
