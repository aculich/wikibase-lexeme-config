Instructions to set up the local wikibase via Docker:

First follow most of the instructions from the MediaWiki site for an Extended Install (https://www.mediawiki.org/wiki/Wikibase/Docker). That is:

* 1.) Install Docker
* 2.) Execute
```
git clone https://github.com/wmde/wikibase-release-pipeline
cd wikibase-release-pipeline
git checkout tags/wmde.20
```

* 3.) Create a launch directory by copying the /example folder of the above repository to a location of your choice, modifying the commands below appropriately:
```
mkdir $HOME/wbdocker
cp -r example/* $HOME/wbdocker
cd $HOME/wbdocker
mv template.env .env
```

* 4.) Personalize the following fields of the .env file in the launch directory ():
MW_ADMIN_NAME
MW_ADMIN_PASS
MW_ADMIN_EMAIL
DB_NAME
DB_USER
DB_PASS

NOTE (MO): I still have not been able to access the sql database within the wikibase container using the DB credentials above, following guidelines from this site (https://stuff.coffeecode.net/2018/wikibase-workshop-swib18.html#_about_the_wikidata_query_service_wdqs).

* 5.) Add the following line to the .env file as well:
WIKIBASE_MAX_DAYS_BACK=365

* 6.) You also need to do make a modification to the docker-compose.extra.yml file in your launch directory. In that file, under the heading wdqs-updater's environment section, add the last line containing the variable WIKIBASE_MAX_DAYS_BACK as indicated:
```
      - WIKIBASE_HOST
      - WDQS_HOST=wdqs.svc
      - WDQS_PORT=9999
      - WIKIBASE_MAX_DAYS_BACK=${WIKIBASE_MAX_DAYS_BACK}
```

* 7.) Execute the extended installation (making sure your Docker Desktop is running):
```
docker-compose -f docker-compose.yml -f docker-compose.extra.yml up -d
```

* 8.) Obtain a copy of the WikibaseLexemes package by following the download link at the following site (https://www.mediawiki.org/wiki/Extension:WikibaseLexeme). As of 4/14/2024, you should choose the 1.41 MediaWiki version from the drop down menu.

* 9.) (Assuming you are in the launch directory which has WikibaseLexemes as a subdirectory), copy the WikibaseLexemes folder to the \extensions folder in the wikibase container using the following syntax:
```
docker cp ./WikibaseLexeme LAUNCH_FOLDER_NAME-wikibase-1:/var/www/html/extensions/
```

* 10.) Also put a copy of the WikibaseLexemes folder in the jobrunner container:
```
docker cp ./WikibaseLexeme LAUNCH_FOLDER_NAME-wikibase-jobrunner-1:/var/www/html/extensions/
```

* 11.) You need to modify the contents of the LocalSettings.php file that is within the wikibase container. One way to do it is to first copy it from inside the container to the outside:
```
docker cp LAUNCH_FOLDER_NAME-wikibase-1:/var/www/html/LocalSettings.php LocalSettings.php  
```

* 12.) Then add the following lines to LocalSettings.php:
```
wfLoadExtension('WikibaseLexeme');

define('Lexeme', 146);
define('Lexeme_talk', 147);
```
NOTE (MO): I don't know if it makes a difference which numbers you chose for the last two lines, which define the namespace for lexemes. These are the numbers used in Wikidata.

* 11.) Copy LocalSettings.php back to the wikibase container:
```
docker cp LocalSettings.php LAUNCH_FOLDER_NAME-wikibase-1:/var/www/html/LocalSettings.php    
```





* 14.) Restart the containers using docker compose restart