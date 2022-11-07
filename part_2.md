### Generating SSH certificate
After successfully creating a user on the keycloak, we need to now log into the stepca system. Here, we would be updating the ca.json file with the client secret that we copied earlier from the Credentials tab at line 53 of the file. 

Now, we need to configure step-ca to accept the your client. 

Run the command `step ca provisioner add keycloak --type=OIDC --client-id step-ca --client-secret [client secret] --configuration-endpoint https://keycloak.internal:8443/realms/step-ca/.well-known/openid-configuration --listen-address :10000`

Now start (or restart) your step-ca instance and run the following command:

`step ssh login user@example.com`

_Remember to use the same email address that was used for your keycloak user._

If all the steps were followed as mentioned then the server will provide a link that will allow you to access the interface. Along with it an SSH certificate will be issued and added to your SSH agent, with your username and email address as principals.

### Single Sign-On for SSH
Now we move to the client system, where we have to download the root.crt using the python port running on step-ca system. To do the same run the following command `step certificate install root.crt` 

Let's configure the step-ca ssh using the command `step ssh config`.

Move to the server system, create a new user on the host. Your username must match the user portion of the email address you’ll use to sign in to keycloak. So the command will be `sudo adduser --quiet --disabled-password --gecos '' [user]`. The host side of our setup is done!

Login to the client system, we need to ssh into the server and to do so first move to the /.step/ssh directory using `cd ~/.step/ssh/`. We need to find the hostname to ssh into the sever by doing `step ssh hosts`. It will list all the hostnames. Now ssh into the server using the hostname found in the previous command by running `ssh testhost`. Upon successful execution of the command, we can now get a ssh connection into the server securely. 
