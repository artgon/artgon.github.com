--- 
layout: post
title: Camel and RabbitMQ
meta-description: A guide for setting up Apache Camel with RabbitMQ
---

A common model for implementing [EIP](http://www.eaipatterns.com/ "Enterprise Integration Patterns") in Java, is to use [Apache Camel](http://camel.apache.org/), with some sort of MQ or JMS backing. One of the more interesting MQ frameworks in use today is the [Erlang](http://www.erlang.org/)-based [RabbitMQ](http://www.rabbitmq.com/). 

Camel is a very widely used [integration platform](http://en.wikipedia.org/wiki/Integration_platform). It's very flexible and takes care of wiring up services, mediating messages, transforming data, among many other useful things. It also comes with tons of [components](http://camel.apache.org/components.html) out of the box. Notice, one of the components it supports is AMQP. However, that would make life far too easy for RabbitMQ users. What the Camel documentation doesn't mention is that Camel is using Apache's [Qpid](http://qpid.apache.org/) as the AMQP implementation. For whatever reason, Qpid does [not play nicely](http://www.rabbitmq.com/interoperability.html) with RabbitMQ, thus the need for some help from, the always handy, [Spring Framework](http://www.springsource.org/).

Spring offers an AMQP solution, with the [Spring AMQP module](http://www.springsource.org/spring-amqp). Since, Spring and RabbitMQ are owned by the same company, it's pretty [clear](http://static.springsource.org/spring-amqp/reference/html/#d0e51) that they're designed to work together. However, in order to get Camel talking to Spring AMQP, we need to roll our own component -- or better yet -- troll [Github](http://www.github.com).

<!--more-->

Github is awesome. The ability to grab someone's code, see its full history, easily fork and modify it -- makes life much easier. Fortunately I found a module that implements a Camel component that can talk to a RabbitMQ broker via RabbitMQ's Java native client and the Spring AMQP implementation. The module was created by Bluelock, you can find it [here](https://github.com/Bluelock/camel-spring-amqp). Additionally awesome, is that Bluelock submitted the module to Maven Central, so it's super easy to get at.

First, let's add some dependencies to our pom.xml file. Add the following to your standard Java and Camel dependencies:

{% highlight xml %}
<dependency>
  <groupId>com.bluelock</groupId>
  <artifactId>camel-spring-amqp</artifactId>
  <version>1.0</version>
</dependency>
<dependency>
  <groupId>javax.jms</groupId>
  <artifactId>jms</artifactId>
  <version>1.1</version>
</dependency>
{% endhighlight %}

Next, we'll need to modify the camel-context.xml (or whatever you've named it) and add the following headers:


{% highlight xml %}
xmlns:rabbit="http://www.springframework.org/schema/rabbit"
xsi:schemaLocation="http://www.springframework.org/schema/rabbit http://www.springframework.org/schema/rabbit/spring-rabbit-1.0.xsd"
{% endhighlight %}

Then the beans:

{% highlight xml %}
<rabbit:connection-factory id="amqpConnectionFactory" />
<rabbit:template id="amqpTemplate" connection-factory="amqpConnectionFactory" 
      message-converter="messageConverter"/>
<rabbit:admin connection-factory="amqpConnectionFactory"/>

<bean id="amqpConnectionFactory" 
      class="org.springframework.amqp.rabbit.connection.CachingConnectionFactory">
    <property name="host" value="localhost"/>
    <property name="port" value="5672"/>
    <property name="username" value="guest"/>
    <property name="password" value="guest"/>
    <property name="virtualHost" value="/"/>
</bean>

<bean id="messageConverter" class="amqp.spring.converter.XStreamConverter"/>
{% endhighlight %}

The first 3 beans set up the rabbit converter, the template and the connection factory, for Spring AMQP. The next two bean definitions set the configuration (default settings) for the RabbitMQ broker and the XStream converter, provided by Bluelock's framework. 

I assume the converter is necessary because, unlike JMS (i.e. when using Camel+ActiveMQ) you can't just serialize Java Objects using Java's built-in serialization -- you have to explicitly specify the method of serializing the data to the broker. 

Now for the fun part; setting up the Camel routes. Being a developer, my killer feature in Camel is being able to define my routes via DSL, in my IDE. Using XML to set these up (i.e. using Mule ESB, WSO2, etc.) is a real pain. Furthermore, you can use Groovy, Scala and of course Java-based DSLs. 

In this example I'm using the Scala DSL for defining the routes. I'll demonstrate two (rather simple) routes producing to and consuming from the broker.

Producer:

{% highlight scala %}
class Producer extends RouteBuilder
{
    handle[ Exception ]
    {
        to("file:tmp/test/error")
    }.maximumRedeliveries(0).handled

    val jaxb = new JaxbDataFormat()
    jaxb.setContextPath("com.artgon.message.schema")

    "file:tmp/test?move=done" ==>
    {
        to("log:FileMover")
        unmarshal(jaxb)
        to("log:ArtMessage")

        split(_.in[ java.util.List[ArtMessage] ])
        {
            to("spring-amqp:myExchange:testQueue:test?type=direct")
        }
    }
}
{% endhighlight %}

Grab an XML file, and unmarshal into a list of ArtMessage objects. Split the list into individual objects and dump them into a queue.

Consumer:

{% highlight scala %}
class Consumer extends RouteBuilder
{
  "spring-amqp:myExchange:testQueue:test?type=direct?concurrentConsumers=8" ==>
  {
      to("log:MQ")
  }
}
{% endhighlight %}

Pretty simple. Grab messages off the queue and log them. 

So that's basically it. I didn't go over [installing](http://www.rabbitmq.com/download.html) RabbitMQ or setting up Maven for compiling Scala and Java [in the same build](http://scala-tools.org/mvnsites/maven-scala-plugin/). However, now you should be able to get going with RabbitMQ and Camel fairly quickly. 

Bluelock has done a great job implementing this module. However, I'm not sure if I'd be fully comfortable throwing this code into a production system because it simply isn't proven yet. On the other hand, it's open source. So if you're willing to fork it and support it yourself, then you're golden. Personally, if I was using Java and Camel for primary development, I would stick with [ActiveMQ](http://activemq.apache.org/). It's designed to work with Camel (i.e. you don't need any of the above setup) and is certainly proven on countless production deployments.
