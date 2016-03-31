## How To


* Install docker toolbox from: https://www.docker.com/products/docker-toolbox
* Create a virtual machine using docker-machine and activate it (follow the guide at: https://docs.docker.com/machine/get-started/)
* If using kinematic, a virtual machine will spin up automatically with name `default` 

### To run the servers in dev:
* clone https://github.com/VirtualWatershed/vw-deploy
* open docker-cli and goto the directory `v1.0/development`
* Now run `docker-compose -f docker-compose.dev.yml up`

This will taka e few minutes as docker will download all the images from docker-hub. 
After everything is up the following servers will be available:

* ip-of-your-docker-machine:5005 - auth server
* ip-of-your-docker-machine:5000 - model server frontend
* ip-of-your-docker-machine:5555 - a celery monitor server to monitor the modelruns
* ip-of-your-docker-machine:1080 - a dummy smtp server for email confirmation when creating an account in auth server

To know your virtual machine's ip use: `docker-machine ip machine-name`

Now to Run iSNoBAL, follow these steps:

1. Create an account at the auth server `ip-of-your-docker-machine:5005`. Use a dummy email (abc@xyz.com)
2. Verify e-mail at `ip-of-your-docker-machine:1080`
3. Now get an JWT auth token:

```
POST http://ip-of-your-docker-machine:5005/api/v1/auth
Content-Type: application/json
body: {"username":"abc@xyz.com","password":"whatever-you-have-set"}
```
This will return an access token like this:

```json

{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZGVudGl0eSI6MiwiaWF0IjoxNDU5NDU3MTI2LCJuYmYiOjE0NTk0NTcxMjYsImV4cCI6MTQ1OTU0MzUyNn0.0K1vMHCww5vx_D-psUAWPpUbYnlYhUEMZUM7mqHpzxk"
}

```

Now we can go ahead and create a model run.

```
POST http://ip-of-your-docker-machine:5000/api/modelruns
Authorization: JWT eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZGVudGl0eSI6MiwiaWF0IjoxNDU5NDU3MTI2LCJuYmYiOjE0NTk0NTcxMjYsImV4cCI6MTQ1OTU0MzUyNn0.0K1vMHCww5vx_D-psUAWPpUbYnlYhUEMZUM7mqHpzxk
Content-Type: application/json
body: {"title":"some title","model_name":"isnobal"}
```

You should get a response:

```json

{
  "created_at": "2016-03-31T20:45:55.267229+00:00",
  "id": 1,
  "model_name": "isnobal",
  "progress_events": [],
  "progress_state": "NOT_STARTED",
  "progress_value": 0,
  "resources": [],
  "title": "test model run for isnobal",
  "user_id": 1
}

```

Now go ahead and upload the input file for isnobal:

```
POST http://ip-of-your-docker-machine:5000/api/modelruns/<id>/upload
Authorization: JWT eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZGVudGl0eSI6MiwiaWF0IjoxNDU5NDU3MTI2LCJuYmYiOjE0NTk0NTcxMjYsImV4cCI6MTQ1OTU0MzUyNn0.0K1vMHCww5vx_D-psUAWPpUbYnlYhUEMZUM7mqHpzxk
Content-Type: multipart/form-data
Content-Disposition: form-data; name="file"; filename="your-isnobal-input-file.nc"
Content-Type:
Content-Disposition: form-data; name="resource_type"
input
```

So, the server expects two values, `file` and `resource_type`. For isnobal there is only one resource and the `resource_type` is `input` and the file has to be a valid netcdf file.


Now, we can go ahead and start the modelrun by just sending a PUT request like follows:


```
PUT http://ip-of-your-docker-machine:5000/api/modelruns/<id>/start
Authorization: JWT eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZGVudGl0eSI6MiwiaWF0IjoxNDU5NDU3MTI2LCJuYmYiOjE0NTk0NTcxMjYsImV4cCI6MTQ1OTU0MzUyNn0.0K1vMHCww5vx_D-psUAWPpUbYnlYhUEMZUM7mqHpzxk
Content-Type: application/json
```


Now, If we poll the url `http://ip-of-your-docker-machine:5000/api/modelruns/<id>` We will get the modelrun with its progress_state and progress_events. An example request and output would be:

```
GET http://ip-of-your-docker-machine:5000/api/modelruns/<id>
Authorization: JWT eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZGVudGl0eSI6MiwiaWF0IjoxNDU5NDU3MTI2LCJuYmYiOjE0NTk0NTcxMjYsImV4cCI6MTQ1OTU0MzUyNn0.0K1vMHCww5vx_D-psUAWPpUbYnlYhUEMZUM7mqHpzxk
Content-Type: application/json

```


```json

{
  "created_at": "2016-03-31T20:44:18.838088+00:00",
  "id": 3,
  "model_name": "isnobal",
  "progress_events": [
    {
      "created_at": "2016-03-31T20:49:43.593991+00:00",
      "event_description": "creating input ipw files for each timestep from the input netcdf file (stage 1)",
      "event_name": "processing_input",
      "id": 9,
      "modelrun_id": 3,
      "progress_value": 100
    },
    {
      "created_at": "2016-03-31T20:51:14.148490+00:00",
      "event_description": "creating input ipw files for each timestep from the input netcdf file (stage 2)",
      "event_name": "processing_input2",
      "id": 10,
      "modelrun_id": 3,
      "progress_value": 100
    },
    {
      "created_at": "2016-03-31T20:51:16.872416+00:00",
      "event_description": "Running the ISNOBAL model",
      "event_name": "running_isonbal",
      "id": 11,
      "modelrun_id": 3,
      "progress_value": 100
    },
    {
      "created_at": "2016-03-31T20:51:30.362841+00:00",
      "event_description": "creating output netcdf file from output ipw files",
      "event_name": "ouptut_ipw_to_nc",
      "id": 12,
      "modelrun_id": 3,
      "progress_value": 80.5
    }
  ],
  "progress_state": "RUNNING",
  "progress_value": 0,
  "resources": [
    {
      "created_at": "2016-03-31T20:49:15.272223+00:00",
      "id": 5,
      "modelrun_id": 3,
      "resource_name": "isnobalinput__MeUjMvUVdxr7BnLUnn9cW8.nc",
      "resource_size": 19056924,
      "resource_type": "input",
      "resource_url": "http://ip-of-your-docker-machine:5000/api/modelresources/download/isnobalinput__MeUjMvUVdxr7BnLUnn9cW8.nc"
    }
  ],
  "title": "test model run for isnobal",
  "user_id": 2
}

```

We can use the progress events to show progress bar. the `progress_state` should show either of `QUEUED,RUNNING,FINISHED,ERROR`.













