---
title: SOAP
---

## Introduction

SOAP (initially _Simple Object Access Protocol_) is an XML-based messaging
protocol designed for structured communication between services across networks,
independent of the transport layer (HTTP, SMTP, TCP, etc.).

## SOAP Message Structure

- Envelope: Root element that defines the message as SOAP.
- Header (optional): Carries metadata (auth tokens, routing info, transaction
  context). Common header attributes:
  - `mustUnderstand="1"`: If the destination doesn’t understand it, it must
    generate a SOAP Fault.
  - `actor`: Directs the header to a specific intermediary or recipient URI.
- Body: Contains the main XML payload (procedure call or response).
- Fault (optional): Structured error information.
  - Code
  - Reason
  - Detail

### Sample SOAP Request

```xml
<?xml version="1.0"?>
<soap:Envelope xmlns:soap="http://www.w3.org/2003/05/soap-envelope">
  <soap:Header/>
  <soap:Body xmlns:m="http://www.example.org/stock">
    <m:GetStockPrice>
      <m:StockName>IBM</m:StockName>
    </m:GetStockPrice>
  </soap:Body>
</soap:Envelope>
```

### Sample SOAP Response

```xml
<?xml version="1.0"?>
<soap:Envelope xmlns:soap="http://www.w3.org/2003/05/soap-envelope">
  <soap:Header/>
  <soap:Body xmlns:m="http://www.example.org/stock">
    <m:GetStockPriceResponse>
      <m:Price>34.5</m:Price>
    </m:GetStockPriceResponse>
  </soap:Body>
</soap:Envelope>
```

### Sample Faulty SOAP Response

```xml
<soap:Envelope xmlns:soap="http://www.w3.org/2003/05/soap-envelope">
  <soap:Header/>
  <soap:Body>
    <soap:Fault>
      <soap:Code>
        <soap:Value>soap:Receiver</soap:Value>
      </soap:Code>
      <soap:Reason>
        <soap:Text xml:lang="en">Server error processing GetStockPrice</soap:Text>
      </soap:Reason>
      <soap:Detail>
        <m:ErrorDetail xmlns:m="http://www.example.org/stock">
          <m:ErrorCode>500</m:ErrorCode>
          <m:ErrorMessage>Database unavailable</m:ErrorMessage>
        </m:ErrorDetail>
      </soap:Detail>
    </soap:Fault>
  </soap:Body>
</soap:Envelope>
```

## WSDL & Service Discovery

WSDL (Web Services Description Language) is an XML-based language defining
service interfaces, including operations, messages, data types, bindings, and
endpoints.

**UDDI** was historically used for service registry and discovery, publishing
WSDL to a repository where consumers could find services.

A WSDL includes:

- `<types>`: Schema definitions for data types
- `<message>`: Abstract data (request/response parts)
- `<portType>`: Defines operations & their messages
- `<binding>`: Concrete protocol (SOAP) and encoding
- `<service>`: Endpoint(s) with `<soap:address> `

WSDL defines _what_ can be called (`portType`), _how_ (`binding`), and _where_
(`service`)

```xml
<definitions name="HelloService"
             targetNamespace="http://www.examples.com/wsdl/HelloService.wsdl"
             xmlns="http://schemas.xmlsoap.org/wsdl/"
             xmlns:soap="http://schemas.xmlsoap.org/wsdl/soap/"
             xmlns:tns="http://www.examples.com/wsdl/HelloService.wsdl"
             xmlns:xsd="http://www.w3.org/2001/XMLSchema">

  <types>
    <xsd:schema targetNamespace="http://www.examples.com/wsdl/HelloService.wsdl">
      <!-- define complex types if needed -->
    </xsd:schema>
  </types>

  <message name="sayHelloRequest">
    <part name="name" type="xsd:string"/>
  </message>
  <message name="sayHelloResponse">
    <part name="greeting" type="xsd:string"/>
  </message>

  <portType name="Hello_PortType">
    <operation name="sayHello">
      <input message="tns:sayHelloRequest"/>
      <output message="tns:sayHelloResponse"/>
    </operation>
  </portType>

  <binding name="Hello_Binding" type="tns:Hello_PortType">
    <soap:binding style="rpc"
                  transport="http://schemas.xmlsoap.org/soap/http"/>
    <operation name="sayHello">
      <soap:operation soapAction="urn:examples:helloservice#sayHello"/>
      <input>
        <soap:body use="encoded" namespace="urn:examples:helloservice"
                   encodingStyle="http://schemas.xmlsoap.org/soap/encoding/"/>
      </input>
      <output>
        <soap:body use="encoded" namespace="urn:examples:helloservice"
                   encodingStyle="http://schemas.xmlsoap.org/soap/encoding/"/>
      </output>
    </operation>
  </binding>

  <service name="Hello_Service">
    <port name="Hello_Port" binding="tns:Hello_Binding">
      <soap:address location="http://www.examples.com/SayHello/"/>
    </port>
  </service>
</definitions>
```

## Security in SOAP (WS-Security)

SOAP security is based on _WS-Security_, an OASIS extension for messaging
security, offering:

- _XML Signature_ for integrity and non-repudiation
- _XML Encryption_ for confidentiality
- _Security tokens_ (X.509, SAML, Kerberos, Username/Password)

This allows _end-to-end messaging security_, even through intermediaries .

## Transaction & Reliability Support

SOAP supports `WS-*` standards for transactional integrity and reliability, such
as:

- WS-ReliableMessaging: ensures ordered, lossless message delivery
- WS-AtomicTransaction, WS-BusinessActivity, WS-Coordination for managing
  distributed transactions (ACID)

## SOAP vs REST

| Feature          | SOAP                                       | REST                                      |
| ---------------- | ------------------------------------------ | ----------------------------------------- |
| **Type**         | Protocol (strict standard)                 | Architectural style                       |
| **Format**       | XML only                                   | Any format (JSON, XML, HTML)              |
| **Transport**    | Any protocol (HTTP, SMTP, TCP…)            | HTTP only (verbs: GET, POST, PUT, DELETE) |
| **Statefulness** | Usually stateful, used in transactions     | Stateless                                 |
| **Security**     | WS-Security (message-level), rich features | Transport-level SSL/TLS                   |
| **Reliability**  | Built-in WS-ReliableMessaging              | Retries handled client-side               |
| **Performance**  | Heavier, XML parser overhead               | Lightweight, cacheable, faster            |
| **Flexibility**  | Rigid contract based on WSDL               | Highly flexible, loosely coupled          |
| **Coupling**     | Tightly coupled client/server              | Looser coupling, dynamic via hypermedia   |
