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
| Name            | Operating System | IP Address | Credentials     |
| --------------- |:----------------:| :---------:| ---------------:|
| Step-CA Server  | Ubunutu          | 10.5.5.143 | Student/tartans |
| KeyCloak Server | Ubunutu          | 10.9.8.31  | Student/tartans |
| Client          | Ubunutu          | 10.5.5.119 | Student/tartans |
| Server          | Ubunutu          | 10.9.2.252 | Student/tartans |

## Network Diagram

## Setting up Step-ca server
### Initialize certificate authority
The certificate authority is what you’ll be using to issue and sign certificates, knowing that you can trust anything using a certificate signed by the root certificate.

Run the command `step ca init --ssh` in a terminal to configure your CA. You'll be asked to enter few information.
_Remember to add the `--ssh` argument, else the setup occurs on http._

* Enter any name based on your preference
* Provide the system's IP address, you should know how to find that.
* Enter the IP address and port to be 5555 in the format [IP:port] 
* Provide the CA's first provisioner
* Enter a password, keep a note of the same for future purpose.

### Start the server
* You need to go to the directory /.step/config and now you may start the server now using the command `step-ca ca.json`
* Enter the asked detials.
* Keep this instance running.
_*Check the output, the server must have started running on the provided IP:port, this can now be accessed via our other system*_

### Certificate
On a new terminal generate certificate using the command `step ca certificate keycloak.internal tls.crt tls.key --kty RSA` and choose the first provisioner.
You can inspect the validity of the certificate using the inspect command ` step certificate inspect --short tls.crt`, This provides the necessary details about the certificate like expiration date-time, creation date-time and key used to sign the certificate.
Now, go to the /.step/certs directory and install the root certificate using the command `step cetificate install root_ca.crt` 

### Transfering certificate and key
Python provides http server in python3(which has been already installed for your ease), it’s a useful tool for transferring files over the internet.

Start a python server on a new terminal using the command `python3 -m http.server [port of choice]`. This server ip:port can be accessed by anyone on the same network and be used to download any file from the directory where the command was executed.

#### Download TLS files
On the keycloak machine access the python server initiated on the stepca server using the URL http://[IP]:[port] where the IP is that of the step-ca server's IP address and port is 5555.
And download the tls.crt and tls.key, copy them in the cert's directory in desktop.

The desktop also contains a yaml file, open it to get credentials and keep a note of the same, might be useful.

### Starting Keycloak
* Go to the keycloak server, in the Desktop directory, start the docker first using the command `sudo docker compose up`.
* Browse to https://localhost:8443, accept the risk.
* You would reach the keycloak main page, login into administrative panel using the credential found in yaml file.
* From the left panel click on Master, from the drop down listed click on create realm. Here you dont need to upload a resource file, all you need to do is give a realm name as step-ca and keep the enabled button on. 
* Now we create a client using import client option found in the client's tab left panel and import a JSON file which can be found on the desktop and give a client ID to be step-ca and click on save at the bottom.
* Go to the credential tab and copy the secret, which will have been auto-generated.
* Now we create a user, on the left panel, under user's tab, create user, provide it an ID, email id and check the email verified option (keep it on). The ID and email id can be anything of your choice. 
* Under the credential tab set a password and remember the same for future. Also disable the temporary option there.
*In case any error shows up on the keycloak panel, clear the browser cache. To do so go to settings by clicking on the application menu on the right top corner of the webpage. Settings -> privacy & security -> clear the cookies and site data.*

### Adding Provisioner
After successfully creating the user on the keycloak, we need to now log into the stepca system. Here, using the client secret that we copied earlier from the credentials tab.

First, we need to configure step-ca to accept your client. 

Run the command `step ca provisioner add keycloak --type=OIDC --client-id step-ca --client-secret [client secret] --configuration-endpoint https://keycloak.internal:8443/realms/step-ca/.well-known/openid-configuration --listen-address :10000`

_Make sure that the client secret is the same as the one on the keycloak._

On the step-ca server, stop the step-ca instance using ctrl + C and then restart your step-ca instance using `step-ca ca.json`.

On another terminal run the following command:

`step ssh login user@example.com`

_Remember to use the same email address that was used for your keycloak user._

If all the steps were followed as mentioned then the server will provide a link that will allow you to access the interface. Along with it an SSH certificate will be issued and added to your SSH agent, with your username and email address as principals.

### Server keys and ssh service
On server machine open up a terminal and now in the ~/.step/certs directory run the command `step ca bootstrap --ca-url [URL_provided_during_step-ca_initialization] --fingerprint [fingerprint_found_during_start_of_step-ca_server]`. 

In the *~/.step/certs* directory, lets create a host using the command `step ssh certificate --host testhost ssh_host_ecdsa_key`
Choose the first option, and enter the required details.

Provide the password to decrypt the provisioner key (type in the password that you created earlier)

Once key pair and certificates have been generated restart the ssh service using `sudo service ssh restart`
You can check the status of the service using `sudo service ssh status`

### Single SSH Sign-On from client to server

Run the following command in the directory ~/.step/certs to install root certificate `step certificate install root_ca.crt` 

Now lets now run the command `step ca bootstrap --ca-url [URL_provided_during_step-ca_initialization] --fingerprint [fingerprint_found_during_start_of_step-ca_server]`. 

If asked to overwrite defualts.json and root_ca.crt, press [y] and continue. 

Now using the created host lets login into the server system using secure ssh, and to do so first move to the /.step/certs directory using `cd ~/.step/certs/`. We need to list all the hostnames on the server by doing `step ssh hosts`. It will list all the hostnames. Now ssh into the server using the hostname found in the previous command (that we created earlier) by running `ssh [hostname]` and enter the password. Upon successful execution of the command, we can now get a ssh connection into the server securely. 


### References
* Step CA Documentation: https://smallstep.com/docs/step-ca
* Step CA Tutorials : https://smallstep.com/docs/tutorials
* Small Step Blog for SSH DIY : https://smallstep.com/blog/diy-single-sign-on-for-ssh/ 
* Single Sign on for SSH : https://www.youtube.com/watch?v=ZhxLRlcNUM4
* KeyCloak home page : https://www.keycloak.org/
