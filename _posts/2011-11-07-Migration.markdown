---
layout: default
title: Migration
---

This is just the *beginning*

>{% highlight java %}
import org.restlet.Context;
import org.restlet.ext.jaxrs.JaxRsApplication;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import javax.ws.rs.core.Application;
import java.util.HashSet;
import java.util.Set;

public class WebServicesApplication extends JaxRsApplication
{
	private static final Logger LOG = LoggerFactory.getLogger( WebServicesApplication.class );

	public WebServicesApplication( Context context )
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

			resourceClasses.add( SampleClass.class );

			return resourceClasses;
		}
	}
}
{% endhighlight %}



