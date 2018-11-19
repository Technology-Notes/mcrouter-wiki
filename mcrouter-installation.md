## Automatic installation for Ubuntu 16.04

To simplify dependency installation, we provided an [auto-install script](https://github.com/facebook/mcrouter/blob/master/mcrouter/scripts/install_ubuntu_16.04.sh) that's been tested on Ubuntu 16.04.

The script might also be useful for other systems - make sure to read through its source files. It contains a bunch of ["recipe" files](https://github.com/facebook/mcrouter/tree/master/mcrouter/scripts/recipes) to download and install each mcrouter dependency, including workaround for common pain points.

After you've read through the script and made sure you're fine with the commands it will run, invoke with absolute dir for self-contained install and any arguments to `make`:
```
$ git clone git@github.com:facebook/mcrouter.git
$ ./mcrouter/mcrouter/scripts/install_ubuntu_16.04.sh /home/$USER/mcrouter-install/ -j4
[sudo] password for ...:
...
$ ~/mcrouter-install/install/bin/mcrouter --help
```

## Docker

### Official Docker

We provide this [Dockerfile](https://github.com/facebook/mcrouter/blob/master/mcrouter/scripts/docker/Dockerfile) for you to build a docker image (base on ubuntu 14:04).

### Third party Dockers

* https://registry.hub.docker.com/u/jamescarr/mcrouter/
* https://registry.hub.docker.com/u/jamescarr/mcrouter-rest-api/