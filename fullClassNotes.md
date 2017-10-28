# Mongo Class Shortcut Notes

>> Note: We are working with Linux terminals, Ctrl-C/Ctrl-V won't work
>> 		Use Ctrl-INS and Shift-INS to cut and paste, instead

## Setup

1. Use Docker to set up a Mongo Container -
```
http://play-with-docker.com 

     Select not a robot
     +Add New Instance
```		
2. In running instance (black section, bottom right), get the docker image -
```
docker pull wgrasser/mongoflights
```
	
3. Find the image we are working with
```
docker images
```

4. Copy Image Id from response (such as 92a9a3c76f29)

ctrl-c won't work, use ctrl-ins or right-click-copy

4a. Start the docker image and the bash shell
```
docker run -it \<imageId\> /bin/bash
```

the prompt will change to root@###\>
		
5. In the docker image, start the Mongo daemon in the background
```
mongod -f /etc/mongod.conf &
```
this runs the Mongo daemon in the background

6. Start the Mongo Shell		
```
mongo
```		

## Part 1 - Working with the Mongo Shell
In the previous step, we started the Mongo Shell

1. List the available databases -
```
show dbs;
```

2. When we use a new database, it isn't created until we insert a collection/document -
```
use courses;

show dbs;
```
Notice courses isn't there, because Mongo won't create a database until there's data
		
3. Now insert an empty document into the collection schedules
```
db.schedules.insert({});

show dbs;
```

4. Insert document with some real data
```
db.schedules.insert({"academicTerm" : "2013/2014",
	"events" : [
	 {
	      "classPeriod" : {
	           "start" : "18/09/2013 17:30",
	           "end" : "18/09/2013 19:00"
	      },
	      "location" : [
	           {
	                "type" : "ROOM",
	                "name" : "F4",
	                "id" : "2448131363674"
	           }
	      ],
	      "title" : "Gestão : Problemas",
	      "course" : {
	           "acronym" : "Ges5",
	           "name" : "Gestão",
	           "academicTerm" : "1 Semestre 2013/2014",
	           "id" : "1610612925989"
	      }
	 },
	 {
	      "classPeriod" : {
	           "start" : "28/10/2013 14:30",
	           "end" : "28/10/2013 15:30"
	      },
	      "location" : {
	           "type" : "ROOM",
	           "name" : "QA02.4",
	           "id" : "2448131363664",
	           "topLevelSpace" : {
	                "type" : "CAMPUS",
	                "id" : "2448131360897",
	                "name" : "Alameda"
	           }
	      }
	 }
	]
	});
 ```
 
5. See what is in the schedules collection
```
db.schedules.find({});
```
6. That works, but it's hard to look at
```
db.schedules.find({}).pretty();
```

## Part 2 - Working with a larger data set

>> Background: <BR/>
Sample Data from United States Department of Transportation On-Time Performance<BR/>
Imported to Mongo using MongoImport<BR/>
&nbsp;&nbsp;&nbsp;&nbsp;Mongoimport -d flights -c flightstats --type csv --file 642648472_T_ONTIME.csv --headerline

The docker image already the needed flight data loaded. If you are using your own docker or mongo instance, visit the [United States Department of Transportation On-Time Performance](https://www.transtats.bts.gov/DL_SelectFields.asp?Table_ID=236&DB_Short_Name=On-Time) and grab some data to work with

1. Use the existing database -
```
use flights
```
About find: find() takes two parameters, query and projection<BR/>
&nbsp;&nbsp;&nbsp;&nbsp;query is like the where clause<BR/>
&nbsp;&nbsp;&nbsp;&nbsp;projection is the fields to be returned<BR/>
We start with both empty to get everything ( find({},{}) = find({}) )<BR/>

2. Grab the first record (find all, limit to one) and make it look pretty
```
db.flightstats.find({}).limit(1).pretty();
```

3. We don't need all of those fields, just limit to the few care about
```
db.flightstats.find(
	    {},
	    {
	        "FL_DATE" : 1, "FL_NUM" : 1, "ORIGIN_CITY_NAME" : 1, 
	        "DEST_CITY_NAME" : 1, "DEST_STATE_ABR" : 1, "DEST_STATE_NM" : 1, 
	        "DEP_TIME" : 1, "ARR_TIME" : 1, "CANCELLED" : 1, 
	        "DIVERTED" : 1, "ACTUAL_ELAPSED_TIME" : 1, "AIR_TIME" : 1
	    }).limit(1).pretty();
```

4. Drop the _id by setting it to 0
```
db.flightstats.find(
	    {},
	    {_id : 0,
	        "FL_DATE" : 1, "FL_NUM" : 1, "ORIGIN_CITY_NAME" : 1, 
	        "DEST_CITY_NAME" : 1, "DEST_STATE_ABR" : 1, "DEST_STATE_NM" : 1, 
	        "DEP_TIME" : 1, "ARR_TIME" : 1, "CANCELLED" : 1, 
	        "DIVERTED" : 1, "ACTUAL_ELAPSED_TIME" : 1, "AIR_TIME" : 1
	    }).limit(10).pretty();
```

5. How many records in June?   
```
db.flightstats.find(
	    {
	        "MONTH": {$eq: 6}
	    }).count();
```

6. How many flights are recorded on the 15th of the month?
```
db.flightstats.find(
	    { 
	        $and: [
	            {"MONTH": {$eq: 6}}, 
	            {"DAY_OF_MONTH": 
	            {$eq: 15}}
	        ]
	    }).count();
```
Note: here we are using [ and ] to manage an array of element

7. How many were going to Ohio?
```
db.flightstats.find(
	    { 
	        $and: [
	            {"DEST_STATE_ABR": {$eq: "OH"}},
	            {"MONTH": {$eq: 6}}, 
	            {"DAY_OF_MONTH": {$eq: 15}}
	        ]}
	    ).count();
```

8. How many of those were early?
```
db.flightstats.find({ 
	    $and: [
	        {"DEST_STATE_ABR": {$eq: "OH"}},
	        {"MONTH": {$eq: 6}}, 
	        {"DAY_OF_MONTH": {$eq: 15}},
	        {"ARR_DELAY": {$lt: 0}}
	    ]}
	).count();
```
9. Now let's look at those records
```
db.flightstats.find(
	    { 
	        $and: [
	            {"DEST_STATE_ABR": {$eq: "OH"}},
	            {"MONTH": {$eq: 6}}, 
	            {"DAY_OF_MONTH": {$eq: 15}},
	            {"ARR_DELAY": {$lt: 0}}
	        ]}
	    ).sort({"ARR_DELAY": 1}).forEach( function(theFlight)  { 
	            print( "Flight " + theFlight.FL_NUM + ", on " + theFlight.FL_DATE + " from " +
	                theFlight.ORIGIN_CITY_NAME + " to " + theFlight.DEST_CITY_NAME + " was " +
	                (theFlight.ARR_DELAY * -1) + " minutes early"); 
	    } );
```

## Part 3 - Aggregation Pipeline


1. Be sure we are in the flights database
```
use flights
```

2. Start with one pipeline operator
```
db.flightstats.aggregate([ {$match : {"DAY_OF_MONTH":15}}])
```
Note: it in the shell allows you to iterate through the returned records
	
3. Add a project to get the data we are interested in
```
db.flightstats.aggregate([{$match : {"DAY_OF_MONTH":15}},
	    { $project : {"_id" : 0,"FL_DATE" : 1, "FL_NUM" : 1, "ORIGIN_CITY_NAME" : 1, 
	              "DEST_CITY_NAME" : 1, "DEST_STATE_ABR" : 1, "DEST_STATE_NM" : 1, 
	              "DEP_TIME" : 1, "ARR_TIME" : 1, "CANCELLED" : 1, 
	               "DIVERTED" : 1, "ACTUAL_ELAPSED_TIME" : 1, "AIR_TIME" : 1}}]);
```

4. Update the match to grab the data we care about
``` 
db.flightstats.aggregate([ 
	    {$match : { $and: [
	                       {"DEST_STATE_ABR": {$eq: "OH"}},
	                       {"MONTH": {$eq: 6}}, 
	                       {"DAY_OF_MONTH": {$eq: 15}},
	                       {"ARR_DELAY": {$lt: 0}} ]}},
	    { $project : {"_id" : 0,"FL_DATE" : 1, "FL_NUM" : 1, "ORIGIN_CITY_NAME" : 1, 
	              "DEST_CITY_NAME" : 1, "DEST_STATE_ABR" : 1, "DEST_STATE_NM" : 1, 
	              "DEP_TIME" : 1, "ARR_TIME" : 1, "CANCELLED" : 1, 
	               "DIVERTED" : 1, "ACTUAL_ELAPSED_TIME" : 1, "AIR_TIME" : 1}}
	]);
```

5. Add our sort to order the data
```
db.flightstats.aggregate([ 
	    {$match : { $and: [
	                       {"DEST_STATE_ABR": {$eq: "OH"}},
	                       {"MONTH": {$eq: 6}}, 
	                       {"DAY_OF_MONTH": {$eq: 15}},
	                       {"ARR_DELAY": {$lt: 0}} ]}},
	    { $project : {"_id" : 0,"FL_DATE" : 1, "FL_NUM" : 1, "ORIGIN_CITY_NAME" : 1, 
	              "DEST_CITY_NAME" : 1, "DEST_STATE_ABR" : 1, "DEST_STATE_NM" : 1, 
	              "DEP_TIME" : 1, "ARR_TIME" : 1, "CANCELLED" : 1, 
	               "DIVERTED" : 1, "ACTUAL_ELAPSED_TIME" : 1, "AIR_TIME" : 1}},
	             { $sort : {"ARR_DELAY": 1}}
	]);
```

6. Which arrival city had the most early flights on June 15th 2017?
```
db.flightstats.aggregate([ 
	    {$match : { $and: [
	                       {"DEST_STATE_ABR": {$eq: "OH"}},
	                       {"MONTH": {$eq: 6}}, 
	                       {"DAY_OF_MONTH": {$eq: 15}},
	                       {"ARR_DELAY": {$lt: 0}} ]}},
	             { $group : { "_id" : "$DEST_CITY_NAME",
	                         flightCount : { $sum : 1},
	                         minutesSaved: { $sum : "$ARR_DELAY"}}},
	             { $sort : {minutesSaved : 1}}
	]);
```

7. How do flights to Ohio really look on June 15th?
```
db.flightstats.aggregate([ 
	    {$match : { $and: [
	                       {"DEST_STATE_ABR": {$eq: "OH"}},
	                       {"MONTH": {$eq: 6}}, 
	                       {"DAY_OF_MONTH": {$eq: 15}}]}},
	    { $project : {
	                 "_id" : 0, "DEST_CITY_NAME" : 1,
	                 early   : {$cond: [{$lt: ["$ARR_DELAY", 0]}, "$ARR_DELAY", 0]},
	                 late    : {$cond: [{$gt: ["$ARR_DELAY", 0]}, "$ARR_DELAY", 0]},
	                 earlyCt : {$cond: [{$lt: ["$ARR_DELAY", 0]}, 1, 0]},
	                 lateCt  : {$cond: [{$gt: ["$ARR_DELAY", 0]}, 1, 0]},
	                 ontimeCt: {$cond: [{$eq: ["$ARR_DELAY", 0]}, 1, 0]}                  
	    }},
	    { $group : { 
	                "_id" : "$DEST_CITY_NAME",
	                flightCount : { $sum : 1},
	                numEarly : { $sum : "$earlyCt"},
	                numLate  : { $sum : "$lateCt"},
	                minutesSaved : { $sum : "$early"},
	                minutesLost : { $sum : "$late"}
	                }},                
	    { $sort : {flightCount : -1}}
	]).pretty();
```

## Part 4 - Indexes

>>The create index is <BR/>
db.collection.createIndex( \<key and index type specification\>, \<options\> )

1. Be sure we are in the flights database
```
use flights;
```

2. Start with one of our queries
```
db.flightstats.find(
	    { 
	        $and: [
	            {"DEST_STATE_ABR": {$eq: "OH"}},
	            {"MONTH": {$eq: 6}}, 
	            {"DAY_OF_MONTH": {$eq: 15}},
	            {"ARR_DELAY": {$lt: 0}}
	        ]}
	    ).explain();
```

that tells us what was run and that a full column scan was run, but we want real stats -
	
3. Add in the stats
```
db.flightstats.find(
	    { 
	        $and: [
	            {"DEST_STATE_ABR": {$eq: "OH"}},
	            {"MONTH": {$eq: 6}}, 
	            {"DAY_OF_MONTH": {$eq: 15}},
	            {"ARR_DELAY": {$lt: 0}}
	        ]}
	    ).explain("executionStats"); 
```
check out these fields, especially: <BR/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"nReturned" : 95,<BR/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"executionTimeMillis" : 896,<BR/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"totalKeysExamined" : 0,<BR/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"totalDocsExamined" : 494266,<BR/>
	
4. Create an index that fits our needs
```
db.flightstats.createIndex({"DEST_STATE_ABR": 1, "MONTH": 1, "DAY_OF_MONTH": 1, "ARR_DELAY": 1});
```

5. Show the indexes
```
db.flightstats.getIndexes();
```

6. See how that impacts our query
```
db.flightstats.find(
	    { 
	        $and: [
	            {"DEST_STATE_ABR": {$eq: "OH"}},
	            {"MONTH": {$eq: 6}}, 
	            {"DAY_OF_MONTH": {$eq: 15}},
	            {"ARR_DELAY": {$lt: 0}}
	        ]}
	    ).explain("executionStats");
```
now look at those stats again: <BR/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"nReturned" : 95, <BR/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"executionTimeMillis" : 1, <BR/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"totalKeysExamined" : 95, <BR/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"totalDocsExamined" : 95,

7. Remove that index
```
db.flightstats.dropIndex({"DEST_STATE_ABR": 1, "MONTH": 1, "DAY_OF_MONTH": 1, "ARR_DELAY": 1});
```
8. Add some others
```
db.flightstats.createIndex({"DEST_STATE_ABR": 1})
db.flightstats.createIndex({"MONTH": 1, "DAY_OF_MONTH": 1});
db.flightstats.createIndex({"ARR_DELAY": 1});

db.flightstats.getIndexes();
```

9. See what is different
```
db.flightstats.find(
	    { 
	        $and: [
	            {"DEST_STATE_ABR": {$eq: "OH"}},
	            {"MONTH": {$eq: 6}}, 
	            {"DAY_OF_MONTH": {$eq: 15}},
	            {"ARR_DELAY": {$lt: 0}}
	        ]}
	    ).explain("executionStats");
```
	
More interesting this time are these sections: <BR/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"winningPlan" : {
				
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"rejectedPlans" : [				




