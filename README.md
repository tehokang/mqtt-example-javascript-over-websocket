
# Fixing disconnection by MQTT Broker

## 1. About this

Please refer to wiki explanation about <a href="https://en.wikipedia.org/wiki/MQTT">MQTT</a> if you need background information. (Korean version: https://www.joinc.co.kr/w/man/12/MQTT/Tutorial)

I've met disconnection issue in MQTT scenario between broker(as server) and client. Totally disconnection was happened by server in a few minutes during working well. So I'd like to know why server is disconnecting client even though client looks good ;)

To make simple environment of this, co-worker could find a MQTT server of open source projects like AWS IoT Broker and fortunately I could see AWS IoT Broker is almost working similar to open source project, Mosquitto broker server.
In case of client, we were using javascript application using paho javascript mqtt library




### 1.1 Server reference (C)
* Project Site : https://projects.eclipse.org/projects/technology.mosquitto
* Source on Github : https://git.eclipse.org/r/mosquitto/org.eclipse.mosquitto

### 1.2 Client reference (Javascript)
* Project Site : https://www.eclipse.org/paho/clients/js/
* Source on Github : https://github.com/eclipse/paho.mqtt.javascript

We ignored the backends(like EC2, Lambda) behind AWS IoT browser because there was no any evidence making server disconnect.
For verification, we only prepared above server and client like following to check who it makes disconnection of server, 

<img src="https://github.com/tehokang/public-figures/blob/master/mqtt/mqtt-server-disconnection.png?raw=true"></img>

## 2. How to build, run and test

## 2.1 How to build MQTT broker

Installing prerequisite libraries
~~~
sudo apt-get install cmake uuid-dev xsltproc docbook-xsl build-essential libc-ares-dev libssl-dev libwebsockets-dev
~~~

Downloading and building source of them 
~~~
git clone https://git.eclipse.org/r/mosquitto/org.eclipse.mosquitto
cd org.eclipse.mosquitto/
vi config.mk
~~~

Enable websocket option 
~~~
WITH_WEBSOCKETS:=yes
~~~

Type make command to build and install
~~~
make
make test
sudo make install
~~~

Installing executable mosquitto client if you'd like example of MQTT client binary.
~~~
sudo apt-get install mosquito-clients
~~~


Adding configuration of listening port in /etc/mosquitto/conf.d/websocket.conf
~~~
port 1883
listener 1884
protocol websockets
~~~



## 2.2 How to run MQTT broker

You can type command like following and see ports listening client via netstat
~~~
sudo /usr/local/sbin/mosquitto -c /etc/mosquitto/conf.d/websocket.conf
~~~

~~~
$netstat -atno
...
tcp        0      0 0.0.0.0:1883            0.0.0.0:*               LISTEN      -               
tcp        0      0 0.0.0.0:1884            0.0.0.0:*               LISTEN      - 
...
~~~


## 2.4 How to run client

You can see index.html having simple javascript codes using paho javascript mqtt library.
Just getting started it, Oops I think you should change the port of MQTT broker listening in index.html.

Please change them for your server.
~~~
host = "192.168.0.23";
port = 2002;
~~~

After them, you can open index.html on your browser like chrome.

## 3. conclusion

Finally we could find main reason who are making disconnection. The websocket of MQTT paho javascript library is sending data with wrong hexadecimal at the specific time, for example, a value of first byte of data sending is changed as wrong right before websocket.send(data). I tried to find suspect who are changing the value and found it was a browser which runs client. 

We were using Opera browser as old-fashion(=version), maybe it's using Craken engine. 
Anyway we could fix disconnection when it turned off <font color=red>JIT</font> option.


<center><font size=10 color=red> JIT </font> was suspect.</center>
