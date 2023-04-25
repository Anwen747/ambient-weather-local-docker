# Get data directly from your Ambient Weather station console and publish it over MQTT using  a Python script in Docker

Create a Docker container which will run a Python script to receive weather data from your Ambient Weather console completely locally, and publish it via MQTT.  This is completely independent of any cloud services.  This works with the Ambient Weather 2902-C weather station, but may work with other models.

All credit for the Python script goes to Austin at https://austinsnerdythings.com/2021/03/20/handling-data-from-ambient-weather-ws-2902c-to-mqtt

I am very grateful for his sharing his work on his Ambient Weather project on his blog.

# Overview

First the awnet app must be set up to upload directly to your own web server.

Then, to build and run the Docker container, copy the files from this repo with the same directory structure as in this repo (with the Python script in a subdirectory called /src). Then you will

1.  Edit the Python script to add your own credentials.
2.  Build the image.
3.  Run the Docker container.

# Set up the awnet app

Set up the awnet app for your weather station according to the Ambient Weather documention at

https://ambientweather.com/faqs/question/view/id/1857/

On the Customized tab, enter the following:
* your server IP/hostname
* the protocol type - choose Ambient Weather
* the path should be ```/data?```
* the port (your choice)
* the upload interval in seconds (I chose 60)


# Edit the Python script ```main.py```

## Set your MQTT host, username, and password:

In the section ```set MQTT vars```, edit the appropriate fields:

```
MQTT_BROKER_HOST = os.getenv('MQTT_BROKER_HOST', "your_mqtt_broker_ip_address")
MQTT_USERNAME = os.getenv('MQTT_USERNAME', "your_mqtt_username")
MQTT_PASSWORD = os.getenv('MQTT_PASSWORD', "your_mqtt_password")
```

## Set the port for the web server:

Edit the port number (shown as 7080 in the following example) to match the port number you have set in your awnet app for your Ambient Weather console:
 ```
     httpd = make_server('', 7080, application)
     print("Serving on http://localhost:7080")
 ```
# Build the image from the Dockerfile
From the directory containing the Dockerfile, issue the following command:

```docker build -t ambweather```

(where ```-t``` tags the image with the name *ambweather*)

To see the image you can use:
    
```docker images```

# Run the container

Use the following docker-compose command to run the container in detached mode:

```docker-compose up -d```

You can now use ```docker ps``` to see the running container, which will be named *ambientweather*.  Your weather data should be being published and you should be able to see it in MQTT by subscribing to the topic ```weather/#```.

## Note regarding logging:

Data will be written to the Docker logs. If you haven't already, you might like to consider
using local logging instead of json logging in Docker, where you can set limits on the size of logs, and
rotate the log files.  References I have used:

https://docs.docker.com/config/containers/logging/configure/
https://juhanajauhiainen.com/posts/why-docker-is-eating-all-your-diskspace

# Other Information

## Notes on the Python script:

The original script is from

https://austinsnerdythings.com/2021/03/20/handling-data-from-ambient-weather-ws-2902c-to-mqtt

I made a small change to the original code:

In the ```application``` section, I changed two lines where the parsing occurs.  I used a comment from the post linked above by Murl Easley.  I commented out the line

```parsed = urlparse(url)```

and changed the next line from 

```result = parse_qs(parsed.geturl())```

to 

```result = parse_qs(environ["QUERY_STRING"])```



## Notes on the requirements.txt file:

Docker uses this to pull in any packages that need to be installed.  In his blog
post, Austin notes that he needed a particular version of paho_mqtt, so that
is the one I have used.

## Notes on the Dockerfile:

I am not a Docker expert by any means - this is the first container I have created, so I do not know if I have done things the best or most efficient way.  These are the references I used to create the Dockerfile:

For the basics on creating a Dockerfile:

https://www.docker.com/blog/containerized-python-development-part-1/

For creating a non-root user:

https://github.com/christianlempa/videos/tree/main/docker-python-debugging-vscode

Prior to creating this Docker container, I was running the Python script as a service on my Ubuntu 2022.04 OS. I chose to base my Docker image on Python 3.10 simply because that is the version of Python I was running in Ubuntu.

## Notes on the docker-compose file:

```network_mode: host``` was needed in order for the data from the console to be received by the web server created in the Python script.