
# ifconfig.pm

## README

This is a slightly modified version of https://github.com/georgyo/ifconfig.io :
* Support for HTTP Connection, Charset, Via, Do-Not-Track and Cache-Control headers
* Added Protocol field and a way of displaying the original protocol if running behind an HTTP proxy 
* Added a way to support real client port when the app is run behind an HTTP proxy
* wget and fetch are treated like curl (providing value without html formating)
* show IP country and AS name/number based on MaxMind GeoLite2 free database using https://github.com/oschwald/maxminddb-golang

#### Build instruction :
* install golang-go
* git clone https://github.com/pfoo/ifconfig.pm.git
* cd ifconfig.pm
* export GOPATH="$(pwd)"
* go get -d -v
* go build

A few parameters can be defined using export before launching the binary :
* export GIN_MODE=debug|release
* export HTTP_PORT="8080"
* export HTTP_HOST="127.0.0.1"
* export FCGI_PORT="4000"
* export FCGI_HOST="127.0.0.1"
* export PROXY_TYPE="FCGI|HTTP|BOTH"

Required for ip country and ASN support :
* Download databases from https://dev.maxmind.com/geoip/geoip2/geolite2/ and place GeoLite2-Country.mmdb and GeoLite2-ASN.mmdb on the same directory than the binary.

#### Running behind an HTTP proxy :
* Run the go program on 127.0.0.1:8080 with PROXY_TYPE environement variable set to HTTP
* Use apache mod_proxy_http to proxy requests from apache to http://127.0.0.1:8080/
* Add followings headers to the proxyfied requests :<br>
	CF-Connecting-IP : The IP address the client is connecting from. Important or the IP will be wrong.<br>
	CF-Connecting-PORT : The port the client is connecting from. Important or the PORT will be wrong.<br>
	CF-Connection : HTTP_CONNECTION header sent by the client to the proxy<br>
	CF-Protocol : The HTTP protocol the client is connecting with. Important or the displayed protocol will be wrong.<br>
* See example apache.http.conf

#### Running behind an FCGI proxy :
* Run the go program on 127.0.0.1:4000 with PROXY_TYPE environement variable set to FCGI
* Use apache mod_proxy_fcgi to proxy requests from apache to fcgi://127.0.0.1:4000/
* See example apache.fcgi.conf

#### Running behind systemd socket activation (see http://0pointer.de/blog/projects/socket-activation.html)
* Run : systemd-socket-activate -l 8000 ./ifconfig.pm
* systemd will listen on port 8000, start the program only when required and forward everything to it using socket
* If everything is working, you can create two files in /etc/systemd/system/ in order to automatically launch the service :<br>
	ifconfig.socket (see example file)<br>
	ifconfig.service (see example file)<br>
* Then execute `systemctl daemon-reload` and `systemctl start ifconfig.socket`

#### ORIGINAL README FROM https://github.com/georgyo/ifconfig.io :

Inspired by ifconfig.me, but designed for pure speed. A single server can do 18,000 requests per seconds while only consuming 50megs of ram.

I used the gin framework as it does several things to ensure that there are no memory allocations on each request, keeping the GC happy and preventing unnessary allocations.

Tested to handle 15,000 requests persecond on modest hardware with an adverage response time of 130ms.
![LoadTest](http://i.imgur.com/xgR4u1e.png)
