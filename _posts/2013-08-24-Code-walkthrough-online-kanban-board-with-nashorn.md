---
layout: post
author: Marcelo Costa
---
This week I present to you a quick Code Walkthrough on my latest invention: The Online Kanban Board

Hello hello my fellow Nashornians, this week I present to you a quick Code Walkthrough on my latest invention: The Online Kanban Board. I’m sure there must be thousands like this out there (better ones I bet) but I decided to code my own when some Support colleagues from work were trying to find out what one of the team members was working on, beyond being the best “tasks bottleneck” detector, this is a nice resource to help everyone to talk about their activities in a good old stand-up meeting, but be aware that meetings can be dangerous. Use the KISS (Keep It Simple, Stupid) principle, get everyone in a room, decide among each other who’s gonna be the “meeting leader”, organize the post-its for each person and go around the room asking these 3 little questions:

- What did you achieve yesterday?
- What will you do today?
- Are you blocked or do you need assistance?

Don’t let the meeting take more than 15 minutes.. if you need to do some code review or brainstorm, schedule other meetings for that, define the agenda and… wait, I’m digressing too much on this subject, let’s see some CODE! 😀

—

Ok, I won’t dive too much on the ‘httpsrv.js‘, it is just a humble upgrade to the one Jim Laskey wrote in his official Nashorn Blog, I just took his code and added some stuff that wanted because command-line I/O wasn’t interesting enough for me to start playing with it, so my version is handling HTTP POST requests and I’m loading a controller.js file to handle non-static-file requests, I will go through the interesting bits later on.

So, let’s start with the HTML and CSS, at the end of this section we should see an interface like this:

kanban

Here’s how the files were structured:

folder_structure

The HTML is quite simple, as you can see, I’m just linking a bunch of stuff that I used to create a good client-side experience ( JQuery-UI for the Draggable and Editable components), then there’s the ‘mykanban.js’ file where I have the code that will be sending the AJAX requests, the post-its will be loaded within the ‘container’ div.

```html
<!DOCTYPE html PUBLIC “-//W3C//DTD HTML 4.01 Transitional//EN” “http://www.w3.org/TR/html4/loose.dtd”&gt;
<html>
<head>
<meta http-equiv=”Content-Type” content=”text/html; charset=UTF-8″>
<title>My Kan Ban board – JQuery + Nashorn + MongoDB</title>
<link rel=”stylesheet” href=”/mykanban/assets/css/jquery-ui.css” />
<script src=”/mykanban/assets/js/jquery-1.9.1.js”></script>
<script src=”/mykanban/assets/js/jquery-ui.js”></script>
<link rel=”stylesheet” href=”/mykanban/assets/css/style.css” />
<link rel=”shortcut icon” href=”http://localhost:8080/mykanban/favicon.ico&#8221; />
<script src=”/mykanban/assets/js/jquery.jeditable.js”></script>
<script src=”/mykanban/assets/js/mykanban.js”></script>
</head>
<body>
<div id=”header”>
<div id=”menu”>
<input id=”addPostIt” type=”image” src=”/mykanban/assets/img/add-icon.png” name=”addPostIt” width=”30″ height=”30″>
</div>
</div>
<div id=”overlay” visible=”false”></div>
<div id=”container”>
<!– post its here –>
</div>
</body>
</html>
</body>
```

The CSS is also simple, I’ve used an old trick to centralize the container div on the page, you can read about it on Maujor’s website (Brazilian guy that is known as the CSS master!):

```css
#container {
position: fixed;
top: 50%;
left: 50%;
margin-top: -300px;
margin-left: -500px;
width: 1000px;
height: 600px;
background: #fdffe5 url(“../img/background.jpg”);
}
```

The client-side Javascript starts by loading all the post-its that are stored in the MongoDB database, it sends a GET AJAX request to the controller which loads the MongoDAO.js and call the ‘readAll()’ function, once the json data is retrieved, it takes the data and call the ‘addpostit()’ function so the draggable div elements can be created, each one with their respective id, task String and position.


```js
$(function() {
    //load post its
    $.getJSON(‘/mykanban/controller.jjsp?action=read’, function(data) {
        var items = [];
        $.each(data, function(key, val) {
            if(data.postits.length>0){
                for(var i=0;i<data.postits.length;i++) {
                   //console.log(“ID: ” + data.postits[i]._id);
                   //console.log(“TASK: ” + data.postits[i].task);
                   //console.log(“POSX: ” + data.postits[i].posX);
                   //console.log(“POSY: ” + data.postits[i].posY)
                  var postitid = data.postits[i]._id;
                  var task = data.postits[i].task;
                  var posx = data.postits[i].posX;
                  var posy = data.postits[i].posY;
                  addpostit(postitid,task,posx,posy);
           }
     }
});
```

It produces an output similar to this one:

```json
{
    “postits”: [
        {
            “_id”: “3”,
            “task”: “Study Nashorn! :D”,
            “posX”: 7,
            “posY”: 86
        },
        {
            “_id”: “6”,
            “task”: “Report weird bug”,
            “posX”: 764,
            “posY”: 80
        }
    ]
}
```

Here’s the controller.js, it handles the data that comes from the client-side and process the Mongo-related actions:

```js
load(“./mykanban/dao/mongoDAO.js”);
function Controller() {
    this.readData = function() {
        return “{ \”postits\” : [” + mongoDAO.readAll() + “]}”;
    }
    this.deleteData = function(params) {
print(“to be deleted: ” + params);
try {
    mongoDAO.delete(params);
}catch(e){
    print(‘Error while deleting the object from Mongo: ‘ + e.printStackTrace());
}
return generateResponse(mongoDAO.readAll());
    }
    this.processData = function(params) {
print(params);
try {
    mongoDAO.create(params);
}catch(e){
    print(‘Error while saving the object into Mongo: ‘ + e);
}
return generateResponse(mongoDAO.readAll());
    }
    function generateResponse(data) {
        var HTML = ” “;
        return HTML;
    }
}
```

It would be cool to come up with some dependency injection mechanism here.. but let’s leave that for later. The DB Persistence layer is comprised of two files ‘MongoDAO.js’ and ‘MongoConnector.js’, the first one loads the second because the connector contains all the “imports” (MongoDB driver) and, now here comes the coolest part, the ‘mongoConnector’ function, which creates a singleton in Javascript through a closure:

```js
var mongodb = Packages.com.mongodb;
var MongoClient = mongodb.MongoClient;
var MongoException = mongodb.MongoException;
var WriteConcern = mongodb.WriteConcern;
var DB = mongodb.DB;
var DBCollection = mongodb.DBCollection;
var BasicDBObject = mongodb.BasicDBObject;
var DBObject = mongodb.DBObject;
var DBCursor = mongodb.DBCursor;
var ServerAddress = mongodb.ServerAddress;
var JSON = mongodb.util.JSON;
var Arrays = java.util.Arrays;
var mongoConnector = (function() {
    //Singleton
    var mongoConnector;
    function init() {
        return {
            getDB : function() {
            var mongo = new MongoClient(“localhost”);
            var db = mongo.getDB(“test”);
            return db;
           }
       }
    }
    return {
        //Get the singleton instance or create a new one
        getInstance : function() {
            if(!mongoConnector) {
                mongoConnector = init();
            }
            return mongoConnector;
        }
    }
    return mongoClient;
})();
```

For those of you who don’t know what a closure is (I won’t even ask about Singleton, just google “Design Patterns” to learn about it), I will try to explain it here (I want to highlight this concept because, to be honest, even though it might seem silly to many programmers, it took me a while to understand it), anyone can memorize “It is a function that returns an inner function that stores the variables defined in the outer function” but comprehending is a whole different story.

In my case here, I didn’t want to create an instance of my mongoConnector for every connection (hence the Singleton), but that’s where Javascript makes everything easier, the ‘getInstance()’ function stores the ‘mongoConnector’ variable that was declared outside its own block of code, notice that the ‘mongoConnector’ function (outer function) is executed only once, it is an IEF (Immediately Executed Function) because it calls itself right after its defintion, i.e., (function() {…})(); , it returns the inner function with the getInstance() function and, at this point, the init() function no longer exists, we won’t have any other expensive operation here, thanks to the closure.

Douglas Crockford’s video entitled ‘Javascript: Good Parts‘ gives a good explanation about it. Highly recommended.

Now our ‘MongoDAO’ can use this single instance for the MongoDB operations:

```js
load(‘./mykanban/dao/mongoConnector.js’);

var mongoDAO = (function() {
//Get connector from singleton
var mongo = mongoConnector.getInstance();

//Select db
var db = mongo.getDB(“test”);

// get list of collections
var collections = db.getCollectionNames();

//Get mongodb collection
var dbCollection = mongo.getDB(“test”).getCollection(“test”);

return {
create: function(someObj) {
//save
dbCollection.save(JSON.parse(someObj));
},
readAll: function() {
var results = [];

var cursorDocJSON = dbCollection.find();

while (cursorDocJSON.hasNext()) {
var cDoc = cursorDocJSON.next();
results.push(cDoc);
}
return results;
},…
```

The greatest thing about this project is that it’s all JSON, end-to-end, even the create/update/delete operations involve the creation of a json formatted ‘postit’ data that gets sent to the controller and processed by MongoDB (JSON.parse()), here’s the function from ‘mykanban.js’ that creates a new post-it:

```js
function updatepostit(element, value) {
var draggable = element.parent();

//console.log(“id: ” +draggable.attr(‘id’));
//console.log(“ID: ” +draggable.attr(‘id’).substr(9));
//console.log(“html: ” +draggable.html());

var postit = {
“_id”: draggable.attr(‘id’).substr(9),
“task”: value,
“posX”: draggable.position().left,
“posY”: draggable.position().top,
};

$.ajax({
type: “POST”,
url: “/mykanban/controller.jjsp”,
// The key needs to match your method’s input parameter (case-sensitive).
data: JSON.stringify( postit ),
contentType: “application/json; charset=utf-8”,
dataType: “json”,
success: function(data){alert(data);},
failure: function(errMsg) {
alert(errMsg);
}
});
}
```

That’s it. if you want to try it out just download the code from github, install MongoDB in your machine, start the database server (just run ‘mongod’, you might need to specify where the files will be stored, in this case use the –dbpath parameter, e.g., ‘mongod –dbpath /var/db/data’) and finally (assuming you have the JDK8 or the OpenJDK built in your machine with Nashorn) start the HTTP Server to see your Kanban board implemented with Nashorn, here’s the command:

`$ jjs -cp lib/mongo-2.10.1.jar:. httpsrv.js`

_*Don’t forget to create some shortcuts to your JJS (Nashorn interpreter):_

Mac OS = `alias jjs=’/Library/Java/JavaVirtualMachines/jdk1.8.0.jdk/Contents/Home/jre/bin/jjs’`

Windows  = Define an environment variable called ‘JAVA8_HOME’ and point to your jdk8 folder, then you can invoke jjs by running this command:

`“%JAVA8_HOME%\jre\bin\jjs” -cp lib\mongo-2.10.1.jar;. httpsrv.js`

I hope you’ve enjoyed it, if you are a Javascript expert and identified any atrocities in my code, please PLEASE share your knowledge on the comments session below.

Have a Nashornian weekend, cheers!
