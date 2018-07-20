# Mô hình

# Cấu hình
## Client
Trên client cần có gói vccloud_check  
Cài nagios nrpe server
```
apt-get install nagios-nrpe-server
```
Allow satellite được kết nối trong file `/etc/nagios/nrpe.cfg` và cấu hình sử dụng `vccloud_check`
```
allowed_hosts=127.0.0.1,10.5.8.61
...
bla bla
```
Restart services
```
/etc/init.d/nagios-nrpe-server restart
```
## Satellite
Add icinga repo
```
wget -O - https://packages.icinga.com/icinga.key | apt-key add -
echo 'deb https://packages.icinga.com/ubuntu icinga-xenial main' >/etc/apt/sources.list.d/icinga.list
apt-get update
```
Cài đặt icinga2
```
apt-get install icinga2
```
Cấu hình syntax highlight trong vim
```
apt-get install vim-icinga2 vim-addon-manager
vim-addon-manager -w install icinga2
```
Cài nagios nrpe
```
apt-get --no-install-recommends install nagios-nrpe-plugin
```
Disable icinga sử dụng lệnh trong thư mục `conf.d` bằng cách comment trong file `/etc/icinga2/icinga2.conf`
```
#include_recursive "conf.d"
```
Restart icinga2
```
service icinga2 restart
```

## Master
Cài icinga2 & icinga web giống trong file [này](Install.md)  
Setup master 
```
root@m-1:~# icinga2 node wizard
Welcome to the Icinga 2 Setup Wizard!

We will guide you through all required configuration details.

Please specify if this is a satellite/client setup ('n' installs a master setup) [Y/n]: n

Starting the Master setup routine...

Please specify the common name (CN) [m-1]: 
Reconfiguring Icinga...
Checking for existing certificates for common name 'm-1'...
Certificates not yet generated. Running 'api setup' now.
Generating master configuration for Icinga 2.
Enabling feature api. Make sure to restart Icinga 2 for these changes to take effect.
Please specify the API bind host/port (optional):
Bind Host []: 
Bind Port []: 

Done.

Now restart your Icinga 2 daemon to finish the installation!
```
Restart icinga2
```
service icinga2 restart
```
Tạo ticket cho s-1 và s-2
```
root@m-1:~# icinga2 pki ticket --cn 's-1'
34b5f0968417add56d31026720453d085d040066
root@m-1:~# icinga2 pki ticket --cn 's-2'
f9935d9ffd380cdaf5b78cb7cbbbf659e153e356
```
### Satellite setup
```
root@s-1:/etc/icinga2# icinga2 node wizard
Welcome to the Icinga 2 Setup Wizard!

We will guide you through all required configuration details.

Please specify if this is a satellite/client setup ('n' installs a master setup) [Y/n]: 

Starting the Client/Satellite setup routine...

Please specify the common name (CN) [s-1]: 

Please specify the parent endpoint(s) (master or satellite) where this node should connect to:
Master/Satellite Common Name (CN from your master/satellite node): m-1

Do you want to establish a connection to the parent node from this node? [Y/n]: 
Please specify the master/satellite connection information:
Master/Satellite endpoint host (IP address or FQDN): 10.5.8.232
Master/Satellite endpoint port [5665]: 

Add more master/satellite endpoints? [y/N]: 
Parent certificate information:

 Subject:     CN = m-1
 Issuer:      CN = Icinga CA
 Valid From:  Jan 22 04:41:07 2018 GMT
 Valid Until: Jan 18 04:41:07 2033 GMT
 Fingerprint: 20 2F 37 98 3B 20 CF 95 21 75 FE 1E 24 F0 8A 55 AA 86 32 49 

Is this information correct? [y/N]: y

Please specify the request ticket generated on your Icinga 2 master (optional).
 (Hint: # icinga2 pki ticket --cn 's-1'): 34b5f0968417add56d31026720453d085d040066

Please specify the API bind host/port (optional):
Bind Host []: 
Bind Port []: 

Accept config from parent node? [y/N]: y
Accept commands from parent node? [y/N]: y

Reconfiguring Icinga...
Disabling feature notification. Make sure to restart Icinga 2 for these changes to take effect.
Enabling feature api. Make sure to restart Icinga 2 for these changes to take effect.

Done.

Now restart your Icinga 2 daemon to finish the installation!
```
Restart icinga2
```
service icinga2 restart
```
```
root@s-2:~# icinga2 node wizard
Welcome to the Icinga 2 Setup Wizard!

We will guide you through all required configuration details.

Please specify if this is a satellite/client setup ('n' installs a master setup) [Y/n]: 

Starting the Client/Satellite setup routine...

Please specify the common name (CN) [s-2]: 

Please specify the parent endpoint(s) (master or satellite) where this node should connect to:
Master/Satellite Common Name (CN from your master/satellite node): m-1

Do you want to establish a connection to the parent node from this node? [Y/n]: 
Please specify the master/satellite connection information:
Master/Satellite endpoint host (IP address or FQDN): 10.5.8.232
Master/Satellite endpoint port [5665]: 

Add more master/satellite endpoints? [y/N]: 
Parent certificate information:

 Subject:     CN = m-1
 Issuer:      CN = Icinga CA
 Valid From:  Jan 22 04:41:07 2018 GMT
 Valid Until: Jan 18 04:41:07 2033 GMT
 Fingerprint: 20 2F 37 98 3B 20 CF 95 21 75 FE 1E 24 F0 8A 55 AA 86 32 49 

Is this information correct? [y/N]: y

Please specify the request ticket generated on your Icinga 2 master (optional).
 (Hint: # icinga2 pki ticket --cn 's-2'): f9935d9ffd380cdaf5b78cb7cbbbf659e153e356

Please specify the API bind host/port (optional):
Bind Host []: 
Bind Port []: 

Accept config from parent node? [y/N]: y
Accept commands from parent node? [y/N]: y

Reconfiguring Icinga...
Disabling feature notification. Make sure to restart Icinga 2 for these changes to take effect.
Enabling feature api. Make sure to restart Icinga 2 for these changes to take effect.

Done.

Now restart your Icinga 2 daemon to finish the installation!
```
Restart icinga2
```
service icinga2 restart
```

### Trên Master
Disable icinga sử dụng lệnh trong thư mục `conf.d` bằng cách comment trong file `/etc/icinga2/icinga2.conf`
```
#include_recursive "conf.d"
```
Cấu hình zone và endpoint trong file `/etc/icinga2/zones.conf`
```
object Endpoint "s-1" {
        host = "10.5.8.61"
}

object Zone "s-1" {
        endpoints = [ "s-1" ]
        parent = "m-1"
}

object Endpoint "s-2" {
        host = "10.5.8.235"
}

object Zone "s-2" {
        endpoints = [ "s-2" ]
        parent = "m-1"
}
object Endpoint "m-1" {
        host = "10.5.8.232"
}

object Zone "m-1" {
        endpoints = [ "m-1" ]
}

object Zone "global-templates" {
        global = true
}

object Zone "director-global" {
        global = true
}
```
Vì sử dụng file check riêng nên trong file `/etc/icinga2/constants.conf` thay đổi cấu hình file check
```
const PluginDir = "/opt/vccloud_check"
```
Trong thư mục `zones.d` lấy file global-templates về và tạo thư mục cho s-1 và s-2
```
mkdir /etc/icinga2/zones.d/s-1
mkdir /etc/icinga2/zones.d/s-2
```
Trong thư mục `s-1` tạo file host và services với nội dung như sau
```
root@m-1:/etc/icinga2/zones.d/s-1# vim hosts.conf
object Host "c-1" {
      import "generic-host"
      address = "10.5.8.154"
      vars.notification["mail-icingaadmin"] = {
        groups = [ "admin" ]
      }
}
root@m-1:/etc/icinga2/zones.d/s-1# vim services.conf
apply Service "check_server_load" {
  import "generic-service"
  check_command = "check_via_nrpe"
  vars.ARG1 = "check_server_load"
  assign where host.name == "c-1"
}

apply Service "check_cpu" {
  import "generic-service"
  check_command = "check_via_nrpe"
  vars.ARG1 = "check_cpu"
  assign where host.name == "c-1"
}

apply Service "check_ram" {
  import "generic-service"
  check_command = "check_via_nrpe"
  vars.ARG1 = "check_ram"
  assign where host.name == "c-1"
}

apply Service "check_rootdisk" {
  import "generic-service"
  check_command = "check_via_nrpe"
  vars.ARG1 = "check_rootdisk"
  assign where host.name == "c-1"
}

```
