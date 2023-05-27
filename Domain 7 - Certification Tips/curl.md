
##### Curl command
cURL is a command-line utility that allows you to perform tons of tasks:
- Transfer data to or from servers
- Display the status of various network configurations of Windows machines or Linux servers

To install cURL on your Ubuntu machine:
``````sh
mkdir ~/install_curl_demo
cd ~/install_curl_demo

sudo apt update
sudo apt install curl
curl --version
``````

##### Downloading Files With Original or Changed Name
The curl command, appended with either the -O or -o options,
lets you download files while keeping the original or setting a different name.

``````sh

# Downloads the NGINX package with the original name nginx-1.20.2.tar.gz
curl -O <http://nginx.org/download/nginx-1.20.2.tar.gz>
# Downloads the NGINX package with the new name mytar.gz
curl -o mytar.gz <http://nginx.org/download/nginx-1.20.2.tar.gz>

``````
##### Retrieving Data Using the GET Request
Run the curl command below to use an HTTP GET request (–get) and retrieve data from the https://adamtheautomator.com website.

``````sh

curl --get <https://adamtheautomator.com>

``````
##### Testing Website HTTP2 Protocol
you can check whether a particular URL supports the new HTTP/2 protocol.

``````sh
curl -I --http2 -s <https://adamtheautomator.com/> | grep HTTP

``````