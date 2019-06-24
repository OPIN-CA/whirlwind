This is a Composer-based installer for the OPIN Whirlwind distribution: a forked
version of the [Lightning](https://www.drupal.org/project/lightning) Drupal
distribution. This fork is being developed by OPIN Software Inc.

## Get Started

In order to create a new project using this distribution run the following
command:
```
$ composer create-project opin/whirlwind MY_PROECT --remove-vcs --repository="https://raw.githubusercontent.com/OPIN-CA/whirlwind/master/"
```
Composer will create a new directory called MY_PROJECT containing a ```docroot``` directory with a full Whirlwind code base therein. The last step of the ```composer create-project``` will ask you if you want to delete the VCS (.git, etc.). Say yes to this. Next run:

```
cd MY_PROJECT
git init .
git add .
git commit -m "Initial commit"
```

to initialize a clean repo and track your files.

You can then perform a regular Drupal install in your prefered way via Dev Desktop, or a VM, or any other method.

## Dev Desktop into an existing site

If you are setting up a new environment using Acquia Dev Desktop to override a site created via Acquia's dashboard you will need to do so using the "composer install" command (as opposed to "create-project"). Pull the environment (code and database) down using Dev Desktop, delete the environment's "docroot" folder, delete and recreate the database, and copy whirlwind's composer.json file to the root directory. Next run:

```
composer install
git add .
git commit -m "Switch to whirlwind"
drush si lightning
drush en module_filter admin_toolbar admin_toolbar_tools adminimal_admin_toolbar fitzroy browsersync linkit paragraphs svg_image_field twig_vardumper twig_field_value twig_tweak yaml_content devel kint field_group dropzonejs_eb_widget menu_block pathauto paragraphs_type_permissions -y
```

You can now push up to Acquia and have a fancy Whirlwind based site.

## Maintenance
```drush make```, ```drush pm-download```, ```drush pm-update``` and their ilk are the old-school way of maintaining your code base. Forget them. You're in Composer land now!

Let this handy table be your guide:

| Task                                            | Drush                                         | Composer                                          |
|-------------------------------------------------|-----------------------------------------------|---------------------------------------------------|
| Installing a contrib project (latest version)   | ```drush pm-download PROJECT```               | ```composer require drupal/PROJECT```             |
| Installing a contrib project (specific version) | ```drush pm-download PROJECT-8.x-1.0-beta3``` | ```composer require drupal/PROJECT:1.0.0-beta3``` |
| Installing a javascript library (e.g. dropzone) | ```drush pm-download dropzone```              | ```composer require bower-asset/dropzone```       |
| Updating all contrib projects and Drupal core   | ```drush pm-update```                         | ```composer update```                             |
| Updating a single contrib project               | ```drush pm-update PROJECT```                 | ```composer update drupal/PROJECT```              |
| Updating Drupal core                            | ```drush pm-update drupal```                  | ```composer update drupal/core```                 |

The magic is that Composer, unlike Drush, is a *dependency manager*. If module ```foo version: 1.0.0``` depends on ```baz version: 3.2.0```, Composer will not let you update baz to ```3.3.0``` (or downgrade it to ```3.1.0```, for that matter). Drush has no concept of dependency management. If you've ever accidentally hosed a site because of dependency issues like this, you've probably already realized how valuable Composer can be.

But to be clear: it is still very helpful to use a site management tool like Drush or Drupal Console. Tasks such as database updates (```drush updatedb```) are still firmly in the province of such utilities. This installer will install a copy of Drush (local to the project) in the ```bin``` directory.

### Specifying a version
you can specify a version from the command line with:

    $ composer require drupal/<modulename>:<version>

For example:

    $ composer require drupal/ctools:3.0.0-alpha26
    $ composer require drupal/token:1.x-dev

In these examples, the composer version 3.0.0-alpha26 maps to the drupal.org version 8.x-3.0-alpha26 and 1.x-dev maps to 8.x-1.x branch on drupal.org.

If you specify a branch, such as 1.x you must add -dev to the end of the version.

**Composer is only responsible for maintaining the code base**.

## Source Control
If you peek at the ```.gitignore``` we provide, you'll see that certain directories, including all directories containing contributed projects, are excluded from source control. This might be a bit disconcerting if you're newly arrived from Planet Drush, but in a Composer-based project like this one, **you SHOULD NOT commit your installed dependencies to source control**.

When you set up the project, Composer will create a file called ```composer.lock```, which is a list of which dependencies were installed, and in which versions. **Commit ```composer.lock``` to source control!** Then, when your colleagues want to spin up their own copies of the project, all they'll have to do is run ```composer install```, which will install the correct versions of everything in ```composer.lock```.

## Gitlab configuration

There are two scopes for configuring the variables: group and project.

Group variables apply to all projects within that group. They can be accessed
by going to the group and navigating to
Settings --> CI/CD --> Environment variables.

Project variables apply only to that project. They can be accessed by going
to the project and navigating to
Settings --> CI/CD --> Environment variables.


### Group variables

It is required that you set the following group variables:
* `SSH_PRIVATE_KEY`
* If using Acquia:
  * `ACQUIA_API_EMAIL`
  * `ACQUIA_API_SECRET`

#### `SSH_PRIVATE_KEY` variable
Paste the private key for the CI user account that will be accessing the Dev and
Staging servers.

#### `ACQUIA_API_EMAIL` and `ACQUIA_API_SECRET` variables
These are the Acquia Cloud API credentials for the user who will be logging
into the Acquia server. _This is not an Acquia API token_. 

The credentials can be found by going to your Account in
Acquia cloud, clicking on the Credentials tab and scrolling down to Cloud API.
There you will see the Acquia Cloud API private key.

https://docs.acquia.com/acquia-cloud/develop/api/auth/v1/

### Project variables

#### `SSH_KNOWN_HOSTS` variable
Run `ssh-keyscan` on the Acquia or Pantheon SSH url(s) and paste the output into 
this variable. The `ssh-keyscan` command is likely installed locally and can be
run from anywhere.

For a Pantheon site only the git repo URL needs to be scanned.

Example:
`ssh-keyscan -p 2222 codeserver.dev.cabf666c-666e-666d-a666-666b666ecb6.drush.in`

For an Acquia site the git repo URL _and_ the SSH url need to be scanned. Merge
the output of both keyscan commands and put it in a single SSH_KNOWN_HOSTS
variable. Both URLs can be found in Dev Desktop.

Example:
`ssh-keyscan svn-29885.prod.hosting.acquia.com`
`ssh-keyscan staging-30445.prod.hosting.acquia.com`

https://docs.gitlab.com/ee/ci/ssh_keys/#verifying-the-ssh-host-keys

#### `THEME_DIR` variable
The theme directory name.
For example, if the theme is located at 
`docroot/themes/custom/new_project_theme/` then this variable should be
`new_project_theme`.

#### `GIT_DEPLOY_URL` variable
The git URL for deployment. It will be something such as
`ssh://testsite@svn-29901.prod.hosting.acquia.com:testsite.git`.

#### `ACQUIA_SSH_URL` variable
SSH URL for running commands on the Acquia Dev server.
Such as `testsite.dev@staging-30406.prod.hosting.acquia.com`.

#### `DEV_LIVE_URL` and `TEST_LIVE_URL` variables
URLs for the Dev and Test (Staging) environments.

#### `SITE_ID` variable
For Acquia this would be the long site alias.
Example:
For an alias such as @cl.prod_opinacer29.dev.dev the generic SITE_ID would
be cl.prod_opinacer29.

For Pantheon the site id can be found at the end of the git clone command
provided in "Connection Info".
Example:
For `git clone ssh://codeserver.dev.cf88500f-f3d6-45ed-a8aa-99afa52fbe4e@codeserver.dev.cf88500f-f7d6-45ed-a8aa-96afa52fbe5e.drush.in:2222/~/repository.git new-project`
the site id would be 'new-project'.

#### `PANTHEON_MACHINE_TOKEN` variable
For Pantheon only. A machine token can be generated by going to your account
settings tab on the Pantheon Dashboard. The machine token can only be viewed
once. This is used to log into Pantheon using Terminus to do tasks such as
clearing the cache.

#### `DEV_LIVE_USERNAME` and `DEV_LIVE_PASSWORD` variable
If you're live Dev environment uses HTTP Basic Authorization then put the username and passwords into these variables.
