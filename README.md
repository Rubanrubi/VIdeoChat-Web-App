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

Create the Project Controller:

In the app/Http/Controllers directory, create a new file named VideoRoomsController.php.  Inside this new file, create a VideoRoomsController class with the dependencies we’ll use:

<?php namespace AppHttpControllers;

   use IlluminateHttpRequest;
   use TwilioRestClient;
   use TwilioJwtAccessToken;
   use TwilioJwtGrantsVideoGrant;

class VideoRoomsController extends Controller
{
}

Inside the new class add four protected variables to hold your Twilio account and API information, then define these variables in your __construct() method.

protected $sid;
protected $token;
protected $key;
protected $secret;

public function __construct()
{
   $this->sid = config('services.twilio.sid');
   $this->token = config('services.twilio.token');
   $this->key = config('services.twilio.key');
   $this->secret = config('services.twilio.secret');
}

The rest of the controller class will be made up of three methods to match our three project routes: index, createRoom, and joinRoom.

Our index method will request a list of video rooms associated with our account credentials, then return that list to an index view which we’ll create later.

public function index()
{
   $rooms = [];
   try {
       $client = new Client($this->sid, $this->token);
       $allRooms = $client->video->rooms->read([]);

        $rooms = array_map(function($room) {
           return $room->uniqueName;
        }, $allRooms);

   } catch (Exception $e) {
       echo "Error: " . $e->getMessage();
   }
   return view('index', ['rooms' => $rooms]);
}

The createRoom method will take a room name from the posted form request.  We’ll check to see if a room by that name already exists.  If there is no room by that name, we’ll create one.  Whether the room is new or not, we then send the user to joinRoom with that room name specified.

public function createRoom(Request $request)
{
   $client = new Client($this->sid, $this->token);

   $exists = $client->video->rooms->read([ 'uniqueName' => $request->roomName]);

   if (empty($exists)) {
       $client->video->rooms->create([
           'uniqueName' => $request->roomName,
           'type' => 'group',
           'recordParticipantsOnConnect' => false
       ]);

       \Log::debug("created new room: ".$request->roomName);
   }

   return redirect()->action('VideoRoomsController@joinRoom', [
       'roomName' => $request->roomName
   ]);
}

The last method we need to create to complete our controller, joinRoom, creates an access token for the user and the specified room, and then sends that information to the room view we will create later. The room view will use the access token to connect the user with the active Video group.

public function joinRoom($roomName)
{
   // A unique identifier for this user
   $identity = Auth::user()->name;

   \Log::debug("joined with identity: $identity");
   $token = new AccessToken($this->sid, $this->key, $this->secret, 3600, $identity);

   $videoGrant = new VideoGrant();
   $videoGrant->setRoom($roomName);

   $token->addGrant($videoGrant);

   return view('room', [ 'accessToken' => $token->toJWT(), 'roomName' => $roomName ]);
}

Create the Views:

We only need two views for this project. We can use Laravel’s default resource/views/welcome.blade.php as a template. Rename welcome.blade.php to index.blade.php. Inside the content class div, find the div with the title class and replace its contents with “Video Chat Room”. Then replace the rest of the content div with a simple form for entering a new video group name, followed by a list of links to existing rooms:

<div class="content">
   <div class="title m-b-md">
       Video Chat Rooms
   </div>

   {!! Form::open(['url' => 'room/create']) !!}
       {!! Form::label('roomName', 'Create or Join a Video Chat Room') !!}
       {!! Form::text('roomName') !!}
       {!! Form::submit('Go') !!}
   {!! Form::close() !!}

   @if($rooms)
   @foreach ($rooms as $room)
       <a href="{{ url('/room/join/'.$room) }}">{{ $room }}</a>
   @endforeach
   @endif
</div>

Make a copy of index.blade.php named room.blade.php. Replace the form and room list in the content div with an empty div with an id of media-div.

<div class="content">
       <div class="title m-b-md">
           Video Chat Rooms
       </div>

       <div id="media-div">
       </div>
   </div>

The core of the work we’ll add to our script block is a series of sequential steps: first creating the user’s local video and audio tracks, then connecting to the group room, and finally initiating participant setup and event listeners.

<!— Insert just above the </head> tag —>
<script src="//media.twiliocdn.com/sdk/js/video/v1/twilio-video.min.js"></script>
<script>
    Twilio.Video.createLocalTracks({
       audio: true,
       video: { width: 300 }
    }).then(function(localTracks) {
       return Twilio.Video.connect('{{ $accessToken }}', {
           name: '{{ $roomName }}',
           tracks: localTracks,
           video: { width: 300 }
       });
    }).then(function(room) {
       console.log('Successfully joined a Room: ', room.name);

       room.participants.forEach(participantConnected);

       var previewContainer = document.getElementById(room.localParticipant.sid);
       if (!previewContainer || !previewContainer.querySelector('video')) {
           participantConnected(room.localParticipant);
       }

       room.on('participantConnected', function(participant) {
           console.log("Joining: '"   participant.identity   "'");
           participantConnected(participant);
       });

       room.on('participantDisconnected', function(participant) {
           console.log("Disconnected: '"   participant.identity   "'");
           participantDisconnected(participant);
       });
    });
    // additional functions will be added after this point
</script>

The above steps reference two methods we’ll need to create: participantConnected and participantDisconnected.  The former method is used both to add the audio and video tracks for existing participants to the view and also to add the tracks for new users as they join. The latter method is triggered by participants leaving the group video chat, removing the departing user’s audio and video from the view.

function participantConnected(participant) {
   console.log('Participant "%s" connected', participant.identity);

   const div = document.createElement('div');
   div.id = participant.sid;
   div.setAttribute("style", "float: left; margin: 10px;");
   div.innerHTML = "<div style='clear:both'>" participant.identity "</div>";

   participant.tracks.forEach(function(track) {
       trackAdded(div, track)
   });

   participant.on('trackAdded', function(track) {
       trackAdded(div, track)
   });
   participant.on('trackRemoved', trackRemoved);

   document.getElementById('media-div').appendChild(div);
}

function participantDisconnected(participant) {
   console.log('Participant "%s" disconnected', participant.identity);

   participant.tracks.forEach(trackRemoved);
   document.getElementById(participant.sid).remove();
}

Here you will see references to our final two methods: trackAdded and trackRemoved. In addition to adding video and audio elements, note that we are also using trackAdded to style the video tracks as we add them.

function trackAdded(div, track) {
   div.appendChild(track.attach()); 
   var video = div.getElementsByTagName("video")[0];
   if (video) {
       video.setAttribute("style", "max-width:300px;");
   }
}

function trackRemoved(track) {
   track.detach().forEach( function(element) { element.remove() });
}

IT Makes Success Thank You!!! Do Follow My Github Account!
