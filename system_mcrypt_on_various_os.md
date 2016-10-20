This How-To will try to summarize various OS-Specific ways of installing the php-mcrypt extension.

## Scientific Linux 6.2
as reported by Daniel [here](https://pyd.io/f/topic/tip-mcrypt-installation-on-scientific-linux-6-2/#post-72655)

I thought I’d pass along a tip related to the MCrypt library that Pydio requires. Neither the **libmcryptpackage** nor the **php-mcrypt** one (which I presume are PHP bindings for libmcrypt) is part of the SL 6.2 distibution, and they’re also not available from SL 6.2′s pre-configured download sites.

However, those packages are available from [EPEL](https://fedoraproject.org/wiki/EPEL). To get them, first configure your machine for the EPEL download site; [per the EPEL wiki](https://fedoraproject.org/wiki/EPEL/FAQ#How_can_I_install_the_packages_from_the_EPEL_software_repository.3F):

`sudo rpm -Uvh http://download.fedoraproject.org/pub/epel/6/i386/epel-release-6-8.noarch.rpm`

Then, simply:

`sudo yum install php-mcrypt`

Yum will install php-mcrypt and libmcrypt as a dependency, and you should be good to go.

This also works for other EL-like distributions: CentOS / RHEL

## Debian / Ubuntu
`apt-get install php-mcrypt`