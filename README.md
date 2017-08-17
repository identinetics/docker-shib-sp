# Shibboleth SP docker image  

A Shibboleth configuration running apache httpd and shibd in a container:

1. Mapping users (apache and shibd) between host & container filesystems (also with multiple container instances)
2. The user-defined network uses static IP addresses.
3. Image is based on centos7
4. The image produces immutable containers, i.e. a container can be removed and re-created
any time without loss of data, because data is stored on mounted volumes.

## Prepare the repository

    git clone https://github.com/identinetics/docker-shib-sp.git
    cd docker-shib-sp
    git submodule update --init
    

## Build the docker image

    cp conf.sh.default conf.sh
    # edit conf.sh
    ./dscripts/build.sh 


## Configure and start the container
 
First, start the container with a shell. This will initialized the docker volumes

    ./dscripts/run.sh -ip bash 

The configure httpd and shibboleth (see next section). The default location for volumes is /var/lib/docker/volumes
When configuration is completed run /start.sh in the container, or restart the container:

    ./dscripts/run.sh     # daemon mode
    
## HTTPD Configuration 

- The default apache httpd configuration is created in /etc/httpd during docker build
- In conf/httpd.conf:
  -- Set the httpd user/group to that defined in conf*.sh
- If the apache httpd is not internet facing, setup the reverse proxy/load balancer etc.


## Shibboleth SP Configuration

There are several good guides available, such as at shibboleth.net and switch.ch

- The default shibd configuration is created in /etc/shibboleth during docker build
- Set the SHIBD_USER in /etc/shibboleth/shib-redhat to that defined in conf*.sh
- Check/modify the config files in /etc/shibboleth according to the documentation, inparticular:
  -- attribute-map.xml and attribute-policy.xml
  -- shibboleth2.xml
  -- new keys must be created manually (keygen.sh -> sp-cert.pem, sp-key.pem)
  -- metagen.sh, then edit and complete the SP metadata and submit it to the registry


### Duplicating/migrating a configuration

An existing configuration might be duplicated or migrated.

- copy conf.sh (or confxx.sh to confxy.sh on the same node)
- edit conf.sh and set the IMGID and PROJSHORT
- dscripts/build.sh (this will also create the docker volumes)
- the default path for the volumes is $DOCKER_VOLUME_ROOT/$CONTAINERNAME, here in short VOLROOT
- copy the contents of the docker volumes from the existing instance
- edit VOLROOT/.etc_httpd_conf/httpd.conf and correct the user
- set servername, docroot and logfile path in VOLROOT/.etc_httpd_conf.d/vhost.conf
- Adapt shibboleth2.xml, attribute-map.xml and attribute-policy.xml in VOLROOT.etc_shibboleth/ 
- create a new sp key and metadata:
 
    dscripts/run.sh -i  # start container in interactive mode
    cd /etc/shibboleth; ./keygen.sh -f
    chown shibd-user sp-*
    metagen.sh
    
- edit metadata and submit it to the metadata feed
- copy the metadata signing key to VOLROOT.etc_shibboleth/metadata_crt.pem (as defined in shibboleth2.xml)
- edit static contents in VOLROOT.var_www
- Besides the SP configuration the load balancer, TLS-certificate and DNS-name need to be configured