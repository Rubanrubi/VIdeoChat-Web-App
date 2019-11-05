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

Getting Started:

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

Private Account Information:

Okay now let’s replace the placeholder values for our .env variables:

TWILIO_ACCOUNT_SID=your-account-sid
TWILIO_ACCOUNT_TOKEN=your-account-token
TWILIO_API_KEY=your-video-api-key
TWILIO_API_SECRET=your-video-api-secret

For this information to be useable within the Laravel application, the config/services.php file includes the following section:
 
return [
  // other array elements exist here

  'twilio' => [
     'sid' => env('TWILIO_ACCOUNT_SID'),
     'token' => env('TWILIO_ACCOUNT_TOKEN'),
     'key' => env('TWILIO_API_KEY'),
     'secret' => env('TWILIO_API_SECRET')
  ]
];

Now, as an example, if we need to access our Twilio account SID inside the application we would use config('services.twilio.sid').

Create Routes
Only three new routes are needed for our project:

*One for handling our default page where users may select an existing video group or name a new one
*A second route for joining an existing Video group
*And a third for creating a new group from a user submitted name.

In routes/web.php let’s remove the route to the welcome view and add the following lines:

Route::get('/', "VideoRoomsController@index");
Route::prefix('room')->middleware('auth')->group(function() {
   Route::get('join/{roomName}', 'VideoRoomsController@joinRoom');
   Route::post('create', 'VideoRoomsController@createRoom');
});

For More Info about code Please Do clone and Check it with your local!

IT Makes Success Thank You!!! Do Follow My Github Account!
