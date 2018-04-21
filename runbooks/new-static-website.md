# Deploying a new static website

Let's say you want to deploy a new website at "new.adicu.com"

## Setup Nginx

Go to `/etc/nginx/sites-available` and create a new file
`/etc/nginx/sites-available/new` with:

```nginx
server {
	listen 80;
	server_name new.adicu.com;
	root /srv/new/public_html/;

	access_log /srv/new/logs/access.log;
	error_log /srv/new/logs/error.log;
}
```

## Setup the HTML files

Create the appropriate directories:

```
mkdir /srv/new
mkdir /srv/new/logs
mkdir /srv/new/public_html/
```

and then copy the HTML files into `/srv/new/public_html/`
([rsync](https://en.wikipedia.org/wiki/Rsync) is your friend here).

Make sure to run:

```
sudo chown -R git:webmaster /srv/new
```

to make sure that the directories have the right permissions.

## Deploy

Now run

```
sudo service nginx restart
```

to restart nginx. This will reload `nginx` with the new configuration --
if there are any errors in the nginx configuration, go back and fix
those.

You should now be able to go to `http://new.adicu.com` and see your
website!

## Encryption

(NOTE: Once Let's Encrypt releases wildcard certificates, this won't be
necessary anymore).

Just run

```
certbot-auto
```

and follow the instructions (please *turn off* `http` support and
force redirect to `https`). `sudo service nginx restart` to test the
changes.

When you're done, sync your changes to `/etc/nginx/` into this directory
(in `files/nginx`).

## Setting-up autodeploy

We use [Travis-CI](https://travis-ci.org/) for our auto-deployment.
Create a `.travis.yml` file in the GitHub repository (use the official
[docs](https://docs.travis-ci.com/) for help) and setup tests.

To make sure Travis has access to our server, you'll need to setup SSH
keys. On your *local* laptop, run

```
ssh-keygen -t rsa -b 4096 -C 'new@travis-ci.org' -f ./deploy_rsa
```

Now, go on our server, run:

```
sudo su travis
cd /home/travis/
vim .ssh/authorized_keys
```

and copy-and-paste the **public key** `(`deploy_rsa.pub`) into that
file.

Once you've done that, follow the instructions
[here](https://docs.travis-ci.com/user/encryption-keys/) to encrypt the
**private key**  and check the encrypted key into the repository.

Now, add the following section into your `.travis.yml`:

```yaml
before_deploy:
- test $TRAVIS_BRANCH == "master"
- openssl aes-256-cbc -K $encrypted_60e6b55cdced_key -iv $encrypted_60e6b55cdced_iv
  -in $TRAVIS_BUILD_DIR/.ci/deploy_rsa.enc -out $TRAVIS_BUILD_DIR/.ci/deploy_rsa -d
- eval "$(ssh-agent -s)"
- chmod 600 $TRAVIS_BUILD_DIR/.ci/deploy_rsa
- ssh-add $TRAVIS_BUILD_DIR/.ci/deploy_rsa

deploy:
  provider: script
  skip_cleanup: true
  script: rsync --recursive --delete-after $TRAVIS_BUILD_DIR/<files> travis@adicu.com:/srv/new/public_html
```

Now, Travis should automatically deploy with every passing build on
master!
