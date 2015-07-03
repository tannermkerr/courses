#Deployment of Nginx Using Salt Formulas (Part 3 of 3)

Nginx (pronounced “engine x”) is a simple web server that we will set up on our Salt minion. We will accomplish this using Salt states. Salt allows you to define a top.sls file that holds the instructions for what packages (such as nginx) will be installed on the minion.

##Configure the nginx.sls File 

Make sure you are connected to your Salt master. You should see a prompt similar to the following:

```
root@salt-master:~#
```

We’ll start by creating our nginx.sls file. The naming of this file is arbitrary: we only need to have it match our top.sls file later on. 

```
mkdir /srv/salt/
vim /srv/salt/nginx.sls
```

The nginx.sls file is what we define as a “state”. Within that state is the instructions we want it to include. We define these parameters within the file:

```
nginx:
  pkg:
    - installed
  service:
    - running
    - require:
      - pkg: nginx
/usr/share/nginx/html/index.html:
  file:
    - managed
    - source: salt://nginx/index.html
    - require:
      - pkg: nginx
```


Let’s break this down line by line. The first three lines specify that the nginx package should be installed. Simple enough. Next, we say that the service should be running. However, before the service is started, we require that the nginx package be installed. Next, we replace the file located at /usr/share/nginx/html/index.html on the minion with the file salt://nginx/index.html located on the master (we will create this file later). Again, this file replacement requires that the nginx package is installed first.

##Create Your HTML File

Now we need to make our custom html file. 

```
vim /srv/salt/nginx/index.html
```

If you desire, you can change the following html to anything you would like on your webserver!

```html
<html>
<head><title>Salt rocks!</title></head>
<body>
<h1>This file brought to you by Salt</h1>
</body>
</html>
```

##Configure top.sls File

Now that we have created our state and html file, we need a way for the master to determine what states it should apply to each minion. To do this, we will create a top.sls file. Salt looks to find this file in the /srv/salt/ directory.

```
vim /srv/salt/top.sls
```

Write this into the file:

```
base:
  ‘minion1':
    - nginx 
  ‘minion22’:
    - not-applied
```

For this example, we have a default base environment. When the minion is told to execute a highstate, it looks through this file to see what it should apply. Assuming we created a minion named “minion1”, the minion will apply the “nginx” state. Because the minion does not match “minion22”, it will not apply “not-applied” (this could be any other package or combination of packages).

##Apply the nginx State

Now apply this state by executing the following command on the salt master:

```
salt 'minion1' state.highstate
```

This applies the top.sls file to minion1. You should see a success message such as the following:

```
Summary
------------
Succeeded: 3 (changed=2)
Failed:	0
------------
Total states run: 	3
```

Check that everything worked as expected by going to the ip address of your minion in your browser (the minion ip can be found in the DigitalOcean control panel).



##Resources:

https://www.digitalocean.com/community/tutorials/how-to-use-the-digitalocean-api-v2
https://developers.digitalocean.com/documentation/v2
https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-14-04
https://www.digitalocean.com/community/tutorials/how-to-configure-ssh-key-based-authentication-on-a-freebsd-server
https://www.digitalocean.com/community/tutorials/how-to-install-salt-on-ubuntu-12-04
https://www.digitalocean.com/community/tutorials/automated-provisioning-of-digitalocean-cloud-servers-with-salt-cloud-on-ubuntu-12-04
https://www.digitalocean.com/community/tutorials/how-to-create-your-first-salt-formula
http://bencane.com/2013/09/03/getting-started-with-saltstack-by-example-automatically-installing-nginx/
http://ingloriousdevops.com/2014/10/14/do-you-want-more-flavor-add-more-salt/