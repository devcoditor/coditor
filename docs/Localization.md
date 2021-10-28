Thimble uses [Pontoon](https://pontoon.mozilla.org) to enable contributors to localize strings. 
This document however is meant for developers who would like to contribute to Thimble and would like an overview of the l10n process.

**_Note_**: Because Thimble includes a forked version of [Brackets](https://github.com/mozilla/brackets), you should also refer to the [Brackets l10n page in the wiki](https://github.com/mozilla/brackets/wiki/Localizing-Bramble).

Here is an overall architecture schematic of the Localization workflow for both Brackets and Thimble. _The green lines indicate the path followed for modifying strings on Thimble while the blue lines indicate the path followed for modifying strings on Brackets_:

![thimble l10n](https://cloud.githubusercontent.com/assets/5816130/13713597/ca32b87a-e797-11e5-88fc-2f519e61b935.png)

# Strings
Any string that can be translated into other languages should be contained in either the `server.properties`, `client.properties`, or `server-client-shared.properties` files for the `en-US` locale which can be found here - [`locales/en-US`](https://github.com/mozilla/thimble.mozilla.org/blob/master/locales/en-US). The format for adding a string to this file is: 
```java
descriptiveStringKey=string
```
where `descriptiveStringKey` represents a camel-cased name that will be used to refer to the string.

Comments can be added using a `#` sign at the beginning of the comment on a new line. This string document is structured in such a way that it delimits different components of the Thimble app (for e.g. Editor, Homepage, etc.) using comment blocks. Any new string should be placed in the appropriate section accordingly.

If you are adding a string that is meant to be used in the client viz. in files in the `public/` folder, add your strings to `client.properties`. If your strings are being rendered on the server views or used in the server viz. in files in the `server/` or `views/` folders, add your strings to the `server.properties` file. If your string is going to be used in both the client and the server, add it to `server-client-shared.properties`.

While the strings in the `.properties` files are the "source of truth," the actual string files used at run time are JSON generated versions of these files. These JSON files **must** be generated for any of the following steps to work.

The necessary JSON strings can be generated using the following command:

     npm run localize

This will result in the JSON strings for each locale being written to the `dist/` folder.

# Server-side localization
We use [node-webmaker-i18n](https://github.com/mozilla/node-webmaker-i18n) as our localization library and [Nunjucks](http://mozilla.github.io/nunjucks) to render localized strings. 

### Static strings in a view
These mainly include strings that are static on a page (for e.g. "Sign in") and do not require client-side processing. Most of these strings are located in one of the [views](https://github.com/mozilla/thimble.mozilla.org/tree/master/views). If you need to include a new string, follow these steps:
* Add your string to the appropriate `.properties` file (either `server.properties` or `server-client-shared.properties`).
* Select the appropriate `views/` html file containing your string and replace the string with its key (as specified in the `.properties` file) in the following format: `{{ gettext("key") }}`.
* **Note:** _If the string contains HTML markup, add the_ `safe` _keyword like so:_ `{{ gettext("key") | safe }}`.

### Dynamic strings in server logic
Sometimes, strings are not rendered in a view. For example, the default project name i.e. "Untitled Project" is not rendered in a view but is used in the server logic when creating a new project. To acquire the translated string for such situations, follow these steps:
* Add your string to the appropriate `.properties` file (either `server.properties` or `server-client-shared.properties`).
* In the specific server route logic, the `req` and `res` objects can be used to get the translated strings. For e.g. `req.gettext("key", "fr")`, where `"key"` is the key name, and `"fr"` the locale to use.
* Refer to https://github.com/mozilla/node-webmaker-i18n#gettext for more information.

### Links in views
If you would like to add a link to a route that is managed by Thimble, make sure you precede every absolute route with `{{ locale }}`. For example,
```html
<a href="/projects/new">New Project</a>
```
should include the locale, like so
```html
<a href="/{{ locale }}/projects/new">New Project</a>
``` 

# Client-side localization
Client scripts only use Nunjucks to render a localized string.

### How it works
Every client script that is contained in the `public/` folder is localized at build-time. When the server is started, for each locale, a localized version of the client script is generated (this is done by `npm run localize-client` which automatically runs when the server is started). 

If in `development` mode, these scripts will be organized in the same file hierarchy as the `public/` folder and will be located in a folder called `client/` under each locale, for e.g. `client/fr/my-script.js`. Any changes made to the scripts in the `public/` folder will be automatically reflected in their corresponding localized versions. This means that changes to scripts do not require a restart of the server.

If in `production` mode, these scripts will first need to be built using `grunt requirejs:dist` after which the built files will be automatically localized in the `dist/` folder once the server starts.

### RequireJS configuration
If you want to edit the requirejs config to include paths to custom scripts (i.e. scripts in the `public/` folder), make sure you follow the format: `/{{ locale }}/path/to/script`

### UI strings
To use a localized string that will appear to the user,
* Add your string to the `client.properties` file.
* Replace the string in your script with `{{ key }}`, where `key` represents the corresponding key for the string.

# Pontoon
Every time a new string is added to the `.properties` files in `locales/en_US` and is committed, it will show up as an "unlocalized string" on Pontoon for each locale. There, contributors will localize the string and once accepted, it will show up on the staging server at https://bramble.mofostaging.net