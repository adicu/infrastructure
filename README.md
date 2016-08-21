# websites-infra
Where to go when troubleshooting URLs &amp; Beyond. Please add/remove/edit knowledge

When updating any adi website configs, please try to commit those changes to the github so that we have a record of it somewhere & please try to update the documentation as necessary! Thanks to Dan for teaching me this stuff a while ago & I’m passing on the info in the best way I can!

Github: https://github.com/adicu/websites-infra


How our urls work
The current adi website, when you go to adicu.com/X or admin.adicu.com/X, if that page exists there as indicated by the most recently deployed code from the “adi-website” repo in Github to our server in the directory “/srv/adi-website/www”. If you want to see the various routes (different links) that ADI’s website has, they are defined initially in Flask here. You will see lines of code that look like this:
@client.route('/events/devfest', methods=['GET'])
@client.route('/contact', methods=['GET'])

These lines tell our website’s app what to do once a client lands on that page. Usually it returns or shows the html file, and sometimes it adds extra data parameters. Other times, it redirects the page to somewhere else.
return render_template('foundry.html')
return render_template('jobfair.html', companies=companies)
return redirect("http://devfe.st")

However, when there isn’t a clear route to go to, for example if someone types in “labs.adicu.com” or “readme.adicu.com”, our flask route handler isn’t sufficient to handle those requests. This becomes the job of our Nginx server, where our flask app is currently running all the time. We have specific rules that we write to utilize redirects here.

You need ssh access in order to be able to access our server/be able to deploy live updates to the adi website/update any URL routing. Basically, the server needs to know that you and your computer are authorized users that can see and manipulate all the data that exists on our server.

How to add people onto the server
For the person who already has access and wants to add others:
Cd ~/root/.ssh
Cat authorized_keys (check who already exists & save it to a TEMP txt file while you’re adding a new person)
The person who wants to be added needs to go to their ssh directory and must copy their local public identity file which will be added to the list of authorized users.

For the person who wants access and wants to be added
Two options
1. Using the computer you want access from:
`cat ~/.ssh/id_rsa.pub`  & securely send this to the person adding you to the server
2. If you are struggling with copy paste/want an easier command, use this command instead
`echo ~/.ssh/id_rsa.pub | pbcopy` (auto copies it to your clipboard) & ctrl+V that in an email/chat to the person that’s adding you.

How to ssh into the website. 
You can ssh into the host’s ip, which the command would be `ssh root@162.243.116.41`

An easier way would be go create or edit a config file, which gives an alias to that login to a server, so you don’t always have to remember the server name. Once you have a file that has all the necessary information, you can instead do `ssh adi-website`.

Edit or create a file here: `~/.ssh/config`

Add this line to that file:
```
Host adi-website
  HostName 162.243.116.41
  User root
  IdentityFile ~/.ssh/id_rsa
```

A snippet of how my file looks like. I use this config file for all the external machines I have access to.

```
Host cunix
  HostName cunix.columbia.edu
  User eek2138
  IdentityFile ~/.ssh/id_rsa

Host clic
  HostName clic.cs.columbia.edu
  User eek2138
  IdentityFile ~/.ssh/id_rsa

Host adi-data
  HostName 104.131.202.10
  User root
  IdentityFile ~/.ssh/id_rsa

Host adi-courses
  HostName 192.241.189.115
  User ubuntu
  IdentityFile ~/.ssh/id_rsa

Host adi-density
  HostName 104.131.178.179
  User root
  IdentityFile ~/.ssh/id_rsa

Host db
  HostName 40.121.62.121
  User eunicekokor
  IdentityFile ~/.ssh/id_rsa

Host eunicode
  HostName 192.241.220.88
  User root
  IdentityFile ~/.ssh/id_rsa
```


Who has access right now to add you to the server & do the above steps 
(I just typed in `cat autorized_keys` and looked at the names)
Sam W, Nate B, Dan S, Raymond X, Alan D, Dina L, Eunice K, Brian Z, Matt P, Lesley C

Ok, now how do we figure out URL handling
ADI “sites-enabled” aka URL Handling Mechanism Script
After you ssh into the website
`cat ~../etc/nginx/sites-enabled/adi-website` to see what the rules are on the server. Edit `sites-enabled` & make changes & save using vim/emacs/idk what else people use these days.
I posted the contents of our current rewrite script to `sites-enabled-config` file on github. (It would be cool if we could start version controlling this, so that we have a changelist of these rules, but it’s not necessary at the moment.)

Most important final steps
Making sure those changes are enacted & you didn’t break everything
Save those changes
`sudo nginx -t`
Note: If the code you wrote in `sites-enabled` doesn’t compile, it will tell you after this step

Restart the Server with those updated changes
`sudo service nginx restart`
Note: the websites will be down during this period, but should come up within a few minutes. If not something went wrong and you need to debug using the logs to see what went wrong.


We should try for only a few people at a time to actually change nginx URL rules. All of ADI’s websites could go down if something goes wrong here and it’s easier to debug to know who made a change and when. Slack the #adi-website channel if you want to add something, either if you want to do it yourself or work with someone else to debug a problem.


Debugging & Learning

Use the Logs on the Server
Where Logs are located: `/var/log/nginx`
In simple terms
Access logs deal with anyone accessing the routes of an adicu.com url
Error logs deal with any errors related to accessing the routes of an adicu.com url

The latest logs are in the access.log file & error.log file. Anything older is stored in overflow “*.log.#” files. The oldest should be the larger numbers and the most recent should be the smallest numbers.
A useful command in debugging are the commands
`tail -f access.log` and `tail -f error.log`
This lets you see
Learning about Nginx Rules and Beyond
A tutorial to help you with nginx
