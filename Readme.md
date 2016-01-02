
# How to generate a root CA to sign Mosquitto Broker and Client instances

__Warning: Root CA keys won't be encrypted and be valid for 10 years!  
This configuration is help full for development.
But insecure for production.__

Initial description found here:  
https://wiki.ubuntuusers.de/CA  
CA_mod.pl is mostly a copy of _CA.pl_ by Tim Hudson which on Ubuntu can be found here:
```Shell
/usr/lib/ssl/openssl.cnf
```
CA_mod.pl output dramatically less secure keys then the Original _CA.pl_

## Server Setup
Clone the project and ```cd``` into the folder.

### 1. Setup the root CA
```Shell
/CA_mod.pl -newca
```
Common Name must be unique for this authority i.e. _myCA_.  
The expiration Dates are defined in _CA_mod.pl_.  

## Setup public Broker

### 2. Create a keys & request signature file files for the public Broker
First get the FQDN (Fully qualified domain name) of your Server, which you will use when asked for a common name.  
```Shell
echo $HOSTNAME
```
Generate key & request file
```Shell
./CA_mod.pl -newreq  
# Just hit enter when asked for  
A challenge password []:  
&  
An optional company name []:  
```
The following Files will appear in your Folder:
- __newkey.pem__ _Your key. You will never ever give this key to anyone._
- __newreq.pem__ _The request if you wan't to sign a Client Key this is all what is needed._

### 3. Sign the request newreq.pem
The expiration time is defined at _openssl.cnf_. You might want to change it.  
```Shell
default_days	= 3590			# how long to certify for
```

```Shell
./CA_mod.pl -sign
```
_newreq.pem_ the request File needed to generate the key can be deleted.
```Shell
rm -rf newreq.pem
```

Rename your certs to something meaningful. i.e. your $HOSTNAME _example.com.key.pem_ _example.com.cert.pem_

### 4. Copy the keys into an appropriate folder.
i.e.
```Shell
cp demoCA/cacert.pem /etc/mosquitto/tls/cacert.pem
cp example.com.cert.pem /etc/mosquitto/tls/example.com.cert.pem
cp example.com.key.pem /etc/mosquitto/tls/example.com.key.pem
```

### 5. Edit mosquitto.config
add the following lines:
```Shell
cafile /etc/mosquitto/tls/cacert.pem
certfile /etc/mosquitto/tls/example.com.cert.pem
keyfile /etc/mosquitto/tls/example.com.key.pem
require_certificate true
```

### 6. Restart Mosquitto and see if it works.
```Shell
# eventually stop mosquitto with your os service
# service mosquitto stop
# or
# /etc/init.d/mosquitto stop
# or the brut force method:
killall mosquitto
# Restart mosquitto from the shell in verbose -v mode to see whats going on.
/usr/sbin/mosquitto -v -c /etc/mosquitto/mosquitto.conf
```
## Setup a Client
Clone the project and ```cd``` into the folder.

### 7. Repeat Steps 2,3,4 for the client.
For security reasons you should run step 2 on the client and only send the _newreq.pem_ to the ca server.

### 8. Check if it works

Subscribe
```Shell
mosquitto_sub -v -h example.com -p 8883 -t /test01 --cafile cacert.pem --cert client01.cert.pem --key client01.key.pem
```
Publish
```Shell
mosquitto_pub -h example.com -p 8883 -t /test01 -m "HELLO" --cafile cacert.pem --cert client01.cert.pem --key client01.key.pem
```
Have a eye on the shell running your Mosquitto Broker in case something goes wrong.

## Connect a local Broker

### 9. Repeat Steps 2,3,4 for the local Broker.
For security reasons you should run step 2 on the client and only send the _newreq.pem_ to the ca server.

## Edit the local Broker Config
See __mosquitto.conf.bridge__ how to setup a Bridge.

Now you can communicate encrypted from local Devices to your local Broker and communicate with the remote Broker encrypted.
Helpful to embed low power nodes i.e. sensors & arduinos which don't allow TLS encryption.

# Helpful snippets
openssl x509 -noout -text -in client01.pem
