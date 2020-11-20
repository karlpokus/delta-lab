# delta-lab
Fiddling with efficient delta updates for docker image tar archives. Let's also compare network transfer-, and storage size between different tools, both full-, and delta updates.

# casync

preparations

```bash
$ mkdir -p local remote docker
# prepare some images
$ docker save ubuntu:18.04 > docker/ubuntu-18.tar
$ docker save ubuntu:20.04 > docker/ubuntu-20.tar
# run vm
$ vagrant up && vagrant ssh
$ cd /vagrant
# build simulated remote indexes and chunk stores
$ casync --store=remote/ubuntu-18.castr make remote/ubuntu-18.caibx docker/ubuntu-18.tar
$ casync --store=remote/ubuntu-20.castr make remote/ubuntu-20.caibx docker/ubuntu-20.tar
```

### local

full update

```bash
# download ubuntu 18
$ casync -v --store=remote/ubuntu-18.castr extract remote/ubuntu-18.caibx local/ubuntu-18.tar
Zero bytes written as sparse files: 3.9M
Bytes cloned through reflinks: 0B
Chunk requests fulfilled from local store: 899
Bytes used from local store: 62.6M
Chunk requests fulfilled from local seed: 0
Bytes used from local seed: 0B
Chunk requests fulfilled from remote store: 0
Bytes used from remote store: 0B
# validate
$ diff docker/ubuntu-18.tar local/ubuntu-18.tar
```

delta update

```bash
# download ubuntu 20
$ casync -v --store=remote/ubuntu-20.castr extract remote/ubuntu-20.caibx --seed=local/ubuntu-18.tar local/ubuntu-20.tar
Zero bytes written as sparse files: 7.2M
Bytes cloned through reflinks: 0B
Chunk requests fulfilled from local store: 1015
Bytes used from local store: 69.2M
Chunk requests fulfilled from local seed: 49
Bytes used from local seed: 2.5M # ~3 %
Chunk requests fulfilled from remote store: 0
Bytes used from remote store: 0B
# validate
$ diff docker/ubuntu-20.tar local/ubuntu-20.tar
```

### remote

delta update

```bash
$ vagrant up
$ vagrant ssh client|server
# gen keys to /vagrant/ssh/casync on client
$ ssh-keygen -t rsa -b 4096 -C casync
# add key on server
$ cat ssh/casync.pub >> /home/vagrant/.ssh/authorized_keys
# add key to ssh agent on client
$ eval `ssh-agent -s`
$ ssh-add ssh/casync
# download ubuntu 20 via ssh
$ casync -v --store=vagrant@<ip>:/vagrant/remote/ubuntu-20.castr \
extract vagrant@<ip>:/vagrant/remote/ubuntu-20.caibx --seed=local/ubuntu-18.tar local/ubuntu-20.tar
Zero bytes written as sparse files: 7.2M
Bytes cloned through reflinks: 0B
Chunk requests fulfilled from local store: 0
Bytes used from local store: 0B
Chunk requests fulfilled from local seed: 49
Bytes used from local seed: 2.5M # still only ~3% :)
Chunk requests fulfilled from remote store: 1015
Bytes used from remote store: 69.2M
```

# todos
- [x] create chunked store and index
- [x] reconstruct blob
- [x] tar.gzip docker images
- [x] avoid gzip for delta updates
- [x] full update
- [x] delta update
- [ ] use s3 + cloudfront to serve index and chunk store since casync is cdn friendly
- [x] s3 presigned won't work for a chunk store
- [x] seed flag position
- [ ] try casync digest <dir>
- [ ] switch caidx for caibx
- [ ] try chunk-size flag
- [ ] try multiple stores flag --extra-store=PATH
- [ ] multiple seeds
- [ ] resume download
- [x] casync ssh

# license
MIT
