## Docker Registry with LDAP Authentication

##### Getting started

First, you need to clone this repo where the registry will be deployed:

```shell
$ git clone git@github.com:tormath1/docker-ldap-registry.git
$ cd docker-ldap-registry/
```

Please check that `docker` and `docker-compose` are installed on your server:

```shell
$ docker version
Client:
 Version:	18.02.0-ce
 API version:	1.36
 Go version:	go1.9.4
 Git commit:	fc4de447b5
 Built:	Tue Feb 13 15:28:01 2018
 OS/Arch:	linux/amd64
 Experimental:	false
 Orchestrator:	swarm

Server:
 Engine:
  Version:	18.02.0-ce
  API version:	1.36 (minimum version 1.12)
  Go version:	go1.9.4
  Git commit:	fc4de447b5
  Built:	Tue Feb 13 15:28:34 2018
  OS/Arch:	linux/amd64
  Experimental:	true
$ docker-compose version
docker-compose version 1.19.0, build unknown
docker-py version: 3.0.1
CPython version: 3.6.4
OpenSSL version: OpenSSL 1.1.0g  2 Nov 2017
```

You should have an output like this one. 

##### Add your SSL certificates

`httpd` is listening on `:443`, you'll need to provide your SSL certificates. 

Suppose that your registry domain is `registry.domain.tld`, you'll need to store your certificates into a `certs` folder:

```shell
$ mkdir certs
$ mv registry.domain.tld.key registry.domain.tld.crt certs/
```

##### Add your configuration

Please update `docker-compose.yaml` with your LDAP configuration: 

```shell
BIND_DN
BIND_PASSWORD
LDAP_URL
```

Your LDAP url must define your LDAP architecture. A quick Google search will show you how to write a good LDAP url. Cheers. 

You also need to update your `ServerName` in `conf/extra/httpd-vhosts.conf`

##### FIRE UP 

Thanks to docker, you only need a few commands in order to run your Docker registry:

```shell
$ docker-compose up -d
$ docker-compose ps
  Name                Command               State              Ports            
--------------------------------------------------------------------------------
httpd      httpd-foreground                 Up      0.0.0.0:443->443/tcp, 80/tcp
registry   /entrypoint.sh /etc/docker ...   Up      5000/tcp
```

You'll have `httpd` and `registry` logs with 

```shell
$ docker-compose logs httpd
$ docker-compose logs registry
```

If you want to have error log and access log from `httpd`:

```shell
$ docker-compose exec httpd tail -f logs/registry.local.tld-error_log
$ docker-compose exec httpd tail -f logs/registry.local.tld-access_log
```

##### Post installation

You can now connect your LDAP user to this registry:

```
$ docker login registry.domain.tld
username: <your-ldap-user> 
password:
```

You are able to pull images:

```
$ docker pull registry.domain.tld/star-wars:ep5-the-best
```

But you need to be in `pusher` LDAP group in order to push images. (Especially for CI like GitLab or Jenkins (lmao))

As registry admin, you'll have to backup registry volumes on persistent disk. Everything you need is in Docker official documentation.


##### Sources

[Apache Module mod_authnz_ldap](https://httpd.apache.org/docs/2.4/mod/mod_authnz_ldap.html)
[Docker Registry with Authentication](https://docs.docker.com/registry/recipes/apache/#setting-things-up)
