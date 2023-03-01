##### ADDITIONAL STEPS FOR MIGRATION OF API MANAGER DATASTORE AND CONFIGURATIONS

Author: Michael Hayes
Description: Used for migrations on ColdFusion API manager and is intended for use by Media3 and organizations Media3 explicitly provides the documentation to.
Date Created: 2022-12-16 14:45:56 Friday

------------
Original Documentation Reference: [Migrate data in ColdFusion API Manager](https://helpx.adobe.com/coldfusion/kb/migrate-redis-data-coldfusion-api-manager.html "Migrate data in ColdFusion API Manager")
Redis CLI Documentation: [Redis CLI Commands](https://redis.io/commands/ "Redis CLI Commands")

------------
### THE STEPS IN THE ORIGINAL DOCUMENTATION REFERENCE ARE A PRE REQUISITE TO THESE STEPS. PLEASE PERFORM THOSE IF YOU HAVE NOT DONE SO ALREADY
###### PRIOR TO DOING ANY OF THE CHANGES STOP ALL SERVICES ASSOCIATED WITH THE API MANAGER ON THE DESTINATION SERVER THAT YOU ARE IMPORTING CONFIGURATIONS TO

- *SOURCE COLDFUSION SERVER MAY REMAIN RUNNING AS TO NOT CAUSE DOWNTIME*

- *!! IF YOU NEED TO RESET THE COLDFUSION API ADMIN PASSWORD A RESTART OF THE SERVICE IS REQUIRED !!*
- if this api is considered high traffic a window to perform this step will need to be coordinated with the customer / organization

------------


##### In this example the {{source}} is a Coldfusion2016 install and the {{destination}} is a ColdFusion2021 install so when you see the following place holders, assume they contain the values displayed below.

	{{SOURCE_API_ROOT}} = C:\ColdFusion2016APIManager
	{{DESTINATION_API_ROOT}} = C:\ColdFusion2021APIManager
	{{REDIS_PASSWORD}} =  Value is located at **{{SOURCE_API_ROOT || DESTINATION_API_ROOT}}\conf\config.xml** from the block of xml shown below.


```xml
<redis>
	<cluster>false</cluster>
	<client>
		<host>127.0.0.1</host>
		<port>6379</port>
		<password>{{REDIS_PASSWORD}}</password>
		<pooling>${sys:apim.home}/conf/pooling.properties</pooling>
	</client>
</redis>
```

------------


###### STEP 1: CHANGE THE CLUSTER NAME IN THE FOLLOWING LOCATIONS AND UPDATE APPEND ONLY TO BE NO IN THE REDIS CONFIGURATION FILE.

- [x] Change the value in the following location
	Location: **{{DESTINATION_API_ROOT}}**\conf\default_conf.xml
		* <clusterName>groot2021-elasticsearch</clusterName>
		** <clusterName>groot-elasticsearch</clusterName>
		
		
- [x] Change the value in the following location 
	Location: **{{DESTINATION_API_ROOT}}**\database\analytics\config\elasticsearch.yml
		**CURRENT VALUE** : cluster.name: groot2021-elasticsearch
		**UPDATED VALUE** : cluster.name: groot-elasticsearch
		
		
- [x] Change the value in the following location 
	Location: **{{DESTINATION_API_ROOT}}**\database\datastore\redis.windows.conf.properties
		**CURRENT VALUE** : appendonly yes
		**UPDATED VALUE** : appendonly no
------------


###### STEP 2: COPY OVER THE FOLLOWING FILES FROM THE SOURCE SERVER

- [x] {{SOURCE_API_ROOT}}\database\datastore\dump.rdb
- [x] {{SOURCE_API_ROOT}}\conf\seed.properties


------------


###### STEP 3: COPY OVER THE KEYSTORE BY EXPORTING THE CONFIGURATIONS ON THE SOURCE SERVER AND IMPORTING TO THE DESTINATION SERVER

- [x] use the endpoint http://localhost:9000/admin/#/keystores for importing
- [x] *ONLY IF JAVA MAXIMUM SECURITY IS REQUIRED* Copy contents from C:\{{SOURCE_API_ROOT}}\jre\lib\security 

------------

###### STEP 4: LOGIN TO REDIS-CLI using C:\{api-manager-root}\database\datastore\redis-cli.exe AND run the following commands.

* you may need to start the redis server manually while the service is not running by typing .\redis-server.exe to allow for commands to be ran*
- [x] Check logs at {{DESTINATION_API_ROOT}}\logs\apimanager.log and double check the steps if the service still does not start.
- [x] COMMAND: info
- [x] COMMAND: CONFIG SET requirepass {{REDIS_PASSWORD}}
- [x] COMMAND: info




**WARNING**: AT THIS POINT YOU SHOULD BE PROMPTED FOR A PASSWORD. IF YOU ARE NOT EXIT AND RE ENTER REDIS-CLI AND TRY AGAIN. IF YOU ARE STILL NOT PROMPTED YOU DID SOMETHING WRONG
- [x] COMMAND: auth {{REDIS_PASSWORD}}
- [x] COMMAND: info 



**PLEASE NOTE**: If the previous steps were successful you will see the number of keys inside of the redis cluster match that of the source server you are migrating from. Exit out of redis-cli, make sure you stop redis-server if you started it manually and confirm the service starts back up.
	
