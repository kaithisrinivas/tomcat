# tomcat Cookbook
[![Build Status](https://travis-ci.org/chef-cookbooks/tomcat.svg?branch=master)](https://travis-ci.org/chef-cookbooks/tomcat) [![Cookbook Version](https://img.shields.io/cookbook/v/tomcat.svg)](https://supermarket.chef.io/cookbooks/tomcat)

Installs and configures Tomcat, Java servlet engine and webserver version 6 and 7 (8 not yet supported).

## Requirements
### Platforms
- Debian / Ubuntu derivatives
- RHEL derivatives
- Fedora

### Chef
- Chef 12.1+

### Cookbooks
- java
- openssl
- yum-epel

## Attributes
- `node['tomcat']['base_version']` - The version of tomcat to install, default `6`.
- `node['tomcat']['port']` - The network port used by Tomcat's HTTP connector, default `8080`.
- `node['tomcat']['proxy_port']` - if set, the network port used by Tomcat's Proxy HTTP connector, default nil.
- `node['tomcat']['proxy_name']` - if set, the proxy name used by Tomcat's Proxy HTTP connector, default nil.
- `node['tomcat']['ssl_port']` - The network port used by Tomcat's SSL HTTP connector, default `8443`.
- `node['tomcat']['ssl_proxy_port']` - if set, the network port used by Tomcat's Proxy SSL HTTP connector, default nil.
- `node['tomcat']['ajp_port']` - The network port used by Tomcat's AJP connector, default `8009`.
- `node['tomcat']['ajp_redirect_port']` - The network port redirected to by Tomcat's AJP connector, default `ssl_port`.
- `node['tomcat']['ajp_listen_ip']` - If set, the network address used by Tomcat's AJP connector, default nil.
- `node['tomcat']['shutdown_port']` - The network port used by Tomcat to listen for shutdown requests, default `8005`.
- `node['tomcat']['catalina_options']` - Extra options to pass to the JVM only during start and run commands, default "".
- `node['tomcat']['java_options']` - Extra options to pass to the JVM, default `-Xmx128M -Djava.awt.headless=true -XX:+UseConcMarkSweepGC`.
- `node['tomcat']['use_security_manager']` - Run Tomcat under the Java Security Manager, default `false`.
- `node['tomcat']['loglevel']` - Level for default Tomcat's logs, default `INFO`.
- `node['tomcat']['deploy_manager_apps']` - whether to deploy manager apps, default `true`.
- `node['tomcat']['authbind']` - whether to bind tomcat on lower port numbers, default `no`.
- `node['tomcat']['max_threads']` - maximum number of threads in the connector pool.
- `node['tomcat']['tomcat_auth']` -
- `node['tomcat']['client_auth']` - string Set to true if you want the SSL stack to require a valid certificate chain before accepting a connection, default `false`.
- `node['tomcat']['instances']` - A dictionary defining additional tomcat instances to run.
- `node['tomcat']['run_base_instance']` - Whether or not to run the "base" tomcat instance, default `true`.
- `node['tomcat']['environment']` - Environment variables to be setup when starting Tomcat
- `node['tomcat']['user']` -
- `node['tomcat']['group']` -
- `node['tomcat']['home']` -
- `node['tomcat']['base']` -
- `node['tomcat']['config_dir']` -
- `node['tomcat']['log_dir']` -
- `node['tomcat']['tmp_dir']` -
- `node['tomcat']['work_dir']` -
- `node['tomcat']['context_dir']` -
- `node['tomcat']['webapp_dir']` -
- `node['tomcat']['lib_dir']` -
- `node['tomcat']['endorsed_dir']` -
- `node['tomcat']['scheme']` set scheme for tomcat connector default value nil
- `node['tomcat']['secure']` to enable secure on or off with false/true default value nil
- `node['tomcat']['uriencoding']` configure uriencoding in server.xml default value 'UTF-8'

### Attributes for SSL
- `node['tomcat']['ssl_cert_file']` - SSL certificate file
- `node['tomcat']['ssl_chain_files']` - SSL CAcert chain files used for generating the SSL certificates
- `node['tomcat']['ssl_max_threads']` - maximum number of threads in the ssl connector pool, default `150`.
- `node['tomcat']['ssl_enabled_protocols']` - SSL enabled protocols. Please use 'TLSv1.2,TLSv1.1,TLSv1' or a smaller subset to mitigate poodle attack issues via SSL.
- `node['tomcat']['ciphers']` - SSL enabled ciphers
- `node['tomcat']['keystore_file']` - Location of the file where the SSL keystore is located
- `node['tomcat']['keystore_password']` - Generated by the `secure_password` method from the openssl cookbook; if you are using Chef Solo, set this attribute on the node
- `node['tomcat']['truststore_password']` - Generated by the `secure_password` method from the openssl cookbook; if you are using Chef Solo, set this attribute on the node
- `node['tomcat']['truststore_file']` - location of the file where the SSL truststore is located
- `node['tomcat']['certificate_dn']` - DN for the certificate
- `node['tomcat']['keytool']` - path to keytool, used for generating the certificate, location varies by platform

## Prerequisites
Due to the multitude of Java implementations you might want to use, this cookbook does not attempt to address the installation of a JRE/JDK. Please make sure that Java has been configured on the machine prior to the application any resources or recipes shipped in this cookbook.

## Usage
Simply include the recipe where you want Tomcat installed.

Due to the ways that some system init scripts call the configuration, you may wish to set the java options to include `JAVA_OPTS`. As an example for a java app server role:

```ruby
name "java-app-server"
run_list("recipe[tomcat]")
override_attributes(
  'tomcat' => {
    'java_options' => "${JAVA_OPTS} -Xmx128M -Djava.awt.headless=true"
  }
)
```

## Running Multiple Instances
To run multiple instances of Tomcat, populate the `instances` attribute, which is a dictionary of instance name => array of attributes.  Most of the same attributes that can be used globally for the tomcat cookbook can also be set per-instance - see resources/instance.rb for details.

If they are not set for a particular instance, the `base`, `home`, `config_dir`, `log_dir`, `work_dir`, `context_dir`, and `webapp_dir` attributes are created by modifying the global values to use the instance name.  For example, under Tomcat 7, with `home` /usr/share/tomcat7, `home` for instance "instance1" would be set to /usr/share/tomcat7-instance1.  The port attributes - `port`, `proxy_port`, `ssl_port`, `ssl_proxy_port`, `ajp_port`, and `shutdown_port` - are not inherited and must be set per-instance, `ajp_redirect_port` is also not inherited but defaults to `ssl_port`. The `ajp_listen_ip` is also not inherited and must be set per instance, when not set it defaults to listening on all adresses.  Other attributes that are not set are inherited unmodified from the global attributes.  Each instance must define `shutdown_port`, and at least one of `port`, `ssl_port` or `ajp_port`.

If you only want to run specific instances and not the "base" tomcat instances, you can set `run_base_instance` to `false`.

Here is an example partial role:

```javascript
...
"override_attributes": {
  "tomcat": {
    "run_base_instance": false,
    "instances": {
      "instance1": {
        "port": 8081,
        "shutdown_port": 8006
      },
      "lookup": {
        "port": 8082,
        "shutdown_port": 8007,
        "java_options": "-Xms1G -Xmx2G"
      }
    },
    ...
  }
  ...
}
```

## Managing Tomcat Users
The recipe `tomcat::users` included in this cookbook is used for managing Tomcat users. The recipe adds users and roles to the `tomcat-users.xml` conf file.

Users are defined by creating a `tomcat_users` data bag and placing [Encrypted Data Bag Items](http://docs.chef.io/chef/essentials_data_bags.html) in that data bag. Each encrypted data bag item requires an 'id', 'password', and a 'roles' field. The data bag key is retrieved from the default location `/etc/chef/encrypted_data_bag_secret`.

```javascript
{
  "id": "reset",
  "password": "supersecret",
  "roles": [
    "manager",
    "admin"
  ]
}
```

If you are a Chef Solo user the data bag items are not required to be encrypted and should not be.

## Defining Environment Variables
If your Tomcat application requires the usage of environment variables, you can define those into the `environment` attribute.

This is a sample on how to set-up some environment variables:

```javascript
...
"override_attributes": {
  "tomcat": {
    "environment": [
      {
        "VariableName": "LOCAL_HOME",
        "VariableValue": "/usr/root"
      },
      {
        "VariableName": "CONFIG_URL",
        "VariableValue": "http://127.0.0.1/config"
      }
    ]
  }
  ...
}
```

## Experimental Functionality
This cookbook is currently undergoing a ground up rewrite that will convert it to a pure library cookbook, more appropriate for the multitude of ways that Tomcat can be installed.  The existing attribute driven installs and tomcat_instance provider will eventually be deprecated in favor of a provider for installation, service management, and 1 or more providers for configuration.

### tomcat_install
tomcat_install installs an instance of the tomcat binary direct from Apache's mirror site. As distro packages are not used we can easily deploy per-instance installations and any version available on the Apache archive site can be installed.

#### properties
- `version`: The version to install. Default: 8.0.32
- `path`: Full path to the install directory. Default: /opt/tomcat_INSTANCENAME_VERSION
- `tarball_base_path`: The base path to the apache mirror containing the tarballs. Default: '[http://archive.apache.org/dist/tomcat/](http://archive.apache.org/dist/tomcat/)'
- `sha1_base_path`: The base path to the apache mirror containing the sha1 file. Default: '[http://archive.apache.org/dist/tomcat/](http://archive.apache.org/dist/tomcat/)'
- `exclude_docs`: Exclude ./webapps/docs from installation. Default true.
- `exclude_examples`: Exclude ./webapps/examples from installation. Default true.
- `exclude_manager`: Exclude ./webapps/manager from installation. Default: false.
- `exclude_host-manager`: Exclude ./webapps/host-manager from installation. Default: false.

#### example
Install an Tomcat 8.0.32 instance named 'helloworld' to /opt/tomcat_helloworld_8_0_32/ with a symlink at /opt/tomcat_helloworld/

```ruby
tomcat_install 'helloworld' do
  version '8.0.32'
end
```

### tomcat_service
tomcat_service sets up the installed tomcat instance to run using the appropriate init system. Currently only sys-v init is supported, but this will eventually support Upstart and Systemd where appropriate.

#### properties
- `path`: Full path to the install directory. Default: /opt/tomcat_INSTANCENAME

#### actions
- `start`
- `stop`
- `disable`
- `restart`

#### example

```ruby
tomcat_service 'helloworld' do
  action :start
end
```

## License & Authors
- Author: Seth Chisamore ([schisamo@chef.io](mailto:schisamo@chef.io))
- Author: Jamie Winsor ([jamie@vialstudios.com](mailto:jamie@vialstudios.com))
- Author: Phillip Goldenburg ([phillip.goldenburg@sailpoint.com](mailto:phillip.goldenburg@sailpoint.com))
- Auther: Mariano Cortesi ([mariano@zauberlabs.com](mailto:mariano@zauberlabs.com))
- Author: Brendan O'Donnell ([brendan.james.odonnell@gmail.com](mailto:brendan.james.odonnell@gmail.com))

```text
Copyright:: 2010-2015, Chef Software, Inc

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```
