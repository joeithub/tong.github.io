title: Java获取IP地址
author: Joe Tong
tags:
  - JAVAEE
categories:  
  - IT 
date: 2019-12-04 14:23:32 
---

Java 通过Request请求获取IP地址

```
public class IpAddressUtil {
    /**
     * 获取Ip地址
     * @param request
     * @return
     */
    private static String getIpAddress(HttpServletRequest request) {
        String Xip = request.getHeader("X-Real-IP");
        String XFor = request.getHeader("X-Forwarded-For");
        if(StringUtils.isNotEmpty(XFor) && !"unKnown".equalsIgnoreCase(XFor)){
            //多次反向代理后会有多个ip值，第一个ip才是真实ip
            int index = XFor.indexOf(",");
            if(index != -1){
                return XFor.substring(0,index);
            }else{
                return XFor;
            }
        }
        XFor = Xip;
        if(StringUtils.isNotEmpty(XFor) && !"unKnown".equalsIgnoreCase(XFor)){
            return XFor;
        }
        if (StringUtils.isBlank(XFor) || "unknown".equalsIgnoreCase(XFor)) {
            XFor = request.getHeader("Proxy-Client-IP");
        }
        if (StringUtils.isBlank(XFor) || "unknown".equalsIgnoreCase(XFor)) {
            XFor = request.getHeader("WL-Proxy-Client-IP");
        }
        if (StringUtils.isBlank(XFor) || "unknown".equalsIgnoreCase(XFor)) {
            XFor = request.getHeader("HTTP_CLIENT_IP");
        }
        if (StringUtils.isBlank(XFor) || "unknown".equalsIgnoreCase(XFor)) {
            XFor = request.getHeader("HTTP_X_FORWARDED_FOR");
        }
        if (StringUtils.isBlank(XFor) || "unknown".equalsIgnoreCase(XFor)) {
            XFor = request.getRemoteAddr();
        }
        return XFor;
    }
}
```

详解
首先，我们获取 X-Forwarded-For 中第0位的IP地址，它就是在HTTP扩展协议中能表示真实的客户端IP。具体就像这样：

> X-Forwarded-For: client, proxy1, proxy2，proxy…

所以你应该知道为什么要取第0位了吧！
如果 X-Forwarded-For 获取不到，就去获取X-Real-IP ，X-Real-IP 获取不到，就依次获取Proxy-Client-IP 、WL-Proxy-Client-IP 、HTTP_CLIENT_IP 、 HTTP_X_FORWARDED_FOR 。最后获取不到才通过request.getRemoteAddr()获取IP，

X-Real-IP 就是记录请求的客户端真实IP。跟X-Forwarded-For 类似。

Proxy-Client-IP 顾名思义就是代理客户端的IP，如果客户端真实IP获取不到的时候，就只能获取代理客户端的IP了。

WL-Proxy-Client-IP 是在Weblogic下获取真实IP所用的的参数。

HTTP_CLIENT_IP 与 HTTP_X_FORWARDED_FOR 可以理解为X-Forwarded-For ， 因为它们是PHP中的用法。

备注：
StringUtils是Apache的工具类, pom.xml加依赖即可；也可以去Apache下载commons-lang3的最新包。

```
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.3.2</version>
</dependency>
```
