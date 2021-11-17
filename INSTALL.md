# Instructions to Install Islandora with ArchivesSpace

Islandora is a completely free open-source software framework designed to help institutions and organizations and their audiences collaboratively manage and discover digital assets using a best practice framework.  Islandora is implemented and supported by an ever-growing international community.

Built on a base of Drupal, Fedora, and Solr, Islandora's flexibility empowers users to work with a huge variety of data types (such as image, video, and pdf) and knowledge domains (such as Chemistry and the Digital Humanities) and provides integration with additional viewers, editors, and data processing applications.

## Prereqs
- [Docker](https://docs.docker.com/get-docker/)
- [Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git) (Optional)

I will not outline how to install the prerequisites within this documentation.

## Getting the Code

Open a terminal, and find a location you'd like to store the repository.

Type:

```
git clone https://github.com/Islandora-Devops/isle-dc
```

This will clone the repository to your local machine. Next you'll enter the folder
```cd isle-dc```

Next you'll copy the contents of sample.env and name the copy .env
```
cp sample.env .env
```

### (Optional) Modify the .env file


If you have a custom domain, open the .env file such that
```
DOMAIN=islandora.traefik.me
```
is changed to 
```
DOMAIN=custom.com
```
such that custom.com is your domain/hostname.

***Make Sure that You point the hostname to your server***

If you'd like the server to create it's own logins for each service, you can enable secrets by changing USE_SECRETS to true

```
USE_SECRETS=true
```

The files will be stored within isle-dc/secrets/live/

## Installation of Islandora

In this installation, we will be taking advantage of the demo server to create our instance. There have been multiple issues with missing configuration and modules when using local and custom installation. To start the installation, run
```
make demo
```

During the installation, you will run into it asking for a password. If this happens please type it in.

While you wait islandora to install, grab a coffee or something cuz it will take some time.

## Configuration of Islandora

Most of islandora should be ready, but you'll still need to do some configuration. If you don't have a real domain name pointed at the server, or left the default hostname within the .env file, please open (with administrative privileges) one of the following files.

Windows:
```C:\Windows\System32\drivers\etc\hosts```
Linux:
```/etc/hosts```

And add the following line to your file
```
127.0.0.1 islandora.traefik.me
```

Next you can open a web browser and open the server by opening
[https://islandora.traefik.me/](https://islandora.traefik.me/)
Or by typing the hostname you dedicated to the server.

Once you're connected, log in, and open the contents page. You can delete all of the contents there by clicking the select all checkbox, and selecting delete content in the drop down menu.

From here you're basically all setup.

# Pending Integration Setup

## Setup Islandora for Module Install
The first step before we can proceed to install the necessary modules for integration, you'll first need to remove the existing content on the server.
### DISCLAIMER!!!
If you specified a hostname for the server at setup, make sure you replace islandora.traefik.me on all links containing this. You won't be able to open the links contained here otherwise.

### Logging In
Navigate to [https://islandora.traefik.me/user/login](https://islandora.traefik.me/user/login). Replace islandora.traefik.me with your hostname if you specified a custom one. Here you'll login to the server. If you left secrets off the default login will be

Username: ```Test```
Password: ```islandora```

If you DID enable secrets, the username will be admin, and the password will be stored within the isle-dc at
```isle-dc/secrets/live/DRUPAL_DEFAULT_ACCOUNT_PASSWORD```

### Remove Default Content

Once you've logged in, you can now go to [https://islandora.traefik.me/admin/content](https://islandora.traefik.me/admin/content). You can now remove all of the items on the content page by selecting all content (top checkbox on the left) and selecting delete content in the Action dropdown menu. Next press Apply to selected items and click delete on the confirmation page.

### Suggested Settings
Here is a list of settings you may want to change before proceeding to the module install.
#### File Hash
File has specified which hashing algorithm is used to ensure integrity of files saved to Islandora. By default, SHA-1 is used which is fast, but less secure than SHA-256 which is the best option to use. You can enable/replace SHA-1 by navigating to [https://islandora.traefik.me/admin/config/media/filehash](https://islandora.traefik.me/admin/config/media/filehash). You will be greeted by the menu where you can check and uncheck which hashing algorithms you'd like to use. You can use multiple, but it is suggested to just use 1 for best performance. If SHA-1 is checked, uncheck it and check SHA-256. Next press Save configuration to save SHA-256 as the hashing algorithm to be used.

#### More to be added in the future


### Module Installation

#### Connecting to the container
In order to get the integrations working, we first need to connect to the drupal container and install modules. In order to do this, we list the containers on the server by running

```docker container ls```

We will be looking for a line with the container id and the container name of islandora/demo:1.0.0-alpha-6 or similar. The following is an example line which 455204e66481 is the container id we're looking for

```455204e66481   islandora/demo:1.0.0-alpha-6         "/init"                  26 minutes ago   Up 26 minutes   80/tcp```

Next we'll connect to the container's bash shell by running the following line where CONTAINER_ID is the same as the one you found from the last step.

```docker exec -it CONTAINER_ID bash```

Now we'll be dropped in the container's bash terminal, and we can begin to install modules.

#### Installing the modules

In order to install the modules we'll first need to run updates. You can do this by running

```
composer update
composer upgrade
```

Once this command is finished running, you can then begin to install the module. I will provide the current release, but you can navigate to [https://www.drupal.org/project/archivesspace](https://www.drupal.org/project/archivesspace) if you'd like to get the link to the latest version. Download the module by entering

```
wget https://ftp.drupal.org/files/projects/archivesspace-8.x-1.2.tar.gz
tar -xvf archivesspace-8.x-1.2.tar.gz -C ./web/modules/contrib/
```

Now that the project is downloaded, you'll need to modify a file in order to allow it to install. I have setup the following command to automatically modify this file, and it may not work correctly if you use a newer version of the module that I had provided.

```sed -i 's#"islandora/controlled_access_terms": "^1.1 || ^2"#"islandora/controlled_access_terms": "^1.1 || ^2 || 8.x-dev"#' web/modules/contrib/archivesspace/composer.json```

We can now move onto installing dependencies. You can do this by simply running the composer require command as follows

```
composer require drupal/address
composer require drupal/title_length
```

Then enable the archivesspace module with islandora using the drush command as follows

```drush en -y archivesspace_islandora_defaults```

Next you'll need to navigate back to the web interface of islandora. Using the browser you logged into before, open [https://islandora.traefik.me/admin/archivesspace/config](https://islandora.traefik.me/admin/archivesspace/config). At this point, you'll need to find the ip address to the archivesspace instance you're looking to integrate with. Once you've found the archivesspace server, enter http://IP.ADDR:8089. This should be the link to the archivesspace API for your archivesspace server. If you have custom configuration, or you have a firewall blocking connections to the api, please fix this and enter the correct information. You'll also need to enter the username and password to the user which can access the archivesspace repository. Once you've entered the correct information, you can then click Save configuration. Go to [https://islandora.traefik.me/admin/archivesspace/batch-update](https://islandora.traefik.me/admin/archivesspace/batch-update) in order test the information you entered. If you get an error, than you need to go back and check your configuration, but if it shows up find, you'll proceed with the migration step.

#### Migrate from archivesspace
You'll first want to open up the container terminal again. If you lost the terminal you can jump back to [Connecting to the container](#connecting-to-the-container). Once in the terminal, to run migrations, you can then run drush as follows

***Disclaimer: You may want to grab a coffee while you wait***

```
drush mim --all
```

Now that all of the migrations have run, you can see all of the items within your archivesspace in the islandora content. If you'd like to enable images within these archivesspace entries, you're not done yet. Continue on to [Modify Content types](#modify-content-types) in order to get this done.

## Add Glossary to the site

Navigate to [https://islandora.traefik.me/admin/structure/views](https://islandora.traefik.me/admin/structure/views) in order to enable the glossary. You can then scroll to the bottom and press Enable next to Gloassary. This will allow for users to see the glossary in the Navbar, and to open /glossary in order to explore the archive from islandora.

## Modify Content types

The archivesspace islandora module creates a few new content types within islandora. What this does is allows us to customize how some content is formatted, what it can store, etc. We'll modify the archival object in order to allow us to upload images to the content migrated by the archivesspace module. First you'll make sure you open the content types menu at [https://islandora.traefik.me/admin/structure/types](https://islandora.traefik.me/admin/structure/types). You'll want to click Manage Fields for the Archival Object. Here you can see a list of fields that can be added to that content type. Click Add field. Select Reference->Image from the Add a new field dropdown list. Select Unlimited from Allowed number of values dropdown list. You'll want to modify Allowed file extensions to the file types that you'd like to allow to be uploaded using comma or space to separate each. Save field settings. You can then set the label to media and save settings. From here you can begin to upload images to Islandora.

## Modify Homepage
After setting up islandora, you may want to point the homepage to a specific part of the website. For Example, you may want to point it at the content search or the archive page. You can do this by first determining the path to the page (Ex: the path to www.google.com/applesauce would be /applesauce). You will then need to navigate to [https://islandora.traefik.me/admin/config/system/site-information](https://islandora.traefik.me/admin/config/system/site-information]) which will lead you to the basic site settings page. You can then modify the path under Front Page -> Default front page. Save configuration when done.

Source: https://it.umn.edu/services-technologies/how-tos/drupal-7-set-page-your-sites-homepage

## Daily Processes when using Islandora

### Upload Images to Content
In order to upload an image to some content on islandora, you'll first need to open the content you want to add the image to. Navigate to [https://islandora.traefik.me/admin/content](https://islandora.traefik.me/admin/content), and search for the content you'd like to add images to. You can also filter by content type, and change that to Archival Object. Open the content, and navigate to the edit tab of it. Scroll down until you find the field you have for uploading files, I defaulted to Image, and click Choose Files to upload your media. Make sure to add Alternative text before Saving. Finally click save to add the image to the content.
