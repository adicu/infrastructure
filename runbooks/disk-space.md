# What to do if we're out of disk space

## Diagnosis 
First, check that we're **really** out of disk space by running `df -h` on our
server. You should see something like:

```
$ df -h
Filesystem                 Size  Used Avail Use% Mounted on
udev                       487M  4.0K  487M   1% /dev
tmpfs                      100M  472K   99M   1% /run
/dev/disk/by-label/DOROOT   20G   15G  3.8G  80% /
none                       4.0K     0  4.0K   0% /sys/fs/cgroup
none                       5.0M     0  5.0M   0% /run/lock
none                       498M  1.2M  496M   1% /run/shm
none                       100M     0  100M   0% /run/user
none                        20G   15G  3.8G  80% /var/lib/docker/aufs/mnt/f4b3989b98627218c8ba47befbbc11b727f30099bcc153b0ddc75280eab7687f
shm                         64M     0   64M   0% /var/lib/docker/containers/c638381b01a35c135f90ffc0e759416604c8955035848d4584102b197080d27f/shm
```

Ignore the `/var/lib/docker/*` file systems. The one you care about is the `/`
file system. Make sure that there's a healthy amount of disk space left
(anything less than 200 MB is a problem, although you might want to clear things
out at 2 GB for safety).

## Clean up after Docker

Docker is the most likely thing hogging all of our disk space. To clean docker
up, run:

```
docker rmi $(docker images -a -q)
```

and 

```
docker rm $(docker ps -a -q)
```

Both commands should give you a bunch of `Error response from daemon` messages
complaining about either `dependent child images` or `cannot remove a running
container`. That's fine.

## Clean up Logs

If that doesn't clear up enough disk space, then the best option is probably
just to buy more from Digital Ocean. If you're **determined** to clear up more
space, though go ahead and check if we have any huge log files sucking up space:

```
$ du -hs /var/log
612M    /var/log

$ du -hs /srv/*/logs
198M    /srv/2016.devfe.st/logs
194M    /srv/2017.devfe.st/logs
247M    /srv/adi-website/logs
419M    /srv/density/logs
128M    /srv/devfe-st/logs
71M     /srv/learn/logs
```

If we do, go ahead and delete them (or compress them for archival storage). You
should restart any processes that are logging to those files (and thus have open
file handles) to actually release the disk space (if all else fails, rebooting
the server should also work).

## Buy more disk space

If all else fails, just buy more disk space from Digital Ocean.
