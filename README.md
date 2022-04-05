# spring-shell-vuln

## Spring4Shell: Spring core RCE vulnerability
--------
Spring has Confirmed the RCE in Spring Framework. The team has just published the statement along with the mitigation guides for the issue. Now, this vulnerability can be tracked as CVE-2022-22965.

Some information about the Spring4Shell vulnerability and have shared the details on Spring4Shell: Details and Exploit post. Additionally, the security team from Praetorian has confirmed Spring Core on JDK9+ is vulnerable to remote code execution due to a bypass for CVE-2010-1622.

Initially, it was started on 30th March, the first notification of the vulnerability was hinted at by the leader of the KnownSec 404 team, Heige. He twitted with the warning message "Spring core RCE （JDK >=9" along with the PoC Picture. 

![image](https://user-images.githubusercontent.com/76834257/161725483-dad89530-63f5-4ea4-8724-382250a75482.png)

As we go live with the story of vulnerability, Heige disappears from Twitter. Don't know the reason behind this but there may be something.

## Events

At the end of 2021, the internet was on fire with the drop of Zero-day a remote code execution vulnerability also known as [Log4Shell](https://www.cyberkendra.com/2021/12/worst-log4j-rce-zeroday-dropped-on.html), in Apache Log4j2. The vulnerability was found by the Alibaba Cloud security team. 

###### - This vulnerability is NOT as bad a Log4Shell. All attack scenario are more complex because of the nature of Class Loader Manipulation attacks work in Java. Spring4Shell exploitation requires deep Java knowledge to get a functioning POC. Class Loader Manipulation is more complicated to understand than the Log4Shell vulnerability.

Today, researchers found another worst vulnerability which may cause severe damage. Now the bug is been tracked as CVE-2022-22965, we may call it Spring4Shell. The vulnerability exists in the Spring core with the JDK version greater or equal to 9.0.

Spring Framework and derived framework spring -beans-*.jar files or CachedIntrospectionResults.class

All the details below are confirmed now. I don't take responsibility for any damage caused.

## Vulnerability Details and Investigation
As one of the world's most popular Java lightweight open-source framework, Spring allows developers to focus on business logic and simplifies the development cycle of Java enterprise applications.

Exploitation requires an endpoint with DataBinder enabled (e.g. a POST request that decodes data from the request body automatically) and depends heavily on the servlet container for the application. For example, when Spring is deployed to Apache Tomcat, the WebAppClassLoader is accessible, which allows an attacker to call getters and setters to ultimately write a malicious JSP file to disk. However, if Spring is deployed using the Embedded Tomcat Servlet Container the classloader is a LaunchedURLClassLoader which has limited access.

However, in the JDK9 version (and above) of the Spring framework, a remote attacker can obtain the AccessLogValve object and malicious field values through the parameter binding function of the framework on the basis of meeting certain conditions.

- It is currently known that triggering this vulnerability requires two basic conditions:
- Use the Spring MVC framework &  JDK9 and above

### (1). Check the JDK version number 
On the running server of the organization system, run the "java -version" command to check the running JDK version. If the version number is less than or equal to 8, it is not affected by the vulnerability.

### (2). Check for Spring framework usage
1. If the organization system project is deployed in the form of a war package, follow the steps below to judge.

- Unzip the war package: Change the suffix of the war file to .zip and unzip the zip file
- Search for a jar file in spring-beans-*.jar format (for example, spring-beans-5.3.16.jar) in the decompression directory. If it exists, it means that the business system is developed using the spring framework.
- If the spring-beans-*.jar file does not exist, search for the existence of the CachedIntrospectionResuLts.class file in the decompression directory. If it exists, it means that the business system is developed using the Spring framework.

2. If the organization system project runs directly and independently in the form of a jar package, judge according to the following steps.

- Unzip the jar package: Change the suffix of the jar file to .zip, and unzip the zip file.
- Search for a jar file in spring-beans-*.jar format (for example, spring-beans-5.3.16.jar) in the decompression directory. If it exists, it means that the business system is developed using the spring framework.
- If the spring-beans-*.jar file does not exist, search for the existence of the CachedIntrospectionResuLts.class file in the decompression directory. If it exists, it means that the business system is developed using the spring framework.

### (3) Comprehensive Investigation
After completing the above two steps of troubleshooting, the following two conditions are met at the same time to determine that it is affected by this vulnerability:

1. JDK version number is 9 and above;
2. using the spring framework or derived framework.

## Vulnerability Fix Guides
Now the [Spring team has fixed the vulnerability](https://www.cyberkendra.com/2022/03/spring4shell-spring-confirmed-rce-in.html) and released the latest versions of Spring Boot 2.6.6 and 2.5.12 that depend on Spring Framework 5.3.18


### WAF protection
On network protection devices such as WAF, implement rule filtering for strings such as "class.*", "Class.*", "*.class.*", and "*.Class.*" according to the actual traffic situation of deployed services. After filtering the rules, test the business operation to avoid additional impact.
### Temporary repair measures
Temporary repair of the leak should be performed in the following two steps at the same time:

1. Search the @InitBinder annotation globally in the application to see if the dataBinder.setDisallowedFields method is called in the method body. If the introduction of this code snippet is found, add {"class.*","Class.* to the original blacklist ","*.class.*", "*.Class.*"}. (Note: If this code snippet is used a lot, it needs to be appended everywhere)

2. Create the following global class under the project package of the application system, and ensure that this class is loaded by Spring (it is recommended to add it in the package where the Controller is located). After the class is added, the project needs to be recompiled and packaged, and tested for functional verification. and republish the project.
import org.springframework.core.annotation.Order;

        import org.springframework.web.bind.WebDataBinder;

        import org.springframework.web.bind.annotation.ControllerAdvice;

        import org.springframework.web.bind.annotation.InitBinder;

        @ControllerAdvice

        @Order(10000)

        public class GlobalControllerAdvice{ 

             @InitBinder

             public void setAllowedFields(webdataBinder dataBinder){

             String[]abd=new string[]{"class.*","Class.*","*.class.*","*.Class.*"};

             dataBinder.setDisallowedFields(abd);

             }

        }

![image](https://user-images.githubusercontent.com/76834257/161733562-9e22bba6-0d1b-447b-8880-f6cb07970de4.png)

From the [Git Repository](https://github.com/spring-projects/spring-framework/commit/7f7fb58dd0dae86d22268a4b59ac7c72a6c22529) of Spring projects, it seems that the Spring developer is working on a fix for the remote code execution vulnerability, but we have to wait for the official confirmation.
