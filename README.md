Project Diagram 



Creating the Network

First create a network using the command 

docker network create goals-net 

Running the Mongodb Container and the Back End Container in the goals-net Network

then use the following string in the node API app in the mongoose.connect function 
mongoose.connect(
  `mongodb://mongodb:27017/course-goals`, //give container name if mongo container in the network
  {
    useNewUrlParser: true,
    useUnifiedTopology: true,
  },
  (err) => {
    if (err) {
      console.error('FAILED TO CONNECT TO MONGODB');
      console.error(err);
    } else {
      console.log('CONNECTED TO MONGODB');
      app.listen(5000);
    }
  }
);

Use the following command to run the node API app in the container named backend using the 

Command: docker run -d -p 5000:5000 --network goals-net --name backend backend:node

Use the following command to run the container named mongodb containing the mongodb database using the 

Command: docker run -d -p 27017:27017 --network goals-net --name mongodb mongo
Now the mongodb and the backend container are able to talk to each other on the network goals-net 

Dockerizing the Front End App

Now lets dockerize the front end React App
First of all build the image of the react app using the command 

Command: docker build -t front:react .
After making the container, run the image using the command

Command: docker run -d -p 3000:3000 -it --name front front:react
Now all of the 3 containers are able to talk to each other but they are communicating locally on the local machine by exposing their ports

Persisting the Data

Problem: 
When we stop and remove the mongodb container the goals that we have entered are all lost this is because when data is stored in the mongodb container when it is removed the data is removed along with it 

Now in order to make the data persist in the mongodb we will create a volume and we will use a named  volume when we run the mongodb so that when we stop and remove the mongodb container our data persists

Command: docker run -d -v data:/data/db --network goals-net --name mongodb mongo

now we see that our data persists 





Access should be Limited

now another requirement for our mongodb database is security and preventing access to our database 

and for that mongodb supports two environment variables 
MONGO_INITDB_ROOT_USERNAME, MONGO_INITDB_ROOT_PASSWORD

and when we use the above variables then the database inside of the mongodb database will be created such that it can only be accessed using the username and the password 

In order to add that extra layer of security we have to change the mongodb string again in the mongoose.connect function to 

`mongodb://mashhood:123@mongodb:27017/course-goals?authSource=admin` in the mongoose.connect function 

mongoose.connect(
  `mongodb://mashhood:123@mongodb:27017/course-goals?authSource=admin`, //give container name if mongo container in the network
  {
    useNewUrlParser: true,
    useUnifiedTopology: true,
  },
  (err) => {
    if (err) {
      console.error('FAILED TO CONNECT TO MONGODB');
      console.error(err);
    } else {
      console.log('CONNECTED TO MONGODB');
      app.listen(5000);
    }
  }
);


now since the backend code has changed so we need to rebuild the backend image 

now we have to stop and remove the mongodb container and run the container again using the following command utilizing the environment variables 

Command: docker run -d -v data:/db/data --network goals-net -e MONGO_INITDB_ROOT_USERNAME=mashhood -e MONGO_INITDB_ROOT_PASSWORD=123 --name mongodb mongo

re run the container using the command 
Command: docker run -d -p 5000:5000 --network goals-net --name backend backend:node
Back end

Data Persist

Making the logs in the app persist using the following flag and option 

-v logs/app/logs
we wanna bind everything in the app folder to our local hosting directory simply to ensure that when we change something in our source code is is reflected inside of the container now for such a bind mount we need the complete path of the app.js file 	to bind as a name to the container internal path 
-v  “/home/mash-hood/LOCAL DRIVE D/DEVOPS LEARNING/[GigaCourse.Com] Udemy - Docker & Kubernetes The Practical Guide/lecture 74/multi-container-startup/backend”:/app

now we are going to use an anonymous volume for the node_modules so that the node_modules in the container are not overwritten by the node_modules in the application folder 

-v /app/node_modules

 Live Source Code Changes

for making live source code changes to be reflected immediately without having to stop and restart the container again and again we are going to use a dependency called  nodemon add the following in the package.json file 

“devDependencies”:{
“nodemom”:”^2.0.4”
}

and write the following in the script 

“start”: “nodemon app.js”

and make the following changes in the docker file 

instead of CMD [“node”,”app.js”]

write CMD {“npm”,”start”}
now your app.js and docker file have changed and now you have to rebuild the image

use the above command to keep running the mongodb container while use the following command to run the backend container 

Command:  docker run -d -p 5000:5000 --network goals-net -v logs:/app/logs -v "/home/mash-hood/LOCAL DRIVE D/DEVOPS LEARNING/[GigaCourse.Com] Udemy - Docker & Kubernetes The Practical Guide/lecture 75/multi-container-startup/backend:/app" -v /app/node_modules --name backend backend:node

in the above command replace this 

-v "/home/mash-hood/LOCAL DRIVE D/DEVOPS LEARNING/[GigaCourse.Com] Udemy - Docker & Kubernetes The Practical Guide/lecture 75/multi-container-startup/backend:/app"

replace the path "/home/mash-hood/LOCAL DRIVE D/DEVOPS LEARNING/[GigaCourse.Com] Udemy - Docker & Kubernetes The Practical Guide/lecture 75/multi-container-startup/backend

with the path where your app.js file is 

before making the changes to the app.js file 


Terminal:

after changing the app,js file


Terminal



now as you recall we have hard coded the username and password string in the connection string 

here in the app.js file 

mongoose.connect(
  `mongodb://mashhood:123@mongodb:27017/course-goals?authSource=admin`, //give container name if mongo container in the network
  {
    useNewUrlParser: true,
    useUnifiedTopology: true,
  },
  (err) => {
    if (err) {
      console.error('FAILED TO CONNECT TO MONGODB');
      console.error(err);
    } else {
      console.log('CONNECTED TO MONGODB!!');
      app.listen(5000);
    }
  }
);

so in order to inject dynamic values and authorize them in the above string we will use environment vairabels in the docker file and in the string above 

mongoose.connect(
  `mongodb://${process.env.MONGODB_USERNAME}:${process.env.MONGODB_PASSWORD}@mongodb:27017/course-goals?authSource=admin`, //give container name if mongo container in the network
  {
    useNewUrlParser: true,
    useUnifiedTopology: true,
  },
  (err) => {
    if (err) {
      console.error('FAILED TO CONNECT TO MONGODB');
      console.error(err);
    } else {
      console.log('CONNECTED TO MONGODB');
      app.listen(80);
    }
  }
);

and modify the dockerfile to 














stop the backend container, rebuild the image and then run it  again since we have changed the docker file 
now we can use the following command to run the container again 

Command: docker run -d -p 5000:5000 --network goals-net -v logs:/app/logs -v "/home/mash-hood/LOCAL DRIVE D/DEVOPS LEARNING/[GigaCourse.Com] Udemy - Docker & Kubernetes The Practical Guide/lecture 75/multi-container-startup/backend:/app" -v /app/node_modules -e MONGODB_USERNAME=mashhood -e MONGODB_PASSWORD=123 --name backend backend:node

remember the username and the password that we have used while running the mongodb container is 

USERNAME: mashhood
PASSWORD: 123

not root and secret 
Note: use the username and password that you have used while running the mongodb container 

now everything works fine 


Front End Live Source Code Changes

In order to do this we will use a bind mount not on the complete front end folder but only on the /app/src folder in the app 

Command: docker run -d -p 3000:3000 -it -v "/home/mash-hood/LOCAL DRIVE D/DEVOPS LEARNING/[GigaCourse.Com] Udemy - Docker & Kubernetes The Practical Guide/lecture 75/multi-container-startup/frontend/src:/app/src" --name front front:react

we can use the .dockerignore file to ignore the node_modules, dockerfile and the .git folder when we copy our files into the container to make the image building process faster

.dockerignore file



live source code changes

/src/App.js file 


Browser

