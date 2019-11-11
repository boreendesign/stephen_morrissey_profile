---
title:  Use python to create nodes in Drupal from a CSV file
summary: An example of creating a Drupal site from imported nodes using Python from a CSV
tags:
- Python
- Drupal
date: "2019-11-08T00:00:00Z"

# Optional external URL for project (replaces project detail page).
external_link: ""

image:
  caption: Creating Drupal nodes with Python
  focal_point: Smart

links:
- icon: twitter
  icon_pack: fab
  name: Follow
url_code: "https://github.com/boreendesign/create_drupal_nodes_python/blob/master/Create%20Drupal%20Nodes%20with%20Python.ipynb"
url_pdf: ""
url_slides: ""
url_video: ""


---
# Use python to create nodes in Drupal from a CSV file

## Steps
1. Create a Drupal 8 Site locally using composer
2. Setup the Rest API
3. Create and run the python script to create the content
4. Import content from a CSV

<span style="color:red">*GOTCHA'S PLEASE READ BELOW*</span>

1. In newer versions of Drupal 8, the Rest UI doesnt seem to always work, I would recommend creating by manually as below. Also install Drush and use `drush cr` to clear the cache if you get any network errors
2. If you run into issues on drupal, try using `drush cr` and POSTMAN is a great tool to test out the endpoints to make sure everything is setup correctly

## Create a Drupal 8 site locally using composer

One of the great reasons to use composer to create your Drupal 8 site is that you can export configurations and iport them on your live site. (This was such a pain in earlier drupal sites but now this is major pain has a great solution).

1. Use your webserver of choice (on MAC I use MAMP free version and will reference this as we go through the tutorial)
2. Create your database, on MAMP
   1. Go to 'Open WebStart page'
   2. Tools -> PHPMYADMIN
   3. Databases -> Create Database -> Enter the name of your Database (for this tutorial : drupal_8_python)
   4. You may also need to create a user for this database and assign if you have not already done so (for this tutorial : username:drupal_8_python, password:drupal_8_python_2020)
3. Add your new url to the hosts file, on MAC run `sudo atom /etc/hosts ` and then enter `127.0.0.1 drupal_8_python.local`
4. Create your vhosts in the MAMP folder, if you have the ATOM editor installed `atom /Applications/MAMP/conf/apache/extra/httpd-vhosts.conf`
   1. Add the following (see code snippet below - Apache Vhosts Code Snippet)
5. Install composer (lots of info online for this for different operating systems)
6. Go to the folder on your local system where you want to install your new drupal 8 site `composer create-project drupal-composer/drupal-project:8.x-dev INSERT_YOUR_NEW_SITE_HERE --no-interaction` for this example we will use `composer create-project drupal-composer/drupal-project:8.x-dev drupal_8_python --no-interaction`
7. Restart your webserver
8. Then vist `http://drupal_8_python.local` and continue with the installation of your new site entering the database details from earlier
9. You should then be able to login at `http://drupal_8_python.local/user`
10. Thats the end of this part, you have now gotten Drupal 8 successfully installed with Composer, nice job and you now can do configuration exports when you want to go to production!


### Apache Vhosts Code Snippet

```python
<VirtualHost *:80>
      DocumentRoot [INSERT LOCATION TO YOUR DRUPAL FOLDER]/drupal_8_python/web
      ServerName drupal_8_python.local

      <Directory [INSERT LOCATION TO YOUR DRUPAL FOLDER]/drupal_8_python/web>
          Options FollowSymLinks MultiViews
          Order deny,allow
          Allow from all
      </Directory>
</VirtualHost>
```


## Setup the Rest API
1. Enable the following modules by going to /admin/modules
  * HAL
  * HTTP Basic Authentication
  * RESTful Web Services
  * Serialization
2. Now we are going to install the Rest UI module on the command line with `composer require drupal/restui`
3. Install the module through EXTEND on your drupal site and go to `/admin/config/services/rest/resource/entity%3Anode/edit`
Configuration Screen: ![Alt](/project/import_to_drupal/config_ui.png "Config Screen")
4. Now create a sample piece of content so we have something to search for to test our rest API
5. We now should also create a user and role to access our API, I created a role call `api_access` and a user called 'api_admin', make sure that the api_admin user has the api_access role
6. Next go to the permissions for this user and make sure they have access to create, view and delete the content you want to be able to create later using Python or another language
7. You can now test this out if you like with the POSTMAN tool that you can download (Go to Screenshots below to see what to put in each tab)
   1. To get the value for X-CSRF-Token, go to `[YOUR DOMAIN]/session/token` and paste in the value, should be something like `yvxnn50NYMfMF2IEIn6s7vMvW3CQQVLxcb6njPU3InA`
8. IF THE ABOVE FAILS :Go to the command line and type `drush cex`
9. Now from the cmd line 'atom config/sync/rest.resource.entity.node.yml'
10. Check the contents of the file with the code block below - REST IMPORT THROUGH CONFIG
11. You are now ready for the next section which is importing nodes through python

## REST IMPORT THROUGH CONFIG

```
langcode: en
status: true
dependencies:
  module:
    - basic_auth
    - hal
    - node
    - serialization
id: entity.node
plugin_id: 'entity:node'
granularity: resource
configuration:
  methods:
    - GET
    - POST
    - PATCH
    - DELETE
  formats:
    - hal_json
    - json
  authentication:
    - basic_auth
```

## SETUP POSTMAN
Url and Params: ![Alt](/project/import_to_drupal/postman1.png "Config Screen")
Authorization: ![Alt](/project/import_to_drupal/postman2.png "Config Screen")
Headers: ![Alt](/project/import_to_drupal/postman3.png "Config Screen")
Body: ![Alt](/project/import_to_drupal/postman4.png "Config Screen")


## Create and run the python script to create the content

Below is the code that you can find in the attached Jupyter notebook or run straight as a python file. This was converted from the POSTMAN commands from above.



```python
# NOTE IF THERE ARE ERRORS - TRYING DRUSH CR
import requests

base_url = 'http://drupal_8_python.local'

#Get CSRF token
token = str(requests.get(base_url + '/session/token').text)
drupal_endpoint = base_url + '/node?_format=hal_json'

#Set all required headers
drupal_headers = {'Content-Type':'application/hal+json',
    'X-CSRF-Token':token
}

#Include all fields required by the content type, using basic examples but from this you can add any fields including custom ones
drupal_payload =  '''{
    "_links": {
      "type": {
        "href": "'''+base_url+'''/rest/type/node/page"
      }
    },
    "title":[{"value":"Sample Drupal Node Page"}],
    "body":[{"value":"Body with some sample content"}]
    }'''





```


```python
#Post the new node (a Contact) to the endpoint.
r = requests.post(drupal_endpoint, data=drupal_payload, headers=drupal_headers, auth=('api_user','PASSWORD'))


#Check was a success
if r.status_code == 201:
    print("Success")
else:
    print("Fail")
    print(r.status_code)
    print(r.text)

```

    Success

## Import content from a CSV
I will be updating the python script to import from a CSV but please check out another article I wrote from scraping, it should be pretty easy in the meantime to use this to create a CSV and then import with the above script.[SCraping data to a csv file with Python](/project/python-scraping/)


*I wrote all of this as I was going through the steps from scratch, except the initial installs of composer, jupyter notebooks, drush and postman, please do let me know in the comments if I got anything wrong, missed anything or need to go into detail or provide external links to, thanks and I hope it helps a few of you, this all took me quite some time to work through the kinks, hopefully it will save you the time. *

[Get the code](https://github.com/boreendesign/create_drupal_nodes_python/blob/master/Create%20Drupal%20Nodes%20with%20Python.ipynb)
