# vw-deploy
Docker compose files for the vw services

# How to

### Dev Environment

All we need is:

```
docker-compose -f docker-compose.dev.yml up

```

It will make the following services available:

* vwadaptor: docker-machine-ip:5000
* vwamonitor: docker-machine-ip:5000



 Service  |         Desc          |          URL
:--------:|:---------------------:|:---------------------:
vwadaptor |  vw model webservice  | docker-machine-ip:5000
vwmonitor |    Celery monitor     | docker-machine-ip:5555
 vwauth   |  auth service for vw  | docker-machine-ip:5005
vwworker  | worker service for vw |           X
