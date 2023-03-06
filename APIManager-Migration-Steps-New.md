##### ADDITIONAL STEPS FOR MIGRATION OF API MANAGER DATASTORE AND CONFIGURATIONS

Author: Michael Hayes
Description: Used for migrations on ColdFusion API manager and is intended for use by Media3 and organizations Media3 explicitly provides the documentation to.

Date Created: 2023-03-06 12:51:56 Monday

------------


	 ## Step 1.) Stop Service on the destination server
	 
	```
	 Stop-Service ColdFuson*api*
	```
	
	## Step 2.) Copy the Dump.rdb and appendonly.aof file from the source server & set appendonly to be no inside the conf.properties file
		
			Example: C:\ColdFusion2018APIManager\database\datastore\dump.rdb
			Example: C:\ColdFusion2018APIManager\database\datastore\appendonly.aof
	
			Example for appendOnly: 
			
			C:\ColdFusion2021APIManager\database\datastore\redis.windows.conf.properties
	
	## Step 3.) Copy The Password from the source/destination servers config.xml. The following script will get the password and copy it to your clipboard if ran on either source or destination server. 
	
			Example: C:\ColdFusion2018APIManager\conf\config.xml
			Note: {{REDIS_PASSWORD}} is the placeholder for the password
			
			```
			$paths = @(

				"C:\ColdFusion2018APIManager\",
				"C:\ColdFusion2021APIManager\"
			)

			foreach ($path in $paths) {
				if (Test-Path $path -PathType Container) {
				   $SourceLocation = $path;
				}
			};
			
				
			function Get-RedisPassword {
				$xmlFile = "$SourceLocation\conf\config.xml"
				$xml = [xml](Get-Content $xmlFile)

				$password = $xml.root.redis.client.password
				
				return $password
			}


			$RedisPassword = Get-RedisPassword;
			
			
			Set-ClipBoard $RedisPassword
			
			```
	
	## Step 4.) Navigate to Redis-cli.exe and authenticate using the destination servers {{REDIS_PASSWORD}}
		```
			$paths = @(

				"C:\ColdFusion2018APIManager\",
				"C:\ColdFusion2021APIManager\"
			)

			foreach ($path in $paths) {
				if (Test-Path $path -PathType Container) {
				   $SourceLocation = $path;
				}
			};
			
				
			function Get-RedisPassword {
				$xmlFile = "$SourceLocation\conf\config.xml"
				$xml = [xml](Get-Content $xmlFile)

				$password = $xml.root.redis.client.password
				
				return $password
			}


			$RedisPassword = Get-RedisPassword;
			
			
			Set-ClipBoard $RedisPassword
			CD $SourceLocation\database\datastore;


            Start-Process .\redis-server.exe
			
			echo "keys *" | .\redis-cli.exe -a $RedisPassword
			
			
		
		```
		
		## Step 5.) Inside of redis-cli run the following
		
		```
			CONFIG SET requirepass {{SOURCE_SERVER:REDIS_PASSWORD}}
			AUTH {{SOURCE_SERVER:REDIS_PASSWORD}}
			save
			shutdown
		```
		
		## Step 6.) Wait for the dump.rdb file to save. IF it is a large file it can take a minute or two.
			[x] Change appendonly back to yes and then confirm you can login to redis-cli.exe again using the new password.
		
		
		
		
		
		
		
		

