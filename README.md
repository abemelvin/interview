# Interview - Prompt 2

## 1. Getting started

This solution requires the Vagrant binary (https://www.vagrantup.com), and utilizes Virtualbox as the underlying hypervisor (https://www.virtualbox.org).

After cloning the repository to your local machine, initializing the environment is simple. Run the command `vagrant up` from the same directory as the `Vagrantfile`. The automated setup sequence will begin; this initial process may take 5-10 minutes depending on your machine's resources.

## 2. Logging in

Once your local command prompt returns, the automated setup is complete. You can now log in to the environment by running the command `vagrant ssh` from the same directory as the `Vagrantfile`. You should receive a prompt like this:

```
vagrant@vagrant-ubuntu-trusty-64:~$
```

Within the `/home/vagrant/` directory there should be an already cloned `admin` repository, for ease of testing this solution. All of the services specified in the prompt (SSL-enabled web server, GitLab web UI, and GitLab SSH) are running on their respective ports (443, 4443, and 22). 
For ease of use, port 443 has been forwarded to your local machine's port 1443, and port 4443 has been forwarded to your local machine's port 4443. This way, you can view the SSL-enabled web server by using your local browser and visiting `https://localhost:1443` and the GitLab web UI similarly by visiting `https://localhost:4443`.

![GitLab-UI](https://github.com/abemelvin/interview/blob/master/doc/img/gitlab.png?raw=true)

![Web-UI](https://github.com/abemelvin/interview/blob/master/doc/img/web.png?raw=true)

## 3. Verifying functionality

To demonstrate that this environment works, change your present working directory to `/home/vagrant/admin/`. As per the prompt's example, let's create a file named `ps.sh`:

```
echo '#!/bin/bash' >> ps.sh
echo 'ps aux' >> ps.sh
```

Now, let's add this script to the repository and push it:

```
git add ps.sh
git commit -m "test"
git push
```

We can verify that the output of our script is being hosted on our web server:

```
curl -k https://localhost:443
```

Or, we can simply browse to `https://localhost:1443` using our local machine's browser.

![Push-Results](https://github.com/abemelvin/interview/blob/master/doc/img/results.png?raw=true)


