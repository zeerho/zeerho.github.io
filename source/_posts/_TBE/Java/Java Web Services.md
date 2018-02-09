---
title: Java Web Services
date: 2017-01-01 09:00:00
tags: [Java]
---

# 1. 基于 SOAP 的 Web 服务

## 1.1 TimeServer 的服务端点接口（SEI，Service Endpoint Interface）

```java
package ch01.ts; //time server

import javax.jws.WebService;
import javax.jws.WebMethod;
import javax.jws.soap.SOAPBinding;
import javax.jws.soap.SOAPBinding.Style;

@WebService
@SOAPBinding(style=Style.RPC)
public interface TimeServer{
    @WebMethod String getTimeAsString();
    @WebMethod long getTimeAsElapsed();
}
```

## 1.2 TimeServer 服务实现 Bean（SIB，Service Implementation Bean）[^new Date()]

```java
package ch01.ts;

import java.util.Date;
import javax.jws.WebService;

@WebService(endpointInterface="ch01.ts.TimeServer")
public class TimeServerImpl implements TimeServer{
    public String getTimeAsString(){return new Date().toString();}
    public long getTimeAsElapsed(){return new Date().getTime(); }
}
```

## 1.3 将 Java 程序发布为 Web 服务

Web 服务可以发布到独立的 Java 应用服务器中，也可以如下简单地为其提供一个 Web 服务发布程序。

```java
package ch01.ts;

import javax.xml.ws.Endpoint;

public class TimeServerPublisher{
    public static void main(String[] args){
        Endpoint.publish("http://127.0.0.1:9876/ts",new TimeServerImpl());
    }
}
```

这样发布的服务同一时刻只能处理一个客户端服务请求。

<span id="return1"></span>
## 1.4 通过浏览器测试 Web 服务

通过浏览器查看由发布程序自动产生的服务契约 WSDL（Web Services Description Language）文档。访问地址 `http://127.0.0.1:9876/ts?wsdl`。详细报文见[这里](#wsdl)。

## 1.5 传输SOAP消息时HTTP请求的内容结构

```xml
POST http://127.0.0.1:9876/ts HTTP/ 1.1
Accept:text/xml
Accept:multipart/*
Accept:application/soap
User-Agent:SOAP::Lite/Perl/0.69
Content-Length:434
Content-Type:text/xml;charset=utf-8
SOAPAction:""

<?xml version="1.0" encoding="UTF-8"?>
<soap:Envelope
    soap:encodingStyle="http://schemas.xmlsoap.org/soap/encoding/"
    xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/"
    xmlns:soapenc="http://schemas.xmlsoap.org/soap/encoding/"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:tns="http://ts.ch01/"
    xmlns:xsd="http://www.w3.org/2001/XMLSchema">
    <soap:Body>
        <tns:getTimeAsString xsi:nil="true"/>
    </soap:Body>
</soap:Envelope>
```

- HTTP 请求第一行指定了请求方法为“POST”。在 SOAP 请求中，通常是 POST 方式，而不是 GET，因为只有  POST 请求才有 Body 域，可以用来封装一个 SOAP 消息。URL 之后是协议和协议版本号。
- HTTP 头之后是一些由冒号分割的键值对。它们出现的顺序是任意的。“Accept”键以 MIME（Multipurpose Internet Mail Extension）类型/子类型的方式出现，3个“Accept”键表明请求者客户端可以分别处理任意类型的 XML 文档，附带任意数量的不同类型文档（SOAP消息通常可以有任意多个附件）的请求响应，以及一个 SOAP 文档。“SOAPAction”键通常用来表示这个 HTTP 请求是一个 Web 服务请求，同时取值可以为空字符串，也有可能是请求的 Web 服务操作的名称。
- 一个空行分隔 HTTP 头数据和 Body 域。本例中 Body 域包含一个 SOAP 文档，称为 SOAP 信封（SOAP Envelope）。

针对 Web 服务来说，底层的 Java 支持库处理 HTTP 请求，并将 SOAP 信封拆包，分析出客户端请求的服务操作，然后调用对应的方法，生成相应的 SOAP 消息返回给客户端。

## 1.6 来自 TimeServer 服务的 HTTP 响应

```xml
HTTP/1.1 200 OK
Content-Length:323
Content-Type:text/xml;charset=utf-8
Client-Date:Mon,28 Apr 2008 02:12:54 GMT
Client-Peer:127.0.0.1:9876
Client-Response-Num:1

<?xml version="1.0"?>
<soapenv:Envelope
    xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/"
    xmlns:xsd="http://www.w3.org/2001/XMLSchema">
    <soapenv:Body>
        <ans:getTimeAsStringResponse xmlns:ans="http://ts.ch01/">
            <return>Mon Apr 28 14:12:54 CST 2008</return>
        </ans:getTimeAsStringResponse>
    </soapenv:Body>
</soapenv:Envelope>
```

根据 WSDL 文档提供的信息（这里未列出），要求从 Web 服务返回的值出现在 XML 的 `<return>` 元素中。

## 1.7 Java 语言实现的 Web 服务客户端

```java
package ch01.ts;

import javax.xml.namespace.QName;
import javax.xml.ws.Service;
import java.net.URL;

class TimeClient{
    public static void main(String[] args) throws Exception{
        URL url=new URL("http://localhost:9876/ts?wsdl");
        //Qualified name of the service.
        QName qname=new QName("http://ts.ch01/","TimeServerImplService");
        //Create a factory for the service.
        Service service=Service.create(url,qname);
        //Extract the endpoint interface, the service "port".
        TimeServer eif=service.getPort(TimeServer.class);
        
        System.out.println(eif.getTimeAsString());
        System.out.println(eif.getTimeAsElapsed());
    }
}
```

Java客户端明确地创建了一个语法格式为“URI：本地名称”的 XML 限定名，`java.xml.namespace.Qname` 类用来表示一个 XML 限定名。本类中的命名空间 URI 在 WSDL 文档中提供；本地名称在 WSDL 文档最后一段的 Service 元素中描述（这里是“TimeServerImplService”）。
调用 `Service.create()` 方法后，执行 `getPort()` 方法，该方法返回一个 Java 对象，可以通过此对象调用 portType 中描述的操作。

# 2. 全面了解 WSDL

## 2.1 从 WSDL 文档中生成客户端支持代码

2.1.1 使用 wsimport

通过 Jdk1.6+ 提供的 wsimport 工具轻松完成基于 SOAP 协议的 Web 服务客户端生成工作。

**首先确认服务端已经运行，这样才能访问到 WSDL。**
在 cmd.exe 中运行命令：
`wsimport -d class -s source -keep http://localhost:9876/ts?wsdl`
    
- `-d`：class 文件存放目录；
- `-s`：java 文件存放目录；
- `-keep`：保留 java 文件。

该命令以 cmd.exe 运行目录为当前目录。class 和 source 都是当前目录下已存在的自定义目录。默认使用服务实现的包名作为客户端代码包名，注意避免覆盖。若 SEI 采用 Document 的绑定方式则会自动生成相关工件。
其中，契约地址可换成本地 wsdl 文件的路径。

根据之前的 wsdl 文档，wsimport 会生成 TimeServer.class 和 TimeServerImplService.class 两个字节码文件。可基于这两个文件编写服务调用客户端。

wsimport 使用的编码为操作系统的编码，且源代码中的注释使用的是英文。

2.1.2 使用 wsdl2java

wsdl2java.exe 通常在 webservice 框架（如 CXF、AXIS2）的文件夹内。

`wsdl2java [options] [URL|wsdl文件]`

- `-d`：源码输出目录；
- `-compile`：是否编译源码；
- `-classdir`：字节码输出目录；
- `-b`：指定数据绑定文件；

生成的客户端中会出现类似 `JaxbElement<String>` 的对象，而不是 `String`。可以通过参数 `-b` 来指定数据绑定文件，并在该文件中进行设置。
比如先创建文件 cxf_binding.xml，内容如下。然后指定参数 `-b cxf_binding.xml`，若文件不在当前路径，则需写明绝对路径。

```xml
<jaxb:bindings version="2.0" xmlns:jaxb="http://java.sun.com/xml/ns/jaxb">
    <jaxb:bindings>
        <jaxb:globalBindings generateElementProperty="false" />
    </jaxb:bindings>
</jaxb:bindings>

```


```java
package client;

class TimeClientWSDL {
    public static void main(String[] args) {
        TimeServerImplService service = new TimeServerImplService();
        TimeServer eif = service.getTimeServerImplPort();
        
        System.out.println(eif.getTimeAsString());
        System.out.println(eif.getTimeAsElapsed());
    }
}
```

可以看到与服务对应的Qname和服务端点等复杂细节都被隐藏了。

**基于WSDL协议所生成的相关工件（Artifact）编写服务客户端的一些常用做法：**

1. 通过 wsimport 产生的 Class 文件中的构造方法构造一个服务对象，比如本例中的 `TimeServerImplService` 类。
2. 通过构建的服务对象调用对应的 get 方法。

## 2.2 WSDL文档结构
根元素 `<definitions>` 内部分为 3 部分：

- **类型（Types）：**非必需。通常基于像XML模式（XML Schema）这样的类型系统来提供数据类型的定义。这种用来定义数据类型的文档就是XSD（XML Schema Definition）。在数据类型部分可以持有、指向或引入一个XSD。若为空，则只使用简单数据类型，比如`xsd:String`和`xsd:long`。
- **消息（Message）：**定义了实现服务的相关消息。消息使用的数据类型来自前面的类型部分。消息的顺序表明了服务的请求模式：“in/out”对应请求/响应（request/response）模式；“out/in”对应要求/响应（solicit/response）模式。
- **portType：**以命名的操作描述了服务，每一个操作都是一个或多个消息。服务操作的名称由Web服务方法的注解`@WebMethods`指定。“portType”与Java接口类似，抽象描述服务而不包括实现细节。
- **绑定（Binding）：** WSDL定义从抽象到具体的描述。绑定部分可类比为Java接口的实现。它详细说明了“portType”部分的抽象定义：
    - `<soap:binding style="RPC" transport="http://schemas.xmlsoap.org/soap/http">`其中`transport`属性指明收发SOAP消息所使用的传输协议。
    - `style`属性可选“rpc”或“document”，默认为“document”。由SEI中的`@SOAPBinding(style=Style.RPC)`指定。
    - SOAP消息中使用的数据编码格式有两种：“literal”（逐字的）和“encoded”。其中“encoded”不符合WS-I指导原则，一般不使用。
- **服务（service）：**指定了一个或多个端点，端点中描述了服务的功能和所包括的操作。从技术上看，服务部分包括一个或多个port（端口），每个port包括portType（接口）及与之对应的binding（接口实现）。port一词源自分布式系统。

## 2.4 wsgen 工具与 JAX-B 工件（Artifacts）
任何 Document 样式的服务，都需要由 wsgen 工具产生的工件（Artifacts，支持客户端开发的相关代码资源）。
假设 TimeServerImpl.class(注意是 **.class** 而不是 **.java**)的路径为 `C:/workspace/MyProject/build/classes/com/demo/TimeServerImpl.class`，则需要在 `C:/workspace/MyProject/build/classes` 目录内运行下列命令，否则提示 `Class Not Found.`
`wsgen -keep -cp . com.demo.TimeServerImpl`
运行该命令后将在 `../com/demo` 目录中生成一个“javax”文件夹，里面包含生成的代码资源。
在底层，wsgen 工具利用 JAX-B（Java API for XML-Binding）相关的 API 包实现“Java对象**编码（Marshal）**为 XML 文档”和“XML 文档**解码（Unmarshal）**为 Java 对象”的功能。
具体的注解见 `javax.xml.bind.annotation.*`。

**利用wsgen工具产生WSDL文档**
`wsgen -cp "." -wsdl com.demo.TimeServerImpl `
命令运行目录同前文所述。运行后将在`../MyProject/build/classes`内生成相应的`.wsdl`和`.xsd`。

#3. Web Service注解
JSR 181
**注释类：** 
##javax.jws.WebService
**注释：** 
当实现 Web Service 时，`@WebService` 注释标记 Java 类；实现 Web Service 接口时，标记服务端点接口（SEI）。
要点：
• 实现 Web Service 的 Java 类必须指定 `@WebService` 或 `@WebServiceProvider` 注释。不能同时提供这两种注释。
• 此注释适用于客户机/服务器 SEI 或 JavaBeans 端点的服务器端点实现类。
• 如果注释通过 `endpointInterface` 属性引用了某个 SEI，那么还必须使用 `@WebService` 注释来注释该 SEI。
**属性：** 
注释目标：类型
• *name*
`wsdl:portType` 的名称。缺省值为 Java 类或接口的非限定名称。（字符串）
• *targetNamespace*
指定从 Web Service 生成的 WSDL 和 XML 元素的 XML 名称空间。缺省值为从包含该 Web Service 的包名映射的名称空间。（字符串）
• *serviceName*
指定 Web Service 的服务名称`wsdl:service`。缺省值为 "Java 类的简单名称 + Service"。（字符串）
• *endpointInterface*
指定用于定义服务的抽象 Web Service 约定的服务端点接口的限定名。如果指定了此限定名，那么会使用该服务端点接口来确定抽象 WSDL 约定。（字符串）
• *portName*
`wsdl:portName`。缺省值为 WebService.name+Port。（字符串）
• *wsdlLocation*
指定用于定义 Web Service 的 WSDL 文档的 Web 地址。Web 地址可以是相对路径或绝对路径。（字符串）

**注释类：** 
##javax.jws.WebMethod
**注释：** 
`@WebMethod` 注释表示作为一项 Web Service 操作的方法。
将此注释应用于客户机或服务器服务端点接口（SEI）上的方法，或者应用于 JavaBeans 端点的服务器端点实现类。
要点：
• 仅支持在使用 `@WebService` 注释来注释的类上使用 `@WebMethod` 注释。
**属性：** 
注释目标：方法
• *operationName*
指定与此方法相匹配的 `wsdl:operation` 的名称。缺省值为 Java 方法的名称。（字符串）
• *action*
定义此操作的行为。对于 SOAP 绑定，此值将确定 SOAPAction 头的值。缺省值为 Java 方法的名称。（字符串）
• *exclude*
指定是否从 Web Service 中排除某一方法。缺省值为 false。（布尔值）

**注释类：** 
##javax.jws.Oneway
**注释：** 
`@Oneway` 注释将一个方法表示为只有输入消息而没有输出消息的 Web Service 单向操作。
将此注释应用于客户机或服务器服务端点接口（SEI）上的方法，或者应用于 JavaBeans 端点的服务器端点实现类。
**属性：** 
注释目标：方法
没有适用于 Oneway 注释的属性。

**注释类：** 
##javax.jws.WebParam
**注释：** 
`@WebParam` 注释用于定制从单个参数至 Web Service 消息部件和 XML 元素的映射。
将此注释应用于客户机或服务器服务端点接口（SEI）上的方法，或者应用于 JavaBeans 端点的服务器端点实现类。
**属性：** 
注释目标：参数
• *name*
参数的名称。如果操作是远程过程调用（RPC）类型并且未指定`partName` 属性，那么这是用于表示参数的 `wsdl:part` 属性的名称。如果操作是文档类型或者参数映射至某个头，那么 `name` 是用于表示该参数的 XML 元素的局部名称。如果操作是文档类型、参数类型为 BARE 并且方式为 OUT 或 INOUT，那么必须指定此属性。（字符串）
• *partName*
定义用于表示此参数的 `wsdl:part` 属性的名称。仅当操作类型为 RPC 或者操作是文档类型并且参数类型为 BARE 时才使用此参数。（字符串）
• *targetNamespace*
指定参数的 XML 元素的 XML 名称空间。当属性映射至 XML 元素时，仅应用于文档绑定。缺省值为 Web Service 的 `targetNamespace`     。（字符串）
• *mode*
此值表示此方法的参数流的方向。有效值为 IN、INOUT 和 OUT。（字符串）
• *header*
指定参数是在消息头还是消息体中。缺省值为 false。（布尔值）

**注释类：** 
##javax.jws.WebResult
**注释：** 
`@WebResult` 注释用于定制从返回值至 WSDL 部件或 XML 元素的映射。
将此注释应用于客户机或服务器服务端点接口（SEI）上的方法，或者应用于 JavaBeans 端点的服务器端点实现类。
**属性：** 
注释目标：方法
• *name*
当返回值列示在 WSDL 文件中并且在连接上的消息中找到该返回值时，指定该返回值的名称。对于 RPC 绑定，这是用于表示返回值的 `wsdl:part` 属性的名称。对于文档绑定，`name`参数是用于表示返回值的 XML 元素的局部名。对于 RPC 和 DOCUMENT/WRAPPED 绑定，缺省值为 return。对于 DOCUMENT/BARE 绑定，缺省值为"方法名 + Response"。（字符串）
• *targetNamespace*
指定返回值的 XML 名称空间。仅当操作类型为 RPC 或者操作是文档类型并且参数类型为 BARE 时才使用此参数。（字符串）
• *header*
指定头中是否附带结果。缺省值为false。（布尔值）
• *partName*
指定 RPC 或 DOCUMENT/BARE 操作的结果的部件名称。缺省值为`@WebResult.name`。（字符串）

**注释类：** 
##javax.jws.HandlerChain
**注释：** 
`@HandlerChain` 注释用于使 Web Service 与外部定义的处理程序链相关联。
只能通过对 SEI 或实现类使用 `@HandlerChain` 注释来配置服务器端的处理程序。
但是可以使用多种方法来配置客户端的处理程序。可以通过对生成的服务类或者 SEI 使用 `@HandlerChain` 注释来配置客户端的处理程序。此外，可以按程序在服务上注册您自己的 HandlerResolver 接口实现，或者按程序在绑定对象上设置处理程序链。
**属性：** 
注释目标：类型
• *file*
指定处理程序链文件所在的位置。文件位置可以是采用外部格式的绝对 java.net.URL，也可以是类文件中的相对路径。（字符串）
• *name*
指定配置文件中处理程序链的名称。（字符串）

**注释类：** 
##javax.jws.SOAPBinding
**注释：** 
`@SOAPBinding` 注释指定 Web Service 与 SOAP 消息协议之间的映射。
将此注释应用于客户机或服务器服务端点接口（SEI）上的类型或方法，或者应用于 JavaBeans 端点的服务器端点实现类。
方法级别的注释仅限于它可以指定的对象，仅当 style 属性为 DOCUMENT 时才使用该注释。如果未指定方法级别的注释，那么将使用类型的 `@SOAPBinding` 行为。
**属性：** 
注释目标：类型或方法
• *style*
定义发送至 Web Service 和来自 Web Service 的消息的编码样式。有效值为 DOCUMENT 和 RPC。缺省值为 DOCUMENT。（字符串）
• *use*
定义用于发送至 Web Service 和来自 Web Service 的消息的格式。缺省值为 LITERAL。ENCODED 在 Feature Pack for Web Services 中不受支持。（字符串）
• *parameterStyle*
确定方法的参数是否表示整个消息体，或者参数是否是封装在执行操作之后命名的顶级元素中的元素。有效值为 WRAPPED 或 BARE。对于DOCUMENT 类型的绑定只能使用 BARE 值。缺省值为 WRAPPED。（字符串）

JSR 224
**注释类：** 
##javax.xml.ws.BindingType
**注释：** 
`@BindingType` 注释指定在发布此类型的端点时要使用的绑定。
将此注释应用于 JavaBeans 端点或提供程序端点的服务器端点实现类。
要点：
• 可以通过将该注释的值指定为 javax.xml.ws.soap.SOAPBinding.SOAP11HTTP_MTOM_BINDING 或 javax.xml.ws.soap.SOAPBinding.SOAP12HTTP_MTOM_BINDING 来对 Java bean 端点实现类使用 `@BindingType` 注释以启用 MTOM。
**属性：** 
注释目标：类型：
• *value*
指示绑定标识 Web 地址。有效值为 javax.xml.ws.soap.SOAPBinding.SOAP11HTTP_BINDING、javax.xml.ws.soap.SOAPBinding.SOAP12HTTP_BINDING 和 javax.xml.ws.http.HTTPBinding.HTTP2HTTP_BINDING。缺省值为 javax.xml.ws.soap.SOAPBinding.SOAP11HTTP_BINDING。（字符串）

**注释类：** 
##javax.xml.ws.RequestWrapper
**注释：** 
`@RequestWrapper` 注释提供 JAXB 生成的请求包装器 bean、元素名称和名称空间，用于对在运行时使用的请求包装器 bean 进行序列化和反序列化。
从 Java 对象开始时，此元素用来解决 document literal 方式下的重载冲突。在这种情况下，只有 className 属性是必需的。
将此注释应用于客户机或服务器服务端点接口（SEI）上的方法，或者应用于 JavaBeans 端点的服务器端点实现类。
**属性：** 
注释目标：方法
• *localName*
指定用于表示请求包装器的 XML 模式元素的局部名称。缺省值为在 javax.jws.WebMethod 注释中定义的 operationName。（字符串）
• *targetNamespace*
指定请求包装器方法的 XML 名称空间。缺省值为 SEI 的目标名称空间。（字符串）
• *className*
指定用于表示请求包装器的类的名称。（字符串）

**注释类：** 
##javax.xml.ws.ResponseWrapper
**注释：** 
`@ResponseWrapper` 注释提供 JAXB 生成的响应包装器 bean、元素名称和名称空间，用于对在运行时使用的响应包装器 bean 进行序列化和反序列化。
从 Java 对象开始时，此元素用来解决 document literal 方式下的重载冲突。在这种情况下，只有 className 属性是必需的。
将此注释应用于客户机或服务器服务端点接口（SEI）上的方法，或者应用于 JavaBeans 端点的服务器端点实现类。
**属性：** 
注释目标：方法
• *localName*
指定用于表示请求包装器的 XML 模式元素的局部名称。缺省值为 operationName + Response。operationName 是在 `javax.jws.WebMethod` 注释中定义的。（字符串）
• *targetNamespace*
指定请求包装器方法的 XML 名称空间。缺省值为 SEI 的目标名称空间。（字符串）
• *className*
指定用于表示响应包装器的类的名称。（字符串）

**注释类：** 
##javax.xml.ws.ServiceMode
**注释：** 
`@ServiceMode` 注释指定服务提供者是需要对整个协议消息具有访问权还是只需对消息有效内容具有访问权。
要点：
• 仅支持在使用 `@WebServiceProvider` 注释来注释的类上使用 `@ServiceMode` 注释。
**属性：** 
注释目标：类型
• *value*
指示提供者类是接受消息的有效内容 PAYLOAD 还是整个消息 MESSAGE。缺省值为 PAYLOAD。（字符串）

**注释类：** 
##javax.xml.ws.WebFault
**注释：** 
`@WebFault` 注释将 WSDL 故障映射至 Java 异常。对从 WSDL 故障消息引用的全局元素生成的 JAXB 类型进行序列化期间，该注释用来捕获故障的名称。它还可以用来定制从特定于服务的异常到 WSDL 故障的映射。
此注释只能应用于客户机或服务器上的故障实现类。
**属性：** 
注释目标：类型
• *name*
指定用于表示 WSDL 文件中相应故障的 XML 元素的局部名称。必须指定实际值。（字符串）
• *targetNamespace*
指定用于表示 WSDL 文件中相应故障的 XML 元素的名称空间。（字符串）
• *faultBean*
指定故障 bean 类的名称。（字符串）

**注释类：** 
##javax.xml.ws.WebServiceProvider
**注释：** 
`@WebServiceProvider` 注释表示一个类满足 JAX-WS 提供程序实现类的要求。
要点：
• 实现 Web Service 的 Java 类必须指定 `@WebService` 或 `@WebServiceProvider` 注释。不能同时提供这两种注释。
• 只有服务实现类才支持 `@WebServiceProvider` 注释。
• 任何具有 `@WebServiceProvider` 注释的类都必须具有名为 invoke 的操作。
**属性：** 
注释目标：类型
• *targetNamespace*
指定从 Web Service 生成的 WSDL 和 XML 元素的 XML 名称空间。缺省值为从包含该 Web Service 的包名映射的名称空间。（字符串）
• *serviceName*
指定 Web Service 的服务名称 `wsdl:service`。缺省值为 "Java 类的简单名称 + Service"。（字符串）
• *portName*
`wsdl:portName`。缺省值为"类的名称 + Port"。（字符串）
• *wsdlLocation*

JSR 250
**注释类：**
##javax.annotation.Resource
**注释：**
`@Resource` 注释标记应用程序所需要的 WebServiceContext 资源。
将此注释应用于 JavaBeans 端点或提供程序端点的服务器端点实现类。对容器进行初始化时，容器会将 WebServiceContext 资源的实例添加到端点实现中。
**属性：**
注释目标：字段或方法
• *type*
指示资源的 Java 类型。您需要使用缺省值 java.lang.Object 或者 javax.xml.ws.Web ServiceContext 值。如果类型是缺省值，那么必须将资源添加到字段或方法中。在这种情况下，字段的类型或者由方法定义的 JavaBeans 属性的类型必须为 javax.xml.ws.WebServiceContext。（字符串）

**注释类：**
##javax.annotation.PostConstruct
**注释：**
`@PostConstruct` 注释标记需要在对类执行依赖性注入之后才执行的方法。
将此注释应用于 JAX-WS 应用程序处理程序、JavaBeans 端点或提供程序端点的服务器端点实现类。
**属性：**
注释目标：方法

**注释类：**
##javax.annotation.PreDestroy
**注释：**
`@PreDestroy` 注释标记在容器除去实例时必须执行的方法。
将此注释应用于 JAX-WS 应用程序处理程序、JavaBeans 端点或提供程序端点的服务器端点实现类。
**属性：**
注释目标：方法

适用于使用 `@WebService` 注释的类的方法的规则
下列规则适用于使用 `@WebService` 注释来注释的类的方法。
如果某个实现类的 `@WebService` 注释引用了 SEI，那么该实例类不能具有任何 `@WebMethod` 注释。
无论是否指定了 `@WebMethod` 注释，SEI 的所有公用方法都被认为是已显示的方法。在包含 exclude 属性的 SEI 上使用 `@WebMethod` 注释是不正确的。
对于不引用 SEI 的实现类，如果对 `@WebMethod` 注释指定了值 exclude=true，那么不会显示该方法。如果未指定 `@WebMethod` 注释，那么将显示包括继承的方法在内的所有公用方法，但是不包括从 java.lang.Object 继承的方法。

# 附录
## <span id="wsdl">WSDL详细报文</span>
```xml
<?xml version="1.0" encoding="UTF-8"?>
<definitions
    xmlns="http://schemas.xmlsoap.org/wsdl/"
    xmlns:tns="http://ts.ch01/"
    xmlns:xsd="http://www.w3.org/2001/XMLSchema"
    xmlns:soap="http://schemas.xmlsoap.org/wsdl/soap/"
    targetNamespace="http://ts.ch01/"
    name="TimeServerImplService">
    <types></types>
    
    <message name="getTimeAsString"></message>
    <message name="getTimeAsStringResponse">
        <part name="return" type="xsd:string"></part>
    </message>
    <message name="getTimeAsElapsed"></message>
    <message name="getTimeAsElapsedResponse">
        <part name="return" type="xsd:long"></part>
    </message>
    
    <portType name="TimeServer">
        <operation name="getTimeAsString" parameterOrder="">
            <input message="tns:getTimeAsString"></input>
            <output message="tns:getTimeAsStringResponse"></output>
        </operation>
        <operation name="getTimeAsElapsed" parameterOrder="">
            <input message="tns:getTimeAsElapsed"></input>
            <output message="tns:getTimeAsElapsedResponse"></output>
        </operation>
    </portType>
    
    <binding name="TimeServerImplPortBinding" type="tns:TimeServer">
        <soap:binding style="rpc" transport="http://schemas.xmlsoap.org/soap/http">
        </soap:binding>
        <operation name="getTimeAsString">
            <soap:operation soapAction=""></soap:operation>
            <input>
                <soap:body use="literal" namespace="http://ts.ch01/"></soap:body>
            </input>
            <output>
                <soap:body use="literal" namespace="http://ts.ch01/"></soap:body>
            </output>
        </operation>
        <operation name="getTimeAsElapsed">
            <soap:operation soapAction=""></soap:operation>
            <input>
                <soap:body use="literal" namespace="http://ts.ch01/"></soap:body>
            </input>
            <output>
                <soap:body use="literal" namespace="http://ts.ch01/"></soap:body>
            </output>
        </operation>
    </binding>
    
    <service name="TimeServerImplService">
        <port name="TimeServerImplPort" binding="tns:TimeServerImplPortBinding">
            <soap:address location="http://localhost:9876/ts"></soap:address>
        </port>
    </service>
</definitions>
```
[返回](#return1)

[^new Date()]:该服务以字符串的形式向调用者返回当前系统时间或返回一个 Unix 时间戳，即从1970年1月1日（UTC（Universal Time Coordinated）/GMT（Greenwich Mean Time）的午夜零点）开始到目前所经过的秒数，不考虑闰秒。最早出现的 UNIX 操作系统考虑到计算机产生的年代和应用的时限综合取了 1970 年 1 月 1 日作为 UNIX TIME 的纪元时间(开始时间)，而 java 作为源于 UNIX 的一种语言自然也遵循了这一约束。

