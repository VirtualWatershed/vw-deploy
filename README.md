# vw-deploy
Docker compose files for the vw services

# How to

### Dev Environment

All we need is:

```
cd v1.0
docker-compose -f docker-compose.dev.yml up
```

It will make the following services available:


 Service  |                 Desc                 |          URL
:--------:|:------------------------------------:|:---------------------:
vwadaptor |         vw model webservice          | docker-machine-ip:5000
vwmonitor |            Celery monitor            | docker-machine-ip:5555
 vwauth   |         auth service for vw          | docker-machine-ip:5005
 vwmail   | a local smtp mail server for testing | docker-machine-ip:1080
vwworker  |        worker service for vw         |           X

To test out auth:

* Register a user at http://docker-machine-ip:5005/register.

* Go to the mail client at http://docker-machine-ip:1080 and click on the confirmation link

* Then get an auth token with:

```javascript

POST http://docker-machine-ip:5005/api/v1/auth
Content-Type: application/json

{
  "email":"a@b.com",
  "password":"123456",
}

```
It should return:

```json
{
  "access_token": "some_jwt_token"
}
```
* Now test it against vwadaptor service with:

```javascript

GET http://docker-machine-ip:5000/testjwt
Content-Type: application/json
Authorization: JWT some_jwt_token
```

If everything goes right it should return:

```
You are a@b.com
```
