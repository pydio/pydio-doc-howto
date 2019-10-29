SELinux is a linux kernel module security that provides the mechanism for enforcing the security in access control management of the system. As we can see, linux has applied Discret Access Control (DAC) model since it was in conception form. In DAC, each object (file, folder …) has attached attribuites to indicate the ownership and permissions. SELinux, howerver, can provide an other method which is more powerful and flexible in controlling access to system resource. SELinux base on several Mandatory Access Control (MAC) method such as: Type-Domain, Multi-levels Multi-Categories, RBAC. And the combination of such methods forme the context in SELinux, which can make a lot of trouble and difficulties to typical system administrators.

For further information: [selinux](https://www.ibm.com/developerworks/library/l-selinux/)

To configure SELinux, we firstly and pricipally define the domains and the policy, then label objects (files, folders, devices, process …) to the type which is coressponded to domain name.

In this article, we adopt to how to configure SELinux (set selinux context) for pydio under the control of selinux. In Targeted Policy in CentOS, all domain and policy have already been defined and we just label our objects (files, folders, processes) to suitable type.

For further information: [Security-Enhanced_Linux](https://access.redhat.com/site/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Security-Enhanced_Linux/index.html)

### Configuration of SELinux for Pydio
Prerequisites:
OS: CentOS 7
Policy: targeted version 28
Mode: enforcing
Package required: policycoreutils-python

After installation pydio by using rpm, Pydio will be located in several repositories:

+ /var/cache/pydio/ :for php cache
+ /var/lib/pydio/ :for users’ files, data upload ..
+ /etc/pydio :for configurations
+ /usr/share/pydio :php files

To ensure that pydio works normally under the controls of SELinux, the following configuration should be labeled with appropriate type:

    # relabel whole system
    touch /.autorelabel
    reboot

    # label folder /var/cache/pydio (and its sub-folder) with a TYPE: httpd_cache_t
    # command: semanage fcontext change the context of object (file/folder) and all setting can resist after next relabel
    # you can use chcon (change context) for a session
    semanage fcontext -a -t httpd_cache_t "/var/cache/pydio(/.*)?" 

    # httpd_sys_rw_content_t: read/write content
    semanage fcontext -a -t httpd_sys_rw_content_t "/var/lib/pydio(/.*)?" 

    # httpd_sys_content_t: read only content
    semanage fcontext -a -t httpd_sys_content_t "/etc/pydio(/.*)?"

    # httpd_sys_script_exec_t: script 
    semanage fcontext -a -t httpd_sys_script_exec_t "/usr/share/pydio(/.*)?" 

    # Active the change on objects destination
    restorecon -Rv /var/lib/pydio
    restorecon -Rv /etc/pydio
    restorecon -Rv /usr/share/pydio
    restorecon -Rv /var/cache/pydio

#### sebool for httpd: 

SELinux boolean is a solution that provides a way to change the security policy without re-compile whole security policy. Depend on your requirements, variables of boolean shoule be turn on or off. Parametre -P means permenant.

    # turn httpd_unified off. httpd_unified enable httpd_t read/write on httpd_sys_xxx type. It means that there is no difference between httpd_sys_script_exec_t and with http_sys_content_t. Of course, for more security we turn it off and regroupe pydio data to appropriate groups which were labeled in previous step.
    setsebool -P httpd_unified off

    # If you use ssl, and apache will prompt you to input password for certificate/key. Otherwise, turn it off.
    # Normally httpd do not need to communicate with terminal device, but in some case such as input password to read a key file and certification, user must type password from keyboard.
    setsebool -P httpd_tty_comm on 

    # Depend on you use sendmail service or not. In Pydio, we need sendmail function if we active the alert function. 
    # Note: In case of SELinux is disable, this boolean also must be turned on for activation sendmail function (I don't understand why)
    setsebool -P httpd_can_sendmail off 

    # for execution php script
    setsebool -P httpd_enable_cgi on  
    setsebool -P httpd_builtin_scripting on

    # in case of using database on other machine
    setsebool -P httpd_can_network_connect_db on

    # in case of using NFS service for file sharing
    setsebool -P httpd_use_nfs on

    #(or setsebool -P httpd_use_cifs on)

#### Permission for other repositories (pydio users’ data) in the system

    # example path: /path1/path2/pydiodata
    # DAC permission
     chown -R apache:apache /path1/path2/pydiodata
     chmod -R 0755 /path1/path2/pydiodata

    # SELinux permission
     semanage fcontext -a -t httpd_sys_rw_content_t "/path1/path2/pydiodata(/.*)?" 
     restorecon -Rv  /path1/path2/pydiodata
     
#### Specific permision for booster.

When you install/update pydio booster, executable file will be downloaded and saved in **AJXP_DATA_PATH/plugins/helper.booster/pydio** (or /var/lib/pydio/plugins/helper.booster/pydio).

Turn on se bool

    setsebool -P httpd_ssi_exec 1

Set context for pydio booster file

    semanage fcontext -a -t httpd_sys_script_exec_t /var/lib/pydio/plugins/helper.booster/pydio
    restorecon /var/lib/pydio/plugins/helper.booster/pydio
    
#### Install selinux module to allow httpd open service on unreserved port.

Create file enablePydioBooster.te with contents:

    module enablePydioBooster 1.0;

    require {
    	type sysctl_net_t;
    	type httpd_sys_script_t;
    	type unreserved_port_t;
    	class tcp_socket name_bind;
    	class dir search;
    	class file { read open };
    }

    allow httpd_sys_script_t sysctl_net_t:dir search;
    allow httpd_sys_script_t sysctl_net_t:file { read open };

    allow httpd_sys_script_t unreserved_port_t:tcp_socket name_bind;
    
Check syntax 
    
    checkmodule -M -m -o enablePydioBooster.mod enablePydioBooster.te
    
Compile this file

    semodule_package -m enablePydioBooster.mod -o enablePydioBooster.pp

Import new module to SELinux

    semodule -i enablePydioBooster.pp

Add module to SELinux "Targeted" policy and automatically load it on future boots
    
    semodule -s targeted -i enablePydioBooster.pp
 

> Note: To remove, use `semodule -r enablePydioBooster`
