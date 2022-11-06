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

#### *Run Grading Script* 

### Start the server
* You may start the server now using the command `step-ca ca.json`
* Enter the requirred detials.

_*Check the output, the server must have started running on the provided IP:port*_



### References
* Step CA Documentation: https://smallstep.com/docs/step-ca
* Step CA Tutorials : https://smallstep.com/docs/tutorials
* Small Step Blog for SSH DIY : https://smallstep.com/blog/diy-single-sign-on-for-ssh/ 
* Single Sign on for SSH : https://www.youtube.com/watch?v=ZhxLRlcNUM4
* KeyCloak home page : https://www.keycloak.org/
