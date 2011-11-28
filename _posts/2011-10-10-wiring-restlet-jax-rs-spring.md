--- 
layout: post
title: Wiring Restlet's JAX-RS with Spring
published: true
meta-description: Some details on how to wire Spring and JAX-RS using the Restlet framework.
meta: 
  _edit_last: "2"
tags: 
- Java
- jax-rs
- restlet
- spring
type: post
status: publish
---
RESTful web services are all the rage these days and Java's no exception. The most mature among the RESTful frameworks in Java, is the <a href="http://restlet.org">Restlet Framework</a>. In fact, the Restlet developers, had a lot of input on the JAX-RS spec and therefore support the standard really well.

However, one of the issues I had when starting with Restlets (using JAX-RS), was getting them to work with Spring. Restlets alone work well with Spring, but using the JAX-RS extension was slightly more challenging, as you have to create your own method of injecting objects into the framework.
<!--more-->

Here is a sample application class:

{% highlight java %}
import org.restlet.Context;
import org.restlet.ext.jaxrs.JaxRsApplication;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import javax.ws.rs.core.Application;
import java.util.HashSet;
import java.util.Set;

public class SampleWebServicesApplication extends JaxRsApplication
{
	private static final Logger LOG = LoggerFactory.getLogger( SampleWebServicesApplication.class );

	public MtsWebServicesApplication( Context context )
	{
		super( context );
		this.add( new JaxRsApplicationImpl() );

		LOG.info( "Restlet application initialized." );
	}

	/**
	 * This class defines the JAX-RS classes that the application needs
	 * to instantiate.
	 */
	private static class JaxRsApplicationImpl extends Application
	{
		/**
		 * Standard JAX-RS method to define the resources of the application
		 *
		 * @return a set of classes (resources)
		 */
		@Override
		public Set<Class<?>> getClasses()
		{
			LOG.info( "Initializing JAX-RS application..." );

			// define the set of JAX RS classes
			Set<Class<?>> resourceClasses = new HashSet<Class<?>>();

			resourceClasses.add( SampleResource.class );

			return resourceClasses;
		}
	}
}
{% endhighlight %}

_Note: I added the import statements so it would be clear I'm using the JAX-RS extension for Restlets_

This is all fine and good. This is what you would do if you were just creating a regular JAX-RS application using Restlets. However, it's not so simple if you want to use Spring beans. Notice, I'm adding the class I want to bind as my resource via its static classname. Generally, you can't reference Spring beans this way without accessing the internals and that's what we'll have to do here. 

The way to do this is by overriding the Restlet ServerServlet with your own SpringServerServlet class:
{% highlight java %}
public class SpringRestletServlet extends ServerServlet
{
	private static final Logger LOG = LoggerFactory.getLogger( SpringRestletServlet.class );

	/**
	 * Override the createApplication method to inject Spring objects
	 *
	 * @param context - Restlet context
	 * @return - application instance
	 */
	@Override
	public Application createApplication( Context context )
	{
		JaxRsApplication application = (JaxRsApplication) super.createApplication( context );

		// set up spring to be the object creator
		application.setObjectFactory(
				new SpringObjectFactory( getWebApplicationContext().getAutowireCapableBeanFactory() )
		);

		return application;
	}

	private static class SpringObjectFactory implements ObjectFactory
	{
		private final AutowireCapableBeanFactory beanFactory;

		public SpringObjectFactory( AutowireCapableBeanFactory beanFactory )
		{
			this.beanFactory = beanFactory;
		}

		@Override
		public <T> T getInstance( Class<T> jaxRsClass ) throws InstantiateException
		{
			LOG.info( "Wiring JAX-RS class [" + jaxRsClass + "] using Spring." );

			return beanFactory.createBean( jaxRsClass );
		}
	}

	private WebApplicationContext getWebApplicationContext()
	{
		return WebApplicationContextUtils.getRequiredWebApplicationContext( getServletContext() );
	}
}
{% endhighlight %}

The JAX-RS extension allows you to set an object factory, and in this case we're overriding the object factory to use Spring's AutowireCapableBeanFactory. This way, we can specify the static classname and get a valid Spring bean in return. 

One last thing to add is that you'll have to use this SpringServerServlet in your web.xml instead of the standard ServerServlet.

Hopefully that saves you some time!
