--- 
layout: post
title: "JAXB Continued: Generating Schema from Object Model"
published: true
meta: 
  _edit_last: "2"
tags: 
- generate
- Java
- jaxb
- objects
- schema
- unit test
- xml
- xsd
type: post
status: publish
---
As I mentioned in the <a href="http://blog.gonigberg.com/2010/04/21/getting-started-with-jaxb/">previous post</a>, I like to start with my Java object model first and then bind the XML as required. There are obviously countless opinions on both sides of this argument. The two sides of the argument are <em>contract-first</em> and <em>contract-last</em>. As you can probably guess this refers to whether we define the XSD before we create the Java objects, or after we already have our object model coded up.

<a href="http://static.springsource.org/spring-ws/sites/1.5/">Spring WS</a> makes the most convincing argument for <em>contract-first</em> design. In their <a href="http://static.springsource.org/spring-ws/sites/1.5/reference/html/why-contract-first.html">reference documentation</a> they defend their decision to only offer the <em>contract-first</em> option. They make some good points, and I agree, for very complex schemas it may be better to have a contract first so that both parties can have a way to validate their generated results. However, for a simpler model, as I'm going to show you, it's very quick and painless to have JAXB generate a schema for you.

<!--more-->

Let's assume we're using the same Prison schema:
{% highlight java %}@XmlRootElement
public class Prison
{
	private Cell cell;
	private Guard guard;

        ...
}

public class Cell
{
	private String id;
	private Inmate inmate;

        ...
}

public class Inmate
{
	private String name;
	private String id;
	private String sentence;
	private String description;
	private String history;

        ...
}

public class Guard
{
	private String name;
	private String assignment;

        ...
}{% endhighlight %}
I'm going to re-iterate again that it is a very simple schema, but that's not really the point of this post. Now comes the <strong>COOL</strong> part! We generate the schema from the code by writing a quick unit test:
{% highlight java %}public void testSchema() throws JAXBException, IOException
{
	// grab the context
	JAXBContext context = JAXBContext.newInstance( Prison.class );

	final List results = new ArrayList();

	// generate the schema
	context.generateSchema(
            // need to define a SchemaOutputResolver to store to
			new SchemaOutputResolver()
			{
				@Override
				public Result createOutput( String ns, String file )
						throws IOException
				{
                    // save the schema to the list
					DOMResult result = new DOMResult();
					result.setSystemId( file );
					results.add( result );
					return result;
				}
			} );

    // output schema via System.out
	DOMResult domResult = results.get( 0 );
	Document doc = (Document) domResult.getNode();
	OutputFormat format = new OutputFormat( doc );
	format.setIndenting( true );
	XMLSerializer serializer = new XMLSerializer( System.out, format );
	serializer.serialize( doc );
}{% endhighlight %}
As you can see, there a couple of tricks to this -- credit given to <a href="http://technology.amis.nl/blog/2046/using-the-javaxxmlbind-annotations-to-convert-java-objects-to-xml-and-xsd">Amis blog</a>. The first is that you'll need to implement the <em>SchemaOutputResolver</em> interface, done here with an anonymous class that creates the DOMResult. The second is that you'll need to output the result to System.out (and format it nicely of course!). Here is the generated result:

{% highlight xml %}
<xs:schema version="1.0" xmlns:xs="http://www.w3.org/2001/XMLSchema">
    <xs:element name="prison" type="prison"/>
    <xs:complexType name="prison">
        <xs:sequence>
            <xs:element minOccurs="0" name="cell" type="cell"/>
            <xs:element minOccurs="0" name="guard" type="guard"/>
        </xs:sequence>
    </xs:complexType>
    <xs:complexType name="cell">
        <xs:sequence>
            <xs:element minOccurs="0" name="id" type="xs:string"/>
            <xs:element minOccurs="0" name="inmate" type="inmate"/>
        </xs:sequence>
    </xs:complexType>
    <xs:complexType name="inmate">
        <xs:sequence>
            <xs:element minOccurs="0" name="description" type="xs:string"/>
            <xs:element minOccurs="0" name="history" type="xs:string"/>
            <xs:element minOccurs="0" name="id" type="xs:string"/>
            <xs:element minOccurs="0" name="name" type="xs:string"/>
            <xs:element minOccurs="0" name="sentence" type="xs:string"/>
        </xs:sequence>
    </xs:complexType>
    <xs:complexType name="guard">
        <xs:sequence>
            <xs:element minOccurs="0" name="assignment" type="xs:string"/>
            <xs:element minOccurs="0" name="name" type="xs:string"/>
        </xs:sequence>
    </xs:complexType>
</xs:schema>
{% endhighlight %}

Pretty cool, right? Now you have an XSD schema! Obviously, the schema generator can't guess the constraints for each field just from the Java code, so you'll need to still modify this XSD. However, this can save you some work and will give you a great starting point for your XSD. 

I'd love to hear some more feedback on the <em>contract-first</em> vs. <em>contract-last</em> last debate, so leave a comment!
