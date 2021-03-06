#tomcat

####Table of Contents

1. [Overview](#overview)
2. [Module Description](#module-description)
3. [Setup](#setup)
  * [A couple of examples](#a-couple-of-examples)
4. [Usage](#usage)
  * [Classes and Defined Types](#classes-and-defined-types)
    * [Class: tomcat](#class-tomcat)
    * [Define: tomcat::instance](#define-tomcatinstance)
    * [Common parameters](#common-parameters)
    * [Define: tomcat::userdb_entry](#define-tomcatuserdb_entry)
5. [To Do](#to-do)
6. [Contributors](#contributors)

##Overview

The tomcat module installs and configures Apache Tomcat instances from either the packages available in your distribution's repositories, or from any archive file you provide to it.

##Module description

This module will install the desired version of the Apache Tomcat Web Application Container from almost any possible source, including the repositories available on the target system (distribution repositories or third-party sources like [JPackage](http://www.jpackage.org) and [EPEL](https://fedoraproject.org/wiki/EPEL))  
A long list of parameters permit a fine-tuning of the server and the JVM. It is for example possible to configure admin applications, install extra tomcat libraries, configure log4j as the standard logger, or enable the remote JMX listener.  
The creation of individual instances following [Apache's guidelines](http://tomcat.apache.org/tomcat-8.0-doc/RUNNING.txt) is also supported via a custom type.

##Setup

tomcat will affect the following parts of your system:

* tomcat packages and dependencies
* tomcat service(s)
* instances configuration
* tomcat user database(s) and authorized users (defined type)

Including the main class is enough to install the default version of tomcat provided by your distribution, and run it with default settings.

```puppet
include ::tomcat
```

####A couple of examples

Install from archive instead of package

```puppet
class { '::tomcat':
  install_from => 'archive',
  version      => '8.0.15'
}
```

Disable default instance and setup 2 individual instances

```puppet
class { '::tomcat':
  service_ensure => 'stopped',
  service_enable => false
}
tomcat::instance { 'instance1':
  control_port  => 8005,
  http_port     => 8080,
  ajp_connector => false,
  …
}
tomcat::instance { 'instance2':
  control_port    => 8006,
  http_port       => 8081,
  manage_firewall => true,
  …
}
```

Use a non-default JVM and run it with custom options

```puppet
class { '::tomcat':
  java_home => '/usr/java/jre1.7.0_65',
  java_opts => ['-server', '-Xmx2048m', '-Xms256m', '-XX:+UseConcMarkSweepGC']
}
```

Enable the AJP connector on non-default port with custom parameters

```puppet
class { '::tomcat':
  …
  ajp_connector => true,
  ajp_port      => 8090,
  ajp_params    => { address => '127.0.0.1', packetSize => 12288 }
}
```

Enable the manager/host-manager webapps and configure default admin

```puppet
class { '::tomcat':
  …
  admin_webapps  => true,
  admin_user     => 'tomcatmaster',
  admin_password => 'meow'
}
```

Add an additional admin for the manager

```puppet
::tomcat::userdb_entry { 'foo':
  username => 'foo',
  password => 'bar',
  roles    => ['manager-gui', 'manager-script']
}
```

Enable the remote [JMX listener](http://tomcat.apache.org/tomcat-8.0-doc/config/listeners.html#JMX_Remote_Lifecycle_Listener_-_org.apache.catalina.mbeans.JmxRemoteLifecycleListener) for remote JVM monitoring

```puppet
class { '::tomcat':
  …
  jmx_listener      => true,
  jmx_registry_port => '8050',
  jmx_server_port   => '8051',
  jmx_bind_address  => $ipaddress_eth0,
  catalina_opts     => ['-Dcom.sun.management.jmxremote', '-Dcom.sun.management.jmxremote.ssl=false', '-Dcom.sun.management.jmxremote.authenticate=false']
}
```

Use log4j for Tomcat internal logging and provide a custom XML configuration file

```puppet
class { '::tomcat':
  …
  log4j             => true,
  log4j_enable      => true,
  log4j_conf_type   => 'xml',
  log4j_conf_source => 'puppet:///modules/my_configs/tomcat_log4j.xml'
}
```
Use with custom packages/custom installation layouts (eg. with [Ulyaoth](https://forge.puppetlabs.com/aco/ulyaoth))

```puppet
class { '::tomcat':
  package_name               => 'ulyaoth-tomcat8',
  version                    => '8.0.15'
  service_name               => 'tomcat',
  config_path                => '/opt/tomcat/bin/setenv.sh',
  catalina_home              => '/opt/tomcat',
  catalina_pid               => '$CATALINA_TMPDIR/$SERVICE_NAME.pid',
  admin_webapps_package_name => 'ulyaoth-tomcat8-admin',
  tomcat_native              => true,
  tomcat_native_package_name => 'ulyaoth-tomcat-native'
  …
}
```

##Usage

This module distinguishes two different contexts:
* global: default instance and global libraries
* instance: individual tomcat instance

Both contexts share most of their parameters.

###Classes and Defined Types

####Class: `tomcat`

Primary class and entry point of the module

**Parameters within `tomcat`:**

**Packages and service**

#####`install_from`
What type of source to install from. The module will download the necessary files by itself. Valid values are `package` and `archive`. Defaults to `package`.

#####`archive_source`
Source of the tomcat server archive, if installed from archive. Supports local files, puppet://, http://, https:// and ftp://. Defaults to `http://archive.apache.org/dist/tomcat/tomcat-${maj_version}/v${version}/bin/apache-tomcat-${version}.tar.gz`

#####`version`
Tomcat full version number. The valid format is 'x.y.z'. If you install tomcat from package and define this value manually, make **sure** this version of tomcat if available in your system's repositories, since several sub-parameters depend on it. Default depends on the distribution.

#####`package_name`
Tomcat package name. Ignored if installed from archive. Default depends on the distribution.

#####`tomcat_native`
Whether to install the Tomcat Native library. Boolean value. Defaults to `false`.

#####`tomcat_native_package_name`
Tomcat Native library package name. Default depends on the distribution. 

#####`log4j`
Whether to install the log4j library. Boolean value. Defaults to `false`.

#####`log4j_package_name`
Log4j package name. Default depends on the distribution.

See also [Common parameters](#common-parameters)

####Define: `tomcat::instance`

Create a tomcat instance

**Parameters within `tomcat::instance`:**

#####`root_path`
Absolute path to the root of all tomcat instances. Defaults to `/var/lib/tomcats`.  
*Note:* instances will be installed in `${root_path}/${title}` and $CATALINA_BASE will be set to that folder

See also [Common parameters](#common-parameters)

####Common parameters

Parameters common to both `tomcat` and `tomcat::instance`

**Packages and service**

#####`service_name`
Tomcat service name. Defaults to `${package_name}` (global) / `${package_name}_${title}` (instance).

#####`service_ensure`
Whether the service should be running. Valid values are `stopped` and `running`. Defaults to `running`.

#####`service_enable`
Whether to enable the tomcat service. Boolean value. Defaults to `true`.

#####`service_start`
Optional override command for starting the service. Default depends on the platform.

#####`service_stop`
Optional override command for stopping the service. Default depends on the platform.

#####`tomcat_user`
Tomcat user. Defaults to `${service_name}` (Debian) / `tomcat` (all other distributions).

#####`tomcat_group`
Tomcat group. Defaults to `${tomcat_user}`.

#####`enable_extras`
Whether to install tomcat extra libraries. Boolean value. Defaults to `false`.  
*Warning:* extra libraries are enabled globally if defined within the global context

#####`manage_firewall`
Whether to automatically manage firewall rules. Boolean value. Defaults to `false`.

**Security and administration**

#####`admin_webapps`
Whether to enable admin webapps (manager/host-manager). This will also install the required packages if tomcat was installed from package. This parameter is ignored if tomcat was installed from archive, since tomcat archives always contain these apps. Boolean value. Defaults to `true`.

#####`admin_webapps_package_name`
Admin webapps package name. Default depends on the distribution.

#####`create_default_admin`
Whether to create default admin user (roles: 'manager-gui', 'manager-script', 'admin-gui' and 'admin-script'). Boolean value. Defaults to `false`.

#####`admin_user`
Admin user name. Defaults to `tomcatadmin`.

#####`admin_password`
Admin user password. Defaults to `password`.

**Server configuration**

#####`apr_listener`
Whether to enable the [APR Lifecycle Listener](http://tomcat.apache.org/tomcat-8.0-doc/apr.html#APR_Lifecycle_Listener_Configuration)

#####`apr_sslengine`
Name of the SSLEngine to use with the APR Lifecycle Listener. Defaults to `undef` (use tomcat default).

#####`jmx_listener`
Whether to enable the [JMX Remote Lifecycle Listener](http://tomcat.apache.org/tomcat-8.0-doc/config/listeners.html#JMX_Remote_Lifecycle_Listener_-_org.apache.catalina.mbeans.JmxRemoteLifecycleListener). The listener can be further configured via a series of parameters:
 - `jmx_registry_port`: JMX/RMI registry port for the JMX Remote Lifecycle Listener. Defaults to `8050` (global) / `8052` (instance).
 - `jmx_server_port`: JMX/RMI server port for the JMX Remote Lifecycle Listener. Defaults to `8051` (global) / `8053` (instance).
 - `jmx_bind_address`: JMX/RMI server interface address for the JMX Remote Lifecycle Listener. Defaults to `undef` (use tomcat default).

#####`control_port`
Server control port. Defaults to `8005` (global) / `8006` (instance).

#####`threadpool_executor`
Whether to enable the default [Executor (thread pool)](http://tomcat.apache.org/tomcat-8.0-doc/config/executor.html). Boolean value. Defaults to `false`. The Executor can be further configured via a series of parameters (will use Tomcat's defaults if not specified):
 - `threadpool_name`: a unique reference name. Defaults to `tomcatThreadPool`.
 - `threadpool_nameprefix`: name prefix for each thread created by the executor
 - `threadpool_maxthreads`: max number of active threads in this pool
 - `threadpool_minsparethreads`: minimum number of threads always kept alive
 - `threadpool_params`: optional Hash of additional parameters to put in the Executor where the key is the XML attribute name and the value, the attribute's value

#####`executors`
An array of custom `Executor` entries to be added to the `Service` block. Each entry is to be supplied as a Hash of attributes/values for the `Executor` XML node.

#####`http_connector`
Whether to enable the [HTTP connector](http://tomcat.apache.org/tomcat-8.0-doc/config/http.html). Boolean value. Defaults to `true`. The Connector can be further configured via a series of parameters (will use Tomcat's defaults if not specified):
 - `http_port`: HTTP connector port. Defaults to `8080` (global) / `8081` (instance).
 - `http_protocol`: protocol to use
 - `http_use_threadpool`: whether to use the previously described Executor within the HTTP connector. Boolean value. Defaults to `false`.
 - `http_connectiontimeout`: timeout for a connection
 - `http_uriencoding`: encoding to use for URI
 - `http_compression`: whether to use compression. Boolean value. Defaults to `false`.
 - `http_maxthreads`: maximum number of executor threads
 - `http_params`: optional Hash of additional parameters to put in the HTTP Connector where the key is the XML attribute name and the value, the attribute's value
 
#####`ssl_connector`
Whether to enable the [SSL-enabled HTTP connector](http://tomcat.apache.org/tomcat-8.0-doc/config/http.html#SSL_Support). Boolean value. Defaults to `false`. The Connector can be further configured via a series of parameters (will use Tomcat's defaults if not specified):
 - `ssl_port`: SSL connector port. Defaults to `8443` (global) / `8444` (instance). The HTTP connector's `redirect port` will also be set to this value.
 - `ssl_protocol`: protocol to use
 - `ssl_use_threadpool`: whether to use the previously described Executor within the HTTPS connector (boolean)
 - `ssl_connectiontimeout`: timeout for a connection
 - `ssl_uriencoding`: encoding to use for URI
 - `ssl_compression`: whether to use compression. Boolean value. Defaults to `false`.
 - `ssl_maxthreads`: maximum number of executor threads
 - `ssl_clientauth`: whether to require a valid certificate chain from the client
 - `ssl_sslprotocol`: SSL protocol(s) to use
 - `ssl_keystorefile`: path to keystore file
 - `ssl_params`: optional Hash of additional parameters to put in the HTTPS Connector where the key is the XML attribute name and the value, the attribute's value

#####`ajp_connector`
Whether to enable the [AJP connector](http://tomcat.apache.org/tomcat-8.0-doc/config/ajp). Boolean value. Defaults to `true`. The Connector can be further configured via a series of parameters (will use Tomcat's defaults if not specified):
 - `ajp_port`: AJP connector port. Defaults to `8009` (global) / `8010` (instance).
 - `ajp_protocol`: protocol to use. Defaults to `AJP/1.3`
 - `ajp_use_threadpool`: whether to use the previously described Executor within the AJP connector.  Boolean value. Defaults to `false`.
 - `ajp_connectiontimeout`: timeout for a connection
 - `ajp_uriencoding`: encoding to use for URI
 - `ajp_maxthreads`: maximum number of executor threads
 - `ajp_params`: optional Hash of additional parameters to put in the AJP Connector where the key is the XML attribute name and the value, the attribute's value

#####`connectors`
An array of custom `Connector` entries to be added to the `Service` block. Each entry is to be supplied as a Hash of attributes/values for the `Connector` XML node.

#####`jvmroute`
Engine's [jvmRoute](http://tomcat.apache.org/tomcat-8.0-doc/config/engine.html#Common_Attributes) attribute. Defaults to `undef` (use tomcat default).

#####`hostname`
Name of the default [Host](http://tomcat.apache.org/tomcat-8.0-doc/config/host.html). Defaults to `localhost`. The Host can be further configured via a series of parameters (will use Tomcat's defaults if not specified):
 - `host_appbase`: Application Base directory for this virtual host
 - `host_autodeploy`: whether Tomcat should check periodically for new or updated web applications while Tomcat is running
 - `host_deployOnStartup`: whether web applications from this host should be automatically deployed when Tomcat starts
 - `host_undeployoldversions`: whether to clean unused versions of web applications deployed using parallel deployment
 - `host_unpackwars`: whether to unpack web application archive (WAR) files 
 - `host_params`: optional Hash of additional parameters to put in the Host container where the key is the XML attribute name and the value, the attribute's value

#####`lockout_realm`
Whether to enable the [LockOut Realm](http://tomcat.apache.org/tomcat-8.0-doc/config/realm.html#LockOut_Realm_-_org.apache.catalina.realm.LockOutRealm). Boolean value. Defaults to `true`.

#####`userdatabase_realm`
Whether to enable the [UserDatabase Realm](http://tomcat.apache.org/tomcat-8.0-doc/config/realm.html#UserDatabase_Realm_-_org.apache.catalina.realm.UserDatabaseRealm). 
Boolean value. Defaults to `true`. The User Database Realm is inserted within the Lock Out Realm if it is enabled.

#####`realms`
An array of custom `Realm` entries to be added to the `Engine` container. Each entry is to be supplied as a Hash of attributes/values for the `Realm` XML node.

#####`singlesignon_valve`
Whether to enable the [Single Sign On Valve](http://tomcat.apache.org/tomcat-8.0-doc/config/valve.html#Single_Sign_On_Valve). Boolean value. Defaults to `false`.

#####`accesslog_valve`
Whether to enable the [Access Log Valve](http://tomcat.apache.org/tomcat-8.0-doc/config/valve.html#Access_Log_Valve). Boolean value. Defaults to `true`.

#####`valves`
An array of custom `Valve` entries to be added to the `Host` block. Each entry is to be supplied as a Hash of attributes/values for the `Valve` XML node.

#####`globalnaming_resources`
An array of `Resource` entries to be added to the `GlobalNamingResources` block. Each entry is to be supplied as a Hash of attributes/values for the `Resource` XML node. See [Global Resources](http://tomcat.apache.org/tomcat-8.0-doc/config/globalresources.html) for the list of possible attributes.

#####`context_resources`
An array of `ResourceLink` entries to be added to the context.xml file. Each entry is to be supplied as a Hash of attributes/values for the `Resource` XML node. See [Resource Links](http://tomcat.apache.org/tomcat-8.0-doc/config/context.html#Resource_Links) for the list of possible attributes.

**Global configuration file / environment variables**

#####`config_path`
Absolute path to the environment configuration (*setenv*). Default depends on the platform.

Please see [catalina.sh](http://svn.apache.org/repos/asf/tomcat/tc8.0.x/trunk/bin/catalina.sh) for a description of the following environment variables.

#####`catalina_home`
$CATALINA_HOME. Default depends on the platform.

#####`catalina_base`
$CATALINA_BASE. Default depends on the platform.

#####`jasper_home`
$JASPER_HOME. Defaults to `catalina_home`.

#####`catalina_tmpdir`
$CATALINA_TMPDIR. Defaults to `${catalina_base}/temp`

#####`catalina_pid`
$CATALINA_PID. Defaults to: `/var/run/${service_name}.pid`

#####`catalina_opts`
$CATALINA_OPTS. Array. Defaults to `[]`.

#####`java_home`
$JAVA_HOME. Defaults to `undef` (use tomcat default).

#####`java_opts`
$JAVA_OPTS. Array. Defaults to `['-server']`.

#####`jpda_enable`
Enable JPDA debugger. Boolean value. Effective only if installed from archive. Defaults to `false`.

#####`jpda_transport`
$JPDA_TRANSPORT. Defaults to `undef` (use tomcat default).

#####`jpda_address`
$JPDA_ADDRESS. Defaults to `undef` (use tomcat default).

#####`jpda_suspend`
$JPDA_SUSPEND. Defaults to `undef` (use tomcat default).

#####`jpda_opts`
$JPDA_OPTS. Array. Defaults to `[]`.

#####`security_manager`
Whether to enable the security manager. Boolean value. Defaults to `false`.

#####`lang`
Tomcat locale. Defaults to `undef` (use tomcat default).

#####`shutdown_wait`
How long to wait for a graceful shutdown before killing the process. Value in seconds. Only available on RedHat 6- systems and if installed from package. Defaults to `30`.

#####`shutdown_verbose`
Whether to display start/shutdown messages. Boolean value. Only available on RedHat 6- systems and if installed from package. Defaults to `false`.

#####`custom_fragment`
Custom environment variables, one per line.

**Logging**

Some extra documentation about [log4j](http://logging.apache.org/log4j/)'s usage with tomcat is available on [this page](http://tomcat.apache.org/tomcat-8.0-doc/logging.html#Using_Log4j).

#####`log4j_enable`
Whether to use log4j rather than *java.util.logging* for Tomcat internal logging. Boolean value. Defaults to `false`.  
*Warning:* log4j is enabled globally if defined within the global context

#####`log4j_conf_type`
Log4j configuration type. Valid values are `ini` and `xml`. Defaults to `ini`.

#####`log4j_conf_source`
Where to get log4j's configuration from. A [sample file](https://raw.githubusercontent.com/tOnI0/aco-tomcat/master/files/log4j/log4j.properties) is provided with this module. Defaults to the sample file `log4j.properties`.

####Define: `tomcat::userdb_entry`

Create tomcat UserDatabase entries

**Parameters within `tomcat::userdb_entry`:**

#####`username`
User name (string)

#####`password`
User password (string)

#####`roles`
User roles (array of strings)

##To Do

* Create systemd units for Fedora 20+
* Parameters validation

##Contributors

* [etlweather](https://github.com/etlweather)

Features request and contributions are always welcome!
