# Mac Slave

## Requirements

**Restart MacOs after configuration change**

### Configure a Jenkins User
Create an user on the Mac with administrator privileges. It will be your connection user for Mac Plugin Global configuration.

Add sudo NOPASSWD to this user in /etc/sudoers :
[see how to configure sudo without password](https://www.robertshell.com/blog/2016/12/3/use-sudo-command-osx-without-password)

To maximize security, you can configure it only for "chmod" and "sysadminctl" command used by the plugin :

`[USERNAME] ALL = NOPASSWD: /usr/sbin/sysadminctl -addUser mac-?????????? -password ??????????, /usr/sbin/sysadminctl -deleteUser mac-??????????, /bin/chmod -R 700 /Users/mac-??????????/`

### Enable SSH for all users
Go to System Preferences -> Sharing, and enable Remote Login for All users :

<img src="https://zupimages.net/up/19/47/q7yq.png" width="500"/>

### SSH configuration
In /etc/ssh/sshd_config file, uncomment and update values of parameters MaxAuthTries, MaxSessions, ClientAliveInterval and ClientAliveCountMax to your need.

example of configuration for 10 Jenkins and 1 Mac with 10 users allowed :

- MaxAuthTries 10
- MaxSessions 100
- ClientAliveInterval 30
- ClientAliveCountMax 150

For more informations about sshd_config consult the
[Official Documentation](https://man.openbsd.org/sshd_config)

### Install Java Development Kit

You will need this to connect to Jenkins from the command line.

[Java Development Kit](https://www.oracle.com/java/technologies/javase/javase-jdk8-downloads.html)

### Develoment Setup

You will need install Xcode and other development tools and make sure the iOS project can run properly

### Install Fastlane

[Fastlane](https://fastlane.tools/) is tool to build and release mobile apps.

[Setup](https://docs.fastlane.tools/getting-started/ios/setup/)

### Add Slave in Jenkins

Manage Jenkins -> Manage Nodes -> New Node

<img src="https://i.ibb.co/1nx5m0t/newnode.png" width="1300"/>

- `Remote root directory` - Job forders managed by Jenkins
- `of executors` - The maximum number of concurrent builds that Jenkins may perform on this node.
- `Labels` - Define a label for slave node which can be use in the pipeline syntax

Select created Node

<img src="https://i.ibb.co/BwCBGMv/node.png" width="900"/>

Click **Launch** to connect to Jenkins as a slave

### Run Jenkins Slave as a Service

**Create plist**

This is the most complicated step in the process. Create a new file called com.jenkins.ci.plist and add the following.

Replace **<JENKIN_URL>** with the address of Jenkins machine. Note that this example does not use authentication for Jenkins. If your setup uses authentication (it most likely does) you will need to add additional argument to the plist.

```
&lt?xml version="1.0" encoding="UTF-8"?> &lt!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd"> &ltplist version="1.0"> &ltdict> &ltkey>Label&lt/key> &ltstring>com.jenkins.ci&lt/string> &ltkey>UserName&lt/key> &ltstring>jenkins&lt/string> &ltkey>SessionCreate&lt/key> &lttrue/> &ltkey>ProgramArguments&lt/key> &ltarray> &ltstring>java&lt/string> &ltstring>-Djava.awt.headless=true&lt/string> &ltstring>-jar&lt/string> &ltstring>/Users/jenkins/Desktop/slave.jar&lt/string> &ltstring>-jnlpUrl&lt/string> &ltstring>http://<JENKIN_URL>/computer/mac_vm/slave-agent.jnlp&lt/string> &lt/array> &ltkey>KeepAlive&lt/key> &lttrue/> &ltkey>StandardOutPath&lt/key> &ltstring>/Users/jenkins/Desktop/stdout.log&lt/string> &ltkey>StandardErrorPath&lt/key> &ltstring>/Users/jenkins/Desktop/error.log&lt/string> &lt/dict> &lt/plist>
```

**Move to LaunchDaemons**

Move this file to /Library/LaunchDaemons it will be prompted for authentication

**Change file permissions**

Now need to change the file permissions so that it will be handled correctly by root.

```
sudo chown root com.jenkins.ci.plist sudo chmod 644 com.jenkins.ci.plist
```
