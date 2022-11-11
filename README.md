# Securing SSH with step-ca server and KeyCloak(OAuth and OpenID)

_Team **WeARENotGoodAtThis** : [Kirtan Patel](kirtandp@andrew.cmu.edu), [Aryan Chandrakar](aryancha@andrew.cmu.edu), [Naina Kuruballi Mahesh](nkurubal@andrew.cmu.edu), [Sivani Papini](spapini@andrew.cmu.edu)_

## Introduction
In this lab, students will learn to secure web application by generating TLS certificates, once the certificates are generated students will learn about automating the renewal process, creating short lived certificates and issue customized certificates.

### Step-Ca
[Step-ca](https://smallstep.com/docs/step-ca#introduction-to-step-ca) is an online Certificate Authority (CA) for secure, automated X.509 and SSH certificate management. It's secured with TLS and offers several configurable certificate provisioners, flexible certificate templating, and pluggable database backends to suit a wide variety of contexts and workflows. It employs sane default algorithms and attributes, so you don't have to be a security engineer to use it securely.

### KeyCloak
[Keycloak](https://www.keycloak.org/) is an open source IAM (Identity and Access Management) solution which provides desirable features like Single Sign-On and Single Sign-Out through the use of standard protocols like OpenID Connect and OAuth 2.0. As you will see through this lab, it provides a centralized management system for the admins and users. It can also be integrated with LDAP and Active Directory for authentication and authorization purposes. Some other features of keycloak are its high performance, easy scalability, ability to implement custom password policies, and many others

### OAuth
[OAuth]https://oauth.net/) Open Authorization (OAuth) is an open standard that provides a secure method to delegate access. With this method, an application can securely perform tasks on behalf of a user without requiring the users to provide their credentials. This is possible through the token provided by an Identity Provider (IdP) to the third-party application with approval from the user.

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
* Enter the IP address and port in the format [IP:port]
* Provide the CA's first provisioner
* Enter a password, keep a note of the same for future purpose.

### Certificate
Generate certificate using the command `step ca certificate [IP] tls.crt tls.key --kty RSA`
You can inspect the validity of the certificate using the inspect command ` step certificate inspect --short tls.crt`, This provides the necessary details about the certificate like expiration date-time, creation date-time and key used to sign the certificate.
Install the root certificate using the command `step cetificate install root.crt` 

### Start the server
* You may start the server now using the command `step-ca ca.json`
* Enter the asked detials.
_*Check the output, the server must have started running on the provided IP:port, this can now be accessed via our other system*_

### Transfering certificate and key
Python provides http server in python3(which has been already installed for your ease), it’s a useful tool for transferring files over the internet.

Start a python server using the command `python -m http.server [port of choice]`. This server ip:port can be accessed by anyone on the same network and be used to download any file from the directory where the command was executed.

#### Download TLS files
On the keycloak machine access the python server initiated on the stepca server using the URL http://[IP]:[port]
And download the tls.crt and tls.key, copy them in the cert's directory in desktop.

The desktop also contains a yaml file, open it to get credentials and keep a note of the same, might be useful.

### Starting Keycloak
* Start the docker first using the command `sudo docker compose up`.
* Browse to https://localhost:8443, accept the risk.
* You would reach the keycloak main page, login into administrative panel using the credential forund in yaml file.
* Click on the Master in the left panel, in the drop down lst create realm and give it a name.
* Now we create a client, go to the client tab in the left panel and import a JSON file which can be found on the desktop.
* Go to the credential tab and copy the secret, which will have been auto-generated.
* Now we create a user, on the left panel, under user's tab, create user, provide it an ID, email id and check the email verified option (keep it on).
* Under the credential tab set a password and remember the same for future.
*In case any error shows up on the keycloak panel, clear the browser cache.*

### Generating SSH certificate
After successfully creating a user on the keycloak, we need to now log into the stepca system. Here, we would be updating the ca.json file with the client secret that we copied earlier from the Credentials tab at line 53 of the file. 

Now, we need to configure step-ca to accept the your client. 

Run the command `step ca provisioner add keycloak --type=OIDC --client-id step-ca --client-secret [client secret] --configuration-endpoint https://keycloak.internal:8443/realms/step-ca/.well-known/openid-configuration --listen-address :10000`

Now start (or restart) your step-ca instance and run the following command:

`step ssh login user@example.com`

_Remember to use the same email address that was used for your keycloak user._

If all the steps were followed as mentioned then the server will provide a link that will allow you to access the interface. Along with it an SSH certificate will be issued and added to your SSH agent, with your username and email address as principals.

### Server keys and ssh service
On server machine open up a terminal and create ecdsa key pairs in the .step/certs directory using the command `step ssh certificate --host testhost ssh_host_ecdsa_key`
Provide the password to decrypt the provisioner key [tartans]
Key pair should have been generated

Once key pair has been generated strat the ssh service using `sudo service ssh restart`
You can check the status of the service using `sudo service ssh status`

### Single SSH Sign-On from clent to server
Now we move to the client system, where we have to download the root.crt using the python port running on step-ca system. To do the same run the following command `step certificate install root.crt` 

Let's configure the step-ca ssh using the command `step ssh config`.

Move to the server system, create a new user on the host. Your username must match the user portion of the email address you’ll use to sign in to keycloak. So the command will be `sudo adduser --quiet --disabled-password --gecos '' [user]`. The host side of our setup is done!


Login to the client system, we need to ssh into the server and to do so first move to the /.step/ssh directory using `cd ~/.step/ssh/`. We need to find the hostname to ssh into the sever by doing `step ssh hosts`. It will list all the hostnames. Now ssh into the server using the hostname found in the previous command by running `ssh [hostname]`. Upon successful execution of the command, we can now get a ssh connection into the server securely. 


### References
* Step CA Documentation: https://smallstep.com/docs/step-ca
* Step CA Tutorials : https://smallstep.com/docs/tutorials
* Small Step Blog for SSH DIY : https://smallstep.com/blog/diy-single-sign-on-for-ssh/ 
* Single Sign on for SSH : https://www.youtube.com/watch?v=ZhxLRlcNUM4
* KeyCloak home page : https://www.keycloak.org/
