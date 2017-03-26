# Website Manager
A configuration to serve multiple domains using Nginx and Flask on a Raspberry Pi 3.

## Pi Setup
Download a copy of the 64 bit Hypriot OS image from here : [https://github.com/DieterReuter/image-builder-rpi64/releases](https://github.com/DieterReuter/image-builder-rpi64/releases).

Stick onto an SD card following the guide here: [http://blog.hypriot.com/getting-started-with-docker-and-linux-on-the-raspberry-pi/](http://blog.hypriot.com/getting-started-with-docker-and-linux-on-the-raspberry-pi/).

Stick the SD card with loaded image into your Raspberry Pi 3. Plug in.

(Optional but highly recommended: change the hostname, default user, password, root password, ssh settings etc.)

SSH into the Raspberry Pi (with default settings: ```ssh pirate@black-pearl```, password: hypriot).

## WebApp Setup
Each Flask Web app can be configured as a separate GitHub repository. 

It should have a directory structure of:
```
/appX_repository
	appX/
		app.py
		Dockerfile
		static/
			static_stuff.js
			static_stuff.jpg
			static_stuff.css
```
Where 'appX' is 'app1' for the first Flask app, 'app2 for the second Flask app etc.

'app.py' should contain a factory method for creating an app object called ```create_app```.

Dockerfile is a Dockerfile for the app. I use gunicorn to serve the Flask app on
port 8000.

See [this repository](https://github.com/benhoyle/docker-python-test) for details of an example structure.

## Raspberry Pi Directory Structure

On your Raspberry Pi clone the Flask apps into the same folder as the present project.

E.g.:
```
/projects
	/website_manager
		these.files
	/app1_repository
		app1.files
	/app2_repository
		app2.files
```

## Nginx Configuration

You'll need to edit the ```nginx/app.conf``` file to add your domains.

In this file, each ```server {...}``` entry defines one of the Flask Apps.

Change the ```server_name``` from ```www.example1.co.uk``` to your domain for each app.
You can also have configurations such as ```blog.mydomain.com``` and ```www.mydomain.com```.

Add a different server entry for each app. 

The ```web_app1``` entries in the ```proxy_pass``` settings refer back to the 
```links``` entries in the ```docker-compose.yml``` file. Similarly, the port
should be the port bound to gunicorn and exposed in the YAML file.

## docker-compose.yml

In the services section there is an entry for each Flask App and an entry for
the Nginx reverse proxy. Each web_app ```build``` entry points to the contents of 
a corresponding ```appX_repository``` directory (in particular, the ```Dockerfile``` therein).

For production, I'd remove the ```--reload``` flag from the gunicorn command.

The volumes map the static directory on the host machine (the Raspberry Pi) to
a directory in the Flask App container. The nginx container has a ```volumes_from:```
entry that imports these static directories from both App containers. This allows
static content for both apps to be served directly by Nginx (which is faster). In the
```nginx\app.conf``` file you will see that these volumes are referred to in the
static location line.

The ```restart: always``` entry just means that the containers will be restarted
if the Pi restarts or an error occurs.

## Deploying

CD into website_manager.

Run docker-compose: ```docker-compose up --build```.

If it all builds and starts correctly, point your browser at the domains.

You may also need to confirm your router to forward requests on port 80 to the
IP address of the Raspberry Pi.
