# Week 1 — App Containerization

[12:12 AM] Perfect Elorm Avugla
# Week 1 —Docker and App Containerization
## Physical vs virtual servers.
WIth physical servers, there are a lot of downsides since you have to buy servers and these servers are constant whether in or out of use. In most cases, there is a lot of unused capacity.There are some cases where users try to make the most use of the capacity by deploying other applications to get the most of the unused capacity, but that also comes with downsides since the failure of one of the applications affect the other applications. WIth virtual servers, you are able to utilize unuse storage capacity with the help of running many applications on a hypervisor, with this, the applications are somewhat containerized hence performance or failure of one application is not dependent on the other.
## Containers: Container Architecture.
Hardware -- host OS -- hypervisor -- guest OS -- applications & ibraries ## Technical Requirements of week 1
I downloaded the docker extension in the visual studio code to enable me run docker smoothly.  ### Containerized Backend
1. Create a Dockerfile in the Backend-flask folder.
2. Open the file and paste the following.
```yaml
FROM python:3.10-slim-buster WORKDIR /backend-flask COPY requirements.txt requirements.txt
RUN pip3 install -r requirements.txt COPY . . ENV FLASK_ENV=development EXPOSE ${PORT}
CMD [ "python3", "-m" , "flask", "run", "--host=0.0.0.0", "--port=4567"]
```
3.  cd Backend-flask
4. pip3 install -r requirements.txt
5. docker build -t  backend-flask ./backend-flask
6. docker container run --rm -p 4567:4567 -d backend-flask
7. checked and opened port 4567,ad ```yaml /api/activities/home``` to the url You should get something similar to the image below;
8. (https://user-images.githubusercontent.com/68542385/220753976-3c0680b9-b7e8-4aee-9498-d8d23ad4eeec.PNG) 
## Containerized Frontend
1. Create a dockerfile in the frontend-react-js directory.
2. open and paste the following code;
```yaml
FROM node:16.18 ENV PORT=3000 COPY . /frontend-react-js
WORKDIR /frontend-react-js
RUN npm install
EXPOSE ${PORT}
CMD ["npm", "start"]
```
3. cd frontend-react-js
4. run npm install
5. docker build -t frontend-react-js .
6. docker run -it -dp 3000:3000 frontend-react-js
7. move to port 3000 and open the url to view the frontend. ## Creating Multiple Containers.
1. Create docker-compose.yml in the root folder.
2. open and paste the following;
```yaml
version: "3.8"
services:
  backend-flask:
    environment:
      FRONTEND_URL: "https://3000-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}"
      BACKEND_URL: "https://4567-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}"
    build: ./backend-flask
    ports:
      - "4567:4567"
    volumes:
      - ./backend-flask:/backend-flask
  frontend-react-js:
    environment:
      REACT_APP_BACKEND_URL: "https://4567-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}"
    build: ./frontend-react-js
    ports:
      - "3000:3000"
    volumes:
      - ./frontend-react-js:/frontend-react-js
  dynamodb-local:
    # https://stackoverflow.com/questions/67533058/persist-local-dynamodb-data-in-volumes-lack-permission-unable-to-open-databa
    # We needed to add user:root to get this working.
    user: root
    command: "-jar DynamoDBLocal.jar -sharedDb -dbPath ./data"
    image: "amazon/dynamodb-local:latest"
    container_name: dynamodb-local
    ports:
      - "8000:8000"
    volumes:
      - "./docker/dynamodb:/home/dynamodblocal/data"
    working_dir: /home/dynamodblocal   db:
    image: postgres:13-alpine
    restart: always
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=password
    ports:
      - '5432:5432'
    volumes: 
      - db:/var/lib/postgresql/data # the name flag is a hack to change the default prepend folder
# name when outputting the image names
networks: 
  internal-network:
    driver: bridge
    name: cruddur volumes:
  db:
    driver: local  
```
3. Right click on the docker-compose.yml file and then click compose up in the dropdown. - this does both a docker build and a docker run on the container.
   ### Erros I encounted and how I solved them
 a. failed to solve: rpc error: code = Unknown desc = failed to solve with frontend dockerfile.v0: failed to create LLB definition: dockerfile parse error line 15: unknown instruction: DOCKER  - I had to remove an external command I pasted on line 15 in my backend docker file during my backend containirization.
 b. Had an issue commiting my code after the whole process because there were too many changes and github and gitpod were not in sync. So i resorted to commiting after every few changes and that worked. ### Spending Considerations.
I watched Chirag's Week 1 - Spending Considerations;
1. Explored gitpod billings and hourly calculations.
2. I got to know about github codespaces but i did not set it up.
3. I also got to know about AWS cloud9; how it works with ec2 instances in the background and should be avoided if there is already a running EC2 for other projects in order to avoid high cost.
4. Got to know about CloudTrails default 90-day api request log.

##Adding the notifications tab
1. add a notification section to the open api.
```yaml
/api/activities/notifictions:
    get:
      description: 'Return a feed of activity for all of those i follow'
      tags:
        - activities
      parameters: []
      responses:
        '200':
          description: Returns an array of activities
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/Message'
 ```
2. preview the api to be sure it works fine, should look something like this;


##Homework Challenges
1. Running the dockerfile CMD as an external script
a. Created a script file in the backend flask and named it app_run.sh
b. Added the following to the script file
```yaml
#!/bin/bash
> python3 -m flask run --host=0.0.0.0 --port=4567
```
c. I added the CMD in the dockerfile
```yaml
CMD [ "./run_app.sh"]
```

d. In the terminal, backend-flask, i run the following;
```yaml
vi run_app.sh
chmod +x ./run_app.sh
docker build -t  backend-flask:external .
docker run backend-flask:external
```
and it gave the following outputs
![docker build](https://user-images.githubusercontent.com/68542385/222981084-91a273d6-d3d1-4b7d-996b-6e96a0f4dbe3.PNG)
![docker run](https://user-images.githubusercontent.com/68542385/222981087-5501ab9a-1f57-40e9-97d8-f10a3d2bcb43.PNG)

2. Push and tag a image to DockerHub 
