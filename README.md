# Securing SSH with step-ca server and KeyCloak(OAuth and OpenID)

_Team **WeARENotGoodAtThis** : [Kirtan Patel](kirtandp@andrew.cmu.edu), [Aryan Chandrakar](aryancha@andrew.cmu.edu), [Naina Kuruballi Mahesh](nkurubal@andrew.cmu.edu), [Sivani Papini](spapini@andrew.cmu.edu)_

## Introduction
In this lab, students will learn to secure web application by generating TLS certificates, once the certificates are generated students will learn about automating the renewal process, creating short lived certificates and issue customized certificates.

### Step-Ca
[Step-ca](https://smallstep.com/docs/step-ca#introduction-to-step-ca) is an online Certificate Authority (CA) for secure, automated X.509 and SSH certificate management. It's secured with TLS and offers several configurable certificate provisioners, flexible certificate templating, and pluggable database backends to suit a wide variety of contexts and workflows. It employs sane default algorithms and attributes, so you don't have to be a security engineer to use it securely.

### KeyCloak
[Keycloak](https://www.keycloak.org/) is an open source IAM (Identity and Access Management) solution which provides desirable features like Single Sign-On and Single Sign-Out through the use of standard protocols like OpenID Connect and OAuth 2.0. As you will see through this lab, it provides a centralized management system for the admins and users. It can also be integrated with LDAP and Active Directory for authentication and authorization purposes. Some other features of keycloak are its high performance, easy scalability, ability to implement custom password policies, and many others

### OAuth
[OAuth](https://oauth.net/) Open Authorization (OAuth) is an open standard that provides a secure method to delegate access. With this method, an application can securely perform tasks on behalf of a user without requiring the users to provide their credentials. This is possible through the token provided by an Identity Provider (IdP) to the third-party application with approval from the user.

### OpenID
[OpenID](https://openid.net/connect/) OpenID is a decentralized authentication protocol that allows users to authenticate themselves using a third-party identity provider (IdP) service. With this mechanism, users can log in and access various independent services which are not related to each other. It eliminates the need to create and manage usernames and passwords for different services; thus it reduces the possibility of a compromise by eliminating problems caused by weak passwords and password reuse.

## Learning Objective
Through this lab, the students will be able to understand how we can configure step-ca to simplify certificate management in the enterprise network, get to know the setup procedure for the keycloak server and leverage OpenID and OAuth to manage authentication and authorization in  day to day application in a better way. Through this lab they will understand the hasseles and issues with the assymetric key ssh setup, and how step-ca can help with the same.

## Problem Statement
Currently,  the status quo method for setting up secure ssh connections is by importing your public key to the server or downloading a private key of the server for initial access. However, even though communication is secure in this method, the management of users is complicated especially when you have multiple users. For example, one needs to make sure that if someone leaves your company, their access is revoked. Hence, we need a better alternative to solve this access management problem.

## Scenario
Access to the empolyee was provided by the admin by sharing the key via a file. The employee was recently fired, he access the server messes up with it. You have taken it upon yourself to secure the same, using step-ca, Let's go with it!

## Machines
| Name            | Operating System | Credentials     |
| --------------- |:----------------:| :--------------:|
| Step-CA Server  | Ubunutu          | Student/tartans |
| KeyCloak Server | Ubunutu          | Student/tartans |
| Client          | Ubunutu          | Student/tartans |
| Resource Server | Ubunutu          | Student/tartans |

### Network Diagram

![image](https://user-images.githubusercontent.com/49098125/202936858-020c34ec-4f46-4e52-8fd3-c173d2f594d3.png)

## Procedure
### Enabling hostname to IP address resolution
The /etc/hosts file contains the Internet Protocol (IP) host names and addresses for the local host and other hosts in the Internet network. This file is used to resolve a host name into an IP address. In our Network, we will be using this file to resolve the IP addresses of the KeyCloak server and the resource server instead of using a DNS resolver. To make sure we do not run into any address resolution problems, let's add all the required host-address mapping into the hosts file.

1. Login into server-ubuntu VM. Password: tartans
2. Open a terminal and run ifconfig to find the IP address of the system. Record this IP address somewhere as testhost IP address.
3. Repeat steps 1 and 2 for **keycloak-server-ubuntu22** as well.

4. Login into client-ubuntu22 VM. Password: tartans
5. Open a terminal and run the following command -  
`$ sudo gedit /etc/hosts`
6. The file will open in edit mode.
7. Now you need to add the following two to the end of the file.
testhost [IP address of server-ubuntu]
keycloak.internal [IP address of keycloak-server-ubuntu22].

Look at lines 5 and 6 in the image below for reference. [Do not copy the IP address shown in the image.]

<img width="500" alt="picture6-928536533" src="https://user-images.githubusercontent.com/49098125/202944327-f9995082-bc68-4b44-b112-4fc02148eb3b.png">

8. Repeat steps 4 to 7 on **step-ca-server-ubuntu22 and server-ubuntu ** VMs as well. 

This is an important step. If in any part of the lab you get address resolution error, the first step to check is the /etc/hosts file.

Now run grading_script1.py on **step-ca-server-ubuntu22**, **client-ubuntu22** and **server-ubuntu** to verify that you have added the required host-address entries.

- Open a terminal.
- run the following command from /home/student.  
`python3 grading_scripts/grading_script1.py`

*Do not execute the grading scripts with sudo*


### Setting up Step-ca server
### Initialize certificate authority
In this step we will be setting up our CA server. The certificate authority is what you’ll be using to issue and sign certificates, knowing that you can trust anything using a certificate signed by the root certificate.

* Login into the step-ca-server-ubuntu22 machine. Password: tartans

* Open a terminal and run the following command [All the setup will be done in the home folder]:  
    `$ step ca init –ssh`  
[Remember to add the --ssh argument, else the setup occurs on http.]

* You will prompted to choose the deployment type. Choose Standalone.

![1](https://user-images.githubusercontent.com/49098125/202931588-5e4b3ef3-0def-403c-8336-93611b5eda3d.png)

* In the next step you will be asked to name the PKI. Enter any name of your choice like “test” or anything else.
* For DNS names or IP addresses - enter the IP address of the VM. [ Use ifconfig to get the lan IP address of the machine. Remember the IP address needs to be the public IP address of the and not the loopback address.]
* In the next step, you will be again asked to enter the IP address and port to bind your CA to. Here enter the [IP address]:5555. This will bind that port to listen for connections from servers and clients.
* When prompted to name the first provisioner enter student@smallstep.com.
* For password enter tartans.
* The final output should look something like this 

![2](https://user-images.githubusercontent.com/49098125/202931535-db3a35a4-c4c2-41c3-a234-bbf8d745795a.png)


### Starting the step-ca server
* Login into the step-ca-server-ubuntu22 machine. Password: tartans
* Open a terminal if not already open.
* Navigate to home/student/.step/config directory using the cd command.
* To start the server, enter the command  `$ step-ca ca.json`
* You will be asked to enter the password. Remember the password we used to initialize the step-ca? It is “tartans”.
* Enter tartans again as password whenever prompted [A total of 3 time].
* Keep this instance running. Check the output, the server must have started running on the provided [step-ca VM IP address] :5555, this can now be accessed via our other system.

![3](https://user-images.githubusercontent.com/49098125/202931652-7f068618-9146-4108-b32d-e4be8b19f2c5.png)


### Generating Certificate
* Open a new terminal in the step-ca-server-ubuntu22 machine. Navigate to home by using cd /home/student command.
* Enter the following command to generate certificates: `step ca certificate keycloak.internal tls.crt tls.key --kty RSA`
* press enter to select student@smallstep.com [provisioner that we created in the last step] as the provisioner.
* Enter tartans as the password whenever prompted.
* Now the certificates are created and stored in the /home/student directory.
* You can inspect the validity of the certificate using the inspect command: ` step certificate inspect --short tls.crt`
* This provides the necessary details about the certificate like expiration date-time, creation date-time and key used to sign the certificate. 
* Now, go to the /home/student/.step/certs directory and install the root certificate using the command `step cetificate install root_ca.crt` 
* Enter “tartans” as the password.  
At the end of this step, your output should look something like this –

![5](https://user-images.githubusercontent.com/49098125/202932958-d50d9acd-c76b-4740-a692-cb1fd0111dcb.png)

We are know done with setting up our step-ca server. 
Run grading script 1 using the following command:  
`python3 /home/student/grading_scripts/grading_script2.py`  

*Do not execute the grading scripts with sudo*
    
    
### Transfering certificate and key
Python provides http server in python3(which has been already installed for your ease), it’s a useful tool for transferring files over the internet.
* Open a new terminal and navigate to /home/student.
* Start a python server by using the command: `python3 -m http.server 12345`  
A python server will be started at port 12345 and can be now accessed by any system on the same network. This http server can be used to download the files in the /home/student directory.


### Download TLS files
* Start the keycloak server machine. Password: tartans
* Open a browser and enter the following URL: http://[IP of the step-ca server]:12345 to access the http server running in our step-ca server.
* Download tls.crt and tls.key.
* Copy both the files to certs directory in the desktop.
* The desktop also contains a yaml file, open it to get credentials and keep a note of the same, might be useful.

  
### Starting Keycloak
  
* Go to the keycloak server, in the Desktop directory, start the docker first using the command `sudo docker compose up`.
* Browse to https://localhost:8443, accept the risk.
* You would reach the keycloak main page, login into administrative panel using the credential found in yaml file.
* From the left panel click on Master, from the drop down list click on create realm. Here you dont need to upload a resource file, all you need to do is give a realm name as step-ca and keep the enabled button on. 
* Now we create a client using import client option found in the client's tab left panel and import a JSON file which can be found on the desktop and give a client ID to be step-ca and click on save at the bottom.
* Go to the credential tab and copy the secret, which will have been auto-generated.
* Now we create a user, on the left panel, under user's tab, create user, provide it an ID, email id and check the email verified option (keep it on). The ID and email id can be anything of your choice. 
* Under the credential tab set a password and remember the same for future. Also disable the temporary option there.
  
*In case any error shows up on the keycloak panel, clear the browser cache. To do so go to settings by clicking on the application menu on the right top corner of the webpage. Settings -> privacy & security -> clear the cookies and site data.*



Now open a new terminal on Step-ca server and run the grading script.
- run the following command from /home/student.  
`python3 grading_scripts/grading_script3.py`

*Do not execute the grading scripts with sudo*
 

### Adding Provisioner
After successfully creating the user on the keycloak, we need to now log into the stepca system. Here, using the client secret that we copied earlier from the credentials tab.

First, we need to configure step-ca to accept your client. 

Add client's ID, Client's secret and realm name which you specified when configuring keycloak to the command and run the command 
`step ca provisioner add keycloak --type=OIDC --client-id [client_id] --client-secret [client secret] --configuration-endpoint https://keycloak.internal:8443/realms/[realm_name]/.well-known/openid-configuration --listen-address :10000`

_Make sure that the client secret is the same as the one on the keycloak._

Now, on the step-ca server, we stop the step-ca instance using ctrl + C and then restart your step-ca instance using `step-ca ca.json`.

On another terminal run the following command:

`step ssh login [user's_email]`

_Remember to use the same email address that was used for your keycloak user._

If all the steps were followed as mentioned then, the server will provide a link that will allow you to access the interface. Along with it an SSH certificate will be issued and added to your SSH agent, with your username and email address as one of the principals.

  

### Server keys and ssh service
On resource server machine open up a terminal and now in the ~/.step/certs directory run the command `step ca bootstrap --ca-url [URL_provided_during_step-ca_initialization] --fingerprint [fingerprint_found_during_start_of_step-ca_server]`. 

_The fingerprint can be found on the Step-ca server, on the terminal where we ran the step-ca instance._

In the *~/.step/certs* directory, lets create a host using the command `step ssh certificate --host testhost ssh_host_ecdsa_key`.
Choose the first option, and enter the required details.

Provide the password to decrypt the provisioner key (type in the password that you created earlier)

Once key pair and certificates have been generated restart the ssh service using `sudo service ssh restart`
You can check the status of the service using `sudo service ssh status` would look something like.
![server restart](https://user-images.githubusercontent.com/49098125/202933585-5886bf6e-5f33-4c88-9010-a7a38e507080.png)

Now we add our earlier created user(who we created on keycloak) to the resource server as sudoers, this helps in connecting to resource server via ssh connection without need of the super user access.

Run the command `sudo adduser --quiet --disabled-password --gecos ' ' [username]` to add our user.

![adding user on server](https://user-images.githubusercontent.com/49098125/202934307-003749bd-d95a-4235-af0a-a101074c2eb7.png)


Now open a new terminal on Step-ca server and run the grading script.
- run the following command from /home/student.  
`python3 grading_scripts/grading_script4.py`

*Do not execute the grading scripts with sudo*

  
### Single SSH Sign-On from client to server

Run the following command in the directory ~/.step/certs to install root certificate `step certificate install root_ca.crt` 

Now lets now run the command `step ca bootstrap --ca-url [URL_provided_during_step-ca_initialization] --fingerprint [fingerprint_found_during_start_of_step-ca_server]`. 

If asked to overwrite defualts.json and root_ca.crt, press [y] and continue

Install the CA cert for validating user certificates (from /etc/step-ca/certs/ssh_user_key.pub)
`step ssh config --roots > $(step path)/certs/ssh_user_key.pub`. This would look something like

![client sys validiating user certificate](https://user-images.githubusercontent.com/49098125/202934441-adfd4e84-c039-4fb1-af9e-0c8010cdb4e8.png)

Now using the created host lets login into the resource server system using secure ssh, and to do so first move to the /.step/certs directory using `cd ~/.step/certs/`. We'll first list all the hostnames on the server by doing `step ssh hosts`. Now ssh into the server using the hostname found in the previous command (that we created earlier) by running `ssh [hostname]`. Upon successful execution of the command, we can now get a ssh connection into the server securely without enterying any password because we already have the certificate for the same authenticating us as valid user.

![client ssh into host wihtout password, we have certificate no key needed](https://user-images.githubusercontent.com/49098125/202934661-b01b05b4-ac30-47f3-b39e-1296ec6ae2c0.png)

Now open a new terminal on Step-ca server and run the grading script.
- run the following command from /home/student.  
`python3 grading_scripts/grading_script5.py`

*Do not execute the grading scripts with sudo*
  
## Conclusion


## References
* Step CA Documentation: https://smallstep.com/docs/step-ca
* Step CA Tutorials : https://smallstep.com/docs/tutorials
* Small Step Blog for SSH DIY : https://smallstep.com/blog/diy-single-sign-on-for-ssh/ 
* Single Sign on for SSH : https://www.youtube.com/watch?v=ZhxLRlcNUM4
* KeyCloak home page : https://www.keycloak.org/

## Appendix  
#### Grading Script 1  

```
hosts = open("/etc/hosts", "r")

keyCloak = False
testhost = False

for line in hosts:
    if "keycloak.internal" in line:
        keyCloak = True
    if "testhost" in line:
        testhost = True

if not keyCloak:
    print("IP address of KeyCloack server not added to /etc/hosts list. Score += 0.0")
else:
    print("IP address of KeyCloack server successfully added to /etc/hosts list. Score += 5.0")


if not testhost:
    print("IP address of testhost not added to /etc/hosts list. Score += 0.0")
else:
    print("IP address of testhost successfully added to /etc/hosts list. Score += 5.0")

```


#### Grading Script 2  


```
import os
from pathlib import Path
import json
import socket
import netifaces as ni
home_directory = os.path.expanduser("~")

step_config_path = os.path.join(home_directory, ".step", "config")
step_certs_path = os.path.join(home_directory, ".step", "certs")
step_secrets_path = os.path.join(home_directory, ".step", "secrets")
score = 0.0

# check if all the config files exist
config_file = ["ca.json", "defaults.json"]
if not all(list(map(os.path.isfile,[ os.path.join(step_config_path, file) for file in config_file]))):
    print("Failed! Looks like some config files doesn't exist.")
    print("Score = 0.0")
    exit()

# check all the certs and keys exist
certs_list = ["intermediate_ca.crt",  "root_ca.crt",  "ssh_host_ca_key.pub",  "ssh_user_ca_key.pub"]
if not all(list(map(os.path.isfile,[ os.path.join(step_certs_path, file) for file in certs_list]))):
    print("Failed! Looks like some certs doesn't exist.")
    print("Score = 0.0")
    exit()

# check all the private keys exist
secret_keys = ["intermediate_ca_key", "root_ca_key", "ssh_host_ca_key", "ssh_user_ca_key"]

if not all(list(map(os.path.isfile,[ os.path.join(step_secrets_path, file) for file in secret_keys]))):
    print("Failed! Looks like some private keys doesn't exist.")
    print("Score = 0.0")
    exit()

print("\nStep ca setup was successful! +10.0\n")
score += 10.0

print("Checking the connection to step-ca")

f = open(os.path.join(step_config_path, 'defaults.json'))
data = json.load(f)

url = data['ca-url'].split(":")
port = url[2]
print("Port = " + port)
ip = url[1][2:]
fingerprint = data['fingerprint']

sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

port = 5555
ip = ni.ifaddresses('ens32')[ni.AF_INET][0]['addr']
print("Ip = " + ip)

result = sock.connect_ex((ip,port))
if result == 0:
   print("Server is up and running")
   score += 10.0
else:
   print("Oops! Server is not running")
sock.close()
print("\nFinal score = " + str(score) + " out of 20.0\n")
```

#### Grading Script 3  


```
import socket

def check_keycloack_connection(ip):
    try:
        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        if s.connect_ex((ip, 8443)) == 0:
            print("\nSuccessfully connected to keycloak server\nScore = 10.0\n")
        else:
            print("\nCheck connectivity with KeyCloak server: " + ip + ":8443\nScore = 0.0\n")
    except:
        print("\nCheck connectivity with KeyCloak server: " + ip + ":8443\nScore = 0.0\n" )
        exit()

if __name__ == '__main__':
    hosts = open("/etc/hosts", "r")
    ip = ""
    for line in hosts:
        if "keycloak.internal" in line:
        	ip = line.split()[0].strip()
        	print("IP address of keycloack server = " + ip)
    if not ip:
        print("Could not get IP address from /etc/hosts. Score = 0.0")
        exit()
    check_keycloack_connection(ip.strip())
```


#### Grading Script 4  


```
import os

output = os.popen("service ssh status")
terminal_output = output.read()
if not terminal_output:
    print("Error: ssh setup failed\nScore = 0.0")
    exit()
    
if "active" in terminal_output and "SUCCESS" in terminal_output:
    print("ssh setup success\nScore = +10.0")
```

#### Grading Script 5  


```
from paramiko import SSHClient
import os

client = SSHClient()
client.load_host_keys("/home/student/.step/ssh/known_hosts")
# get hostname here
output = os.popen("step ssh hosts")
hostExist = False
for line in output:
    if "testhost" in line:
        print("testhost exists in the hosts list! Score += 5.0")
        hostExist = True
if not hostExist:
    print("testhost does not exist in the hosts list... Score = 0.0")
    exit()

try:
    client.connect("testhost", username="tartan")
    print("Successfully connected to remote host. Score += 5.0")
    client.close()
except:
    print("ERROR: Could not connect to the remote host. Check if proper keys exists in ~/.step/ssh/known_hosts. Score = 0.0")
```
