# apex-source-control
Introducing file-based version-control for Oracle Application Express apps!

* Designed to be used by teams who want to bring their experience in version-control and APEX development together
* Any version-control tool can be used, not just git
* Tested on Linux and OSX (Windows not supported)

## Getting Started
Prerequisites:
* APEXExport.class and APEXExportSplitter.class which come with [APEX 5.0] (http://www.oracle.com/technetwork/developer-tools/apex/downloads/index.html) and above
* An `export APEX_HOME` with `$APEX_HOME/utilities/oracle/apex/*.class` being the location of the above mentioned files
* An installation of [Oracle Instant Client](http://www.oracle.com/technetwork/database/features/instant-client/index-100365.html)
* An `export ORACLE_HOME` with an existing `$ORACLE_HOME/jdbc/lib/jdbc6.jar` 
* `sqlplus` on path

## Installation
#### Method 1: Setting up from another APEX app

1) Make a local directory for your app and in the new directory run npm init and follow the given prompts

2) Add the following lines to your devDependencies in package.json (Note: delete or repurpose the pre-existing scripts value)

	"devDependencies": {
	    "apex-source-control" : "git+ssh://git@github.com:ntreeinc/apex-source-control.git"
	},
	"scripts": {
	   "apex-to-file" : "apex-source-control apex-to-file",
	   "file-to-apex" : "apex-source-control file-to-apex",
	   "new-conf-file" : "apex-source-control new-conf-file",
	   "switch-conf-file" : "apex-source-control switch-conf-file",
	   "read-conf-file" : "apex-source-control read-conf-file",
	   "generate-app-id" : "apex-source-control generate-app-id",
	   "uninstall-apex" : "apex-source-control uninstall-apex"
	},

If you want you can change the npm run commands (under scripts) to anything you'd like.

3) Run the following commands and follow the prompts. The app id, parsing_schema, workspace_name and database connection info should all correspond to the app you want to download.

	npm install
	cd node_modules/apex-source-control
	npm link
	npm run new-conf-file 		#create a config file with the info of the app you want to download
	npm run switch-conf-file 	#tell scripts to use the config file you just created
	npm run apex-to-file 		#download the app locally

You can now set up the application as a git or other version-control repository. Be warned that if you downloaded the app from another developer's copy of the app, or some other version of the app you don't want to overwrite, you should create a new config file and set up a new version of the application.

#### Method 2: Setting up from an APEX export file

1) Make a local directory for your app and in the new directory run npm init and follow the given prompts

2) Add the following lines to your devDependencies in package.json (Note: delete or repurpose the pre-existing scripts value)

	"devDependencies": {
	    "apex-source-control" : "git+ssh://git@github.com:ntreeinc/apex-source-control.git"
	},
	"scripts": {
	   "apex-to-file" : "apex-source-control apex-to-file",
	   "file-to-apex" : "apex-source-control file-to-apex",
	   "new-conf-file" : "apex-source-control new-conf-file",
	   "switch-conf-file" : "apex-source-control switch-conf-file",
	   "read-conf-file" : "apex-source-control read-conf-file",
	   "generate-app-id" : "apex-source-control generate-app-id",
	   "uninstall-apex" : "apex-source-control uninstall-apex"
	},

If you want you can change the npm run commands (under scripts) to anything you'd like.

3) Run:

	npm install
	cd node_modules/apex-source-control
	npm link

5) Copy the export file into your project directory

6) Set up your classpath and run APEXExportSplitter

	export CLASSPATH=$APEX_HOME/utilities:$ORACLE_HOME/jdbc/lib/ojdbc6.jar
	java oracle.apex.APEXExportSplitter $export_file
	
The CLASSPATH variable does not need to be added to your bash profile

7) Rename the generated directory to apex/

8) cd into apex/ and run:

	sed -i 's^@application^@apex/application^g' install.sql

We do this because we need to set the relative path to the install components from the top level directory

9) Remove the old $export_file or place it in a different directory

From here you can now either set up the project as a git/subversion/etc. repository or install into apex using:
	
	npm run new-conf-file
	npm run switch-conf-file
	npm run file-to-apex

### Working with existing apex-source-control project

1) Clone the repository locally

2) Use 'npm install' to install dependencies

3) Run the following commands and follow the prompts. This will set up your config file and put the application into your APEX workspace

	npm run new-conf-file 		#create config file with the info of the app you want to use for version control
	npm run switch-conf-file 	#tell scripts to use the config file you just created
	npm run file-to-apex 		#download the app locally
That's it!

When you run 'npm run new-conf-file' you can either enter the info of the app you are already using to develop (your version will be overwritten by the one in version control) or enter in the info of a non-existent app which will be automatically created after running 'npm run file-to-apex'

## npm scripts Commands
##### npm run apex-to-file
Turn your apex workspace project into a file directory suitable for version control tools
##### npm run file-to-apex
Import your file directory into apex as an apex application. This command will only work with a $PROJECT_HOME/apex/ dir which was generated by APEXExportSplitter
##### npm run uninstall-apex
Uninstall your app from apex
##### npm run create-new-app
Currently not an availible command, may be added in the future.
##### npm run new-conf-file
Creates a new config file using user input. NOTE: As of right now you must run 'npm run switch-conf-file' command to change to the new config file after creating a new one
##### npm run switch-conf-file
Switches the symlink to the config file of user's choice
##### npm run read-conf-file
Outputs the name and contents of the config file currently being used
##### npm run generate-app-id
Logs into your database and automatically generates and unused app-id in order to avoid accidentaly overwritting someone else's app.
Can only be run if your config file is set up with proper database login info (apexappid, parsing_schema & workspace_name not needed).

Note: As of right now the app-id will not automatically be written to your config file (TODO) so you must manually input the number the program gives you.
This script also needs your database login info (in order to check app-ids and whatnot) so your config file must be set up before it can be run.

###Workflow & Project Sanitation
* Ignore config files in version control (.gitignore for git) since you'll likely not want to share login info in version control

TODO write workflow section 

##Config File Explained
Config files are placed in a top level config/ directory.
Each developer has their own config file with their app data which is then refrenced by a symlink named asc.conf (TODO change symlink name).
Config files should never be put under version control (unless you want to share connection data for some reason).
##### apexappid
The unique application id # of your APEX app in Oracle.
Used to tell APEX which application to export.

The application id does not have to already exist in APEX in order to be installed by `npm run file-to-apex` but must not conflict with an app id in a different workspace.
It can be the same as a pre-existing APEX app in the same workspace, but be careful about overwriting work that isn't your own.
##### workspace_name
Name of the workspace where the app is installed or is to be installed to.
The workspace should already exist before usage
##### database_connection
The database_connection info in [JDBC](http://www.orafaq.com/wiki/JDBC) format, i.e. Hostname:Port/SID. Example: `myhost:1521/orcl`.
Used in connect statements such as `sqlplus $username/$password@$database_connection`
##### username
Username of database login
##### password
Password of database login

###Known Issues
#### Deleting a page will always win merges
**To Reproduce**: 
* Have two developers working on different versions of an app (i.e. working on different features to be added).
* Developer 1 deletes a page in the app and pushes to the project's repo.
* Developer 2 merges developer 1's changes and decides to keep the deleted page in the merge.
* When developer 2 reinstalls the app into apex the page will still be deleted.

**Why it occurs**:

Since a page was deleted, it's removed from the install.sql script generated by apex. When git sees the merge of the two developers, only one has changed the install.sql script so it automatically applies those changes to the merge meaning the page is no longer called during install so it's never created in your apex application.

**How to avoid**:

Be extremely careful when deleting pages and consult with your teammates first

**How to fix**:

If you do end up accidently deleting a page then you will have to manually re-add the install lines to the install.sql

#### Random version control noise in application meta-data
TODO expand this section

### Inspiration
These scripts were designed using [this paper] (http://www.rwijk.nl/AboutOracle/psdua.pdf) as reference.
There's a lot of good information about directory structure and general developer workflow so it's definitely worth a read.

It should be noted that if you decide to read the paper we have changed the names/functions of the scripts slightly: file-to-apex=install_apex.sql; uninstall-apex=uninstall_apex.sql; and apex-to-file=apexupdate.sh (Except we don't auto-update using subversion in order to leave more user freedom)
