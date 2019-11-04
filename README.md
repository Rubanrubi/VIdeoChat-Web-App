# VIdeoChat-Web-App

To Built a Video Chat App Using Laravel and Twilio.

![Video Chat using Twilo and Laravel](https://twilio-cms-prod.s3.amazonaws.com/images/laravel.width-808.png)

While words alone are powerful, the inflections of people’s voices, the gestures and expressions we unconsciously flow through during conversation all contain a wealth of information often lost to us in our technology driven communications.

Using Twilio’s Video API you can now add the richness of face to face interactions to any web project.

Here we’ll look at how to create a Laravel web application that gives users the ability to join existing video groups or create their own.

Assumptions:

This walkthrough assumes you have a PHP development environment setup with git installed, a global install of composer, and which enables you to access your project through a browser using localhost or virtual hosts. You will also want to gather some information beforehand:

Your Twilio Account SID & Token
Your Twilio Video API Key and Secret
An empty database named “video”

Getting Started
We’re going to set up video groups on top of a Laravel 5.5 installation with basic auth and a couple of dependencies: twilio/sdk and laravelcollective/html.  We’ll be using the master branch of this repo as a starting place. If you would like to look at the completed project, checkout that repo’s complete branch.

From the command line, in the directory where you keep your project folders, run:

git clone https://github.com/Rubanrubi/VIdeoChat-Web-App VideoGroupChat

Move into our new project directory.
cd ~/VideoGroupChat

Next we’ll use composer to install dependencies. Then we’ll make sure our storage and cache directories are writeable, create our .env from the example, and generate our project key using the following commands:
composer install
chmod -R 775 storage/
chmod -R 775 bootstrap/cache
cp .env.example .env
php artisan key:generate

Open your new .env file.  Update the DB_USERNAME and DB_PASSWORD values if your setup doesn’t match the defaults. Then to generate our database structure we run:
php artisan migrate

You’ll also notice we are connecting to the “video” mysql database. If you run into an error that this database does not exist, you can create it and then run the migration command again:

mysql -u username
create database video;
# EXIT mysql
php artisan migrate
