layout: post
title: 在Gradle中使用jaxb
date: 2014-10-30 13:42:22
author: 黄博文
categories: [持续集成与部署]
tags: [Gradle, CI, Jaxb]
---

jaxb，全称为Java Architecture for Xml Binding,是一种将java对象与xml建立起映射的技术。其主要提供两个功能，一是将java对象映射为xml，二是将xml映射为java对象。JAXB有1.0版和2.0版。2.0版对应的JSR（Java specification request, java规格要求）是JSR 222。jaxb中的xjc工具能够将XML Schema转换为对应的java类。支持的XML类型包括XML DTD，XSD以及WSDL。而schemagen工具则可以将具有相应annotation标记的java类转换为XML结构。

<!-- more -->

ant脚本有xjc插件来实现对xml schema文件转换为java类的工作。而由于ant任务是gradle中的一等公民，所以我们可以直接在gradle脚本中使用ant的xjc插件来实现对xml schema和java类的映射。以下代码演示了如何将xsd格式和wsdl格式的xml转换为具体的java类。

```groovy build.gradle

configurations {
    jaxb
}

dependencies {
    jaxb 'com.sun.xml.bind:jaxb-impl:2.2.7'
    jaxb 'com.sun.xml.bind:jaxb-xjc:2.2.7'
}

ext {
    generatedSourceDir = 'src/main/generated'
}


task jaxb {

    doLast {
        file(generatedSourceDir).mkdirs()

        ant.taskdef(name: 'xjc', classname: 'com.sun.tools.xjc.XJCTask', classpath: configurations.jaxb.asPath)

        ant.xjc(destdir: generatedSourceDir,
                package: 'jaxb.ws.ship',
                schema: 'schema/shiporder.xsd'
        )

        ant.xjc(destdir: generatedSourceDir,
                package: 'jaxb.ws.hello',
                schema: 'schema/weather.wsdl'
        ) {
            arg(value: '-wsdl')
        }

    }
}

clean {
    ant.delete(dir: generatedSourceDir)
}

```

这里实现了将xsd和wsdl格式的xml文档转换为具体的java类。注意一点是如果wsdl中的schema过于简单，可能不会有具体的类生成。另外附上使用的示例文件。

shiporder.xsd文件如下：

```xml shiporder.xsd

<?xml version="1.0" encoding="UTF-8" ?>
<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema">

    <xs:element name="shiporder">
        <xs:complexType>
            <xs:sequence>
                <xs:element name="orderperson" type="xs:string"/>
                <xs:element name="shipto">
                    <xs:complexType>
                        <xs:sequence>
                            <xs:element name="name" type="xs:string"/>
                            <xs:element name="address" type="xs:string"/>
                            <xs:element name="city" type="xs:string"/>
                            <xs:element name="country" type="xs:string"/>
                        </xs:sequence>
                    </xs:complexType>
                </xs:element>
                <xs:element name="item" maxOccurs="unbounded">
                    <xs:complexType>
                        <xs:sequence>
                            <xs:element name="title" type="xs:string"/>
                            <xs:element name="note" type="xs:string" minOccurs="0"/>
                            <xs:element name="quantity" type="xs:positiveInteger"/>
                            <xs:element name="price" type="xs:decimal"/>
                        </xs:sequence>
                    </xs:complexType>
                </xs:element>
            </xs:sequence>
            <xs:attribute name="orderid" type="xs:string" use="required"/>
        </xs:complexType>
    </xs:element>

</xs:schema>

```

weather.wsdl文件内容如下：

```xml weather.wsdl

<?xml version="1.0" encoding="utf-8"?>
<wsdl:definitions xmlns:tm="http://microsoft.com/wsdl/mime/textMatching/" xmlns:soapenc="http://schemas.xmlsoap.org/soap/encoding/" xmlns:mime="http://schemas.xmlsoap.org/wsdl/mime/" xmlns:tns="http://www.webserviceX.NET" xmlns:soap="http://schemas.xmlsoap.org/wsdl/soap/" xmlns:s="http://www.w3.org/2001/XMLSchema" xmlns:soap12="http://schemas.xmlsoap.org/wsdl/soap12/" xmlns:http="http://schemas.xmlsoap.org/wsdl/http/" targetNamespace="http://www.webserviceX.NET" xmlns:wsdl="http://schemas.xmlsoap.org/wsdl/">
    <wsdl:types>
        <s:schema elementFormDefault="qualified" targetNamespace="http://www.webserviceX.NET">
            <s:element name="GetWeather">
                <s:complexType>
                    <s:sequence>
                        <s:element minOccurs="0" maxOccurs="1" name="CityName" type="s:string" />
                        <s:element minOccurs="0" maxOccurs="1" name="CountryName" type="s:string" />
                    </s:sequence>
                </s:complexType>
            </s:element>
            <s:element name="GetWeatherResponse">
                <s:complexType>
                    <s:sequence>
                        <s:element minOccurs="0" maxOccurs="1" name="GetWeatherResult" type="s:string" />
                    </s:sequence>
                </s:complexType>
            </s:element>
            <s:element name="GetCitiesByCountry">
                <s:complexType>
                    <s:sequence>
                        <s:element minOccurs="0" maxOccurs="1" name="CountryName" type="s:string" />
                    </s:sequence>
                </s:complexType>
            </s:element>
            <s:element name="GetCitiesByCountryResponse">
                <s:complexType>
                    <s:sequence>
                        <s:element minOccurs="0" maxOccurs="1" name="GetCitiesByCountryResult" type="s:string" />
                    </s:sequence>
                </s:complexType>
            </s:element>
            <s:element name="string" nillable="true" type="s:string" />
        </s:schema>
    </wsdl:types>
    <wsdl:message name="GetWeatherSoapIn">
        <wsdl:part name="parameters" element="tns:GetWeather" />
    </wsdl:message>
    <wsdl:message name="GetWeatherSoapOut">
        <wsdl:part name="parameters" element="tns:GetWeatherResponse" />
    </wsdl:message>
    <wsdl:message name="GetCitiesByCountrySoapIn">
        <wsdl:part name="parameters" element="tns:GetCitiesByCountry" />
    </wsdl:message>
    <wsdl:message name="GetCitiesByCountrySoapOut">
        <wsdl:part name="parameters" element="tns:GetCitiesByCountryResponse" />
    </wsdl:message>
    <wsdl:message name="GetWeatherHttpGetIn">
        <wsdl:part name="CityName" type="s:string" />
        <wsdl:part name="CountryName" type="s:string" />
    </wsdl:message>
    <wsdl:message name="GetWeatherHttpGetOut">
        <wsdl:part name="Body" element="tns:string" />
    </wsdl:message>
    <wsdl:message name="GetCitiesByCountryHttpGetIn">
        <wsdl:part name="CountryName" type="s:string" />
    </wsdl:message>
    <wsdl:message name="GetCitiesByCountryHttpGetOut">
        <wsdl:part name="Body" element="tns:string" />
    </wsdl:message>
    <wsdl:message name="GetWeatherHttpPostIn">
        <wsdl:part name="CityName" type="s:string" />
        <wsdl:part name="CountryName" type="s:string" />
    </wsdl:message>
    <wsdl:message name="GetWeatherHttpPostOut">
        <wsdl:part name="Body" element="tns:string" />
    </wsdl:message>
    <wsdl:message name="GetCitiesByCountryHttpPostIn">
        <wsdl:part name="CountryName" type="s:string" />
    </wsdl:message>
    <wsdl:message name="GetCitiesByCountryHttpPostOut">
        <wsdl:part name="Body" element="tns:string" />
    </wsdl:message>
    <wsdl:portType name="GlobalWeatherSoap">
        <wsdl:operation name="GetWeather">
            <wsdl:documentation xmlns:wsdl="http://schemas.xmlsoap.org/wsdl/">Get weather report for all major cities around the world.</wsdl:documentation>
            <wsdl:input message="tns:GetWeatherSoapIn" />
            <wsdl:output message="tns:GetWeatherSoapOut" />
        </wsdl:operation>
        <wsdl:operation name="GetCitiesByCountry">
            <wsdl:documentation xmlns:wsdl="http://schemas.xmlsoap.org/wsdl/">Get all major cities by country name(full / part).</wsdl:documentation>
            <wsdl:input message="tns:GetCitiesByCountrySoapIn" />
            <wsdl:output message="tns:GetCitiesByCountrySoapOut" />
        </wsdl:operation>
    </wsdl:portType>
    <wsdl:portType name="GlobalWeatherHttpGet">
        <wsdl:operation name="GetWeather">
            <wsdl:documentation xmlns:wsdl="http://schemas.xmlsoap.org/wsdl/">Get weather report for all major cities around the world.</wsdl:documentation>
            <wsdl:input message="tns:GetWeatherHttpGetIn" />
            <wsdl:output message="tns:GetWeatherHttpGetOut" />
        </wsdl:operation>
        <wsdl:operation name="GetCitiesByCountry">
            <wsdl:documentation xmlns:wsdl="http://schemas.xmlsoap.org/wsdl/">Get all major cities by country name(full / part).</wsdl:documentation>
            <wsdl:input message="tns:GetCitiesByCountryHttpGetIn" />
            <wsdl:output message="tns:GetCitiesByCountryHttpGetOut" />
        </wsdl:operation>
    </wsdl:portType>
    <wsdl:portType name="GlobalWeatherHttpPost">
        <wsdl:operation name="GetWeather">
            <wsdl:documentation xmlns:wsdl="http://schemas.xmlsoap.org/wsdl/">Get weather report for all major cities around the world.</wsdl:documentation>
            <wsdl:input message="tns:GetWeatherHttpPostIn" />
            <wsdl:output message="tns:GetWeatherHttpPostOut" />
        </wsdl:operation>
        <wsdl:operation name="GetCitiesByCountry">
            <wsdl:documentation xmlns:wsdl="http://schemas.xmlsoap.org/wsdl/">Get all major cities by country name(full / part).</wsdl:documentation>
            <wsdl:input message="tns:GetCitiesByCountryHttpPostIn" />
            <wsdl:output message="tns:GetCitiesByCountryHttpPostOut" />
        </wsdl:operation>
    </wsdl:portType>
    <wsdl:binding name="GlobalWeatherSoap" type="tns:GlobalWeatherSoap">
        <soap:binding transport="http://schemas.xmlsoap.org/soap/http" />
        <wsdl:operation name="GetWeather">
            <soap:operation soapAction="http://www.webserviceX.NET/GetWeather" style="document" />
            <wsdl:input>
                <soap:body use="literal" />
            </wsdl:input>
            <wsdl:output>
                <soap:body use="literal" />
            </wsdl:output>
        </wsdl:operation>
        <wsdl:operation name="GetCitiesByCountry">
            <soap:operation soapAction="http://www.webserviceX.NET/GetCitiesByCountry" style="document" />
            <wsdl:input>
                <soap:body use="literal" />
            </wsdl:input>
            <wsdl:output>
                <soap:body use="literal" />
            </wsdl:output>
        </wsdl:operation>
    </wsdl:binding>
    <wsdl:binding name="GlobalWeatherSoap12" type="tns:GlobalWeatherSoap">
        <soap12:binding transport="http://schemas.xmlsoap.org/soap/http" />
        <wsdl:operation name="GetWeather">
            <soap12:operation soapAction="http://www.webserviceX.NET/GetWeather" style="document" />
            <wsdl:input>
                <soap12:body use="literal" />
            </wsdl:input>
            <wsdl:output>
                <soap12:body use="literal" />
            </wsdl:output>
        </wsdl:operation>
        <wsdl:operation name="GetCitiesByCountry">
            <soap12:operation soapAction="http://www.webserviceX.NET/GetCitiesByCountry" style="document" />
            <wsdl:input>
                <soap12:body use="literal" />
            </wsdl:input>
            <wsdl:output>
                <soap12:body use="literal" />
            </wsdl:output>
        </wsdl:operation>
    </wsdl:binding>
    <wsdl:binding name="GlobalWeatherHttpGet" type="tns:GlobalWeatherHttpGet">
        <http:binding verb="GET" />
        <wsdl:operation name="GetWeather">
            <http:operation location="/GetWeather" />
            <wsdl:input>
                <http:urlEncoded />
            </wsdl:input>
            <wsdl:output>
                <mime:mimeXml part="Body" />
            </wsdl:output>
        </wsdl:operation>
        <wsdl:operation name="GetCitiesByCountry">
            <http:operation location="/GetCitiesByCountry" />
            <wsdl:input>
                <http:urlEncoded />
            </wsdl:input>
            <wsdl:output>
                <mime:mimeXml part="Body" />
            </wsdl:output>
        </wsdl:operation>
    </wsdl:binding>
    <wsdl:binding name="GlobalWeatherHttpPost" type="tns:GlobalWeatherHttpPost">
        <http:binding verb="POST" />
        <wsdl:operation name="GetWeather">
            <http:operation location="/GetWeather" />
            <wsdl:input>
                <mime:content type="application/x-www-form-urlencoded" />
            </wsdl:input>
            <wsdl:output>
                <mime:mimeXml part="Body" />
            </wsdl:output>
        </wsdl:operation>
        <wsdl:operation name="GetCitiesByCountry">
            <http:operation location="/GetCitiesByCountry" />
            <wsdl:input>
                <mime:content type="application/x-www-form-urlencoded" />
            </wsdl:input>
            <wsdl:output>
                <mime:mimeXml part="Body" />
            </wsdl:output>
        </wsdl:operation>
    </wsdl:binding>
    <wsdl:service name="GlobalWeather">
        <wsdl:port name="GlobalWeatherSoap" binding="tns:GlobalWeatherSoap">
            <soap:address location="http://www.webservicex.net/globalweather.asmx" />
        </wsdl:port>
        <wsdl:port name="GlobalWeatherSoap12" binding="tns:GlobalWeatherSoap12">
            <soap12:address location="http://www.webservicex.net/globalweather.asmx" />
        </wsdl:port>
        <wsdl:port name="GlobalWeatherHttpGet" binding="tns:GlobalWeatherHttpGet">
            <http:address location="http://www.webservicex.net/globalweather.asmx" />
        </wsdl:port>
        <wsdl:port name="GlobalWeatherHttpPost" binding="tns:GlobalWeatherHttpPost">
            <http:address location="http://www.webservicex.net/globalweather.asmx" />
        </wsdl:port>
    </wsdl:service>
</wsdl:definitions>

```

另外github上还有一些Gradle的插件来帮你实现xml和java对象的转换，但是本质上其实还是使用了jaxb的xjc ant插件实现的，只不过隐藏了实现细节，使用起来更加方便。感兴趣的可以看[https://github.com/jacobono/gradle-jaxb-plugin](https://github.com/jacobono/gradle-jaxb-plugin)。

