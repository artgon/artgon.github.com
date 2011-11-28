--- 
layout: post
title: Optimizing Thread Safety in Java
published: true
meta-description: Some tips on how to increase locking performance in Java, using different types of tools.
meta: 
  _edit_last: "2"
  _aioseop_description: This article discusses various ways to optimize the performance of threaded applications in Java by using java.util.concurrent over the synchronized keyword.
tags: 
- atomic
- concurrent
- Java
- JVM
- lock
- multithreaded
- synchronized
- thread safe
- threading
type: post
status: publish
---
There is an obvious trend in recent years in the hardware industry. It seems as though processors are not getting faster, but rather manufacturers are trying to pack as many cores as possible onto the same chip. As a result multi-threaded software has become more and more desired.

The <em>synchronized</em> keyword is the defacto standard for implementing thread safety in Java. However, as of Java 5 and the introduction of the <em>java.util.concurrent</em> package, there are a few more options for Java developers. For a locking solution there's the <em>Lock</em> interface and for a non-locking solution there's a set of <em>Atomic</em> classes.  

I was wondering why there was need for these new classes so I decided to try them out for myself!
<!--more-->

And now for a contrived example. Let's say we want to have a set of threads wanting to get numbers from a global sequence. So first, we need some sort of sequencing singleton:

{% highlight java %}public enum SequenceSingleton
{
	INSTANCE( 1L );

	private long sequence;

	private SequenceSingleton( long seed )
	{
		this.sequence = seed;
	}

	public static SequenceSingleton getInstance()
	{
		return INSTANCE;
	}

	public synchronized long getNextNumber()
	{
		return sequence++;
	}
}{% endhighlight %}

Since we have a singleton that's going to be hit up by a lot of threads we have to be really careful with any sort of state. The above would be the standard implementation for making sure the sequence is incremented by only one thread at a time.

A simple way to test the throughput of this method is the time the longest-running thread takes to complete. We can use the following class to do this:

{% highlight java %}public class NumberGenerator
{
	private static final int AMOUNT_OF_NUMBERS = 100000;
	private static final int NUMBER_OF_THREADS = 20;

	public static void main( String... args )
	{
		for( int i = 0; i < NUMBER_OF_THREADS; i++ )
		{
			Thread numberThread = new Thread( new GetNumbersForAWhile() );
			numberThread.start();
		}
	}

	private static class GetNumbersForAWhile implements Runnable
	{
		@Override
		public void run()
		{
			long startTime = System.currentTimeMillis();
			for( int i = 0; i < AMOUNT_OF_NUMBERS; i++ )
			{
				SequenceSingleton.getInstance().getNextNumber();
			}
			long endTime = System.currentTimeMillis();
			System.out.println( "Took: " + ( endTime - startTime ) + "ms." );
		}
	}
}{% endhighlight %}
With the standard implementation, the longest running thread took 150ms.

Now for the locking implementation using the new <em>java.util.concurrent.locks.Lock</em> interface:

{% highlight java %}public enum SequenceLockSingleton
{
	INSTANCE( 1L );

	private long sequence;

	private Lock lock = new ReentrantLock();

	private SequenceLockSingleton( long seed )
	{
		this.sequence = seed;
	}

	public static SequenceLockSingleton getInstance()
	{
		return INSTANCE;
	}

	public long getNextNumber()
	{
		lock.lock();
		try
		{
			return sequence++;
		}
		finally
		{
			lock.unlock();
		}
	}
}{% endhighlight %}

This time the longest running thread took 122ms. This is a relatively minor improvement but perhaps with a more complex system it may be more noticeable. 

Let's try out the <em>AtomicLong</em> class (which admittedly suits this contrived example extraordinarily well) to solve the thread safety issue:

{% highlight java %}public enum SequenceAtomicSingleton
{
	INSTANCE( 1L );

	private AtomicLong sequence;

	private SequenceAtomicSingleton( long seed )
	{
		this.sequence = new AtomicLong( seed );
	}

	public static SequenceAtomicSingleton getInstance()
	{
		return INSTANCE;
	}

	public long getNextNumber()
	{
		return sequence.getAndAdd( 1L );
	}
}{% endhighlight %}
This time the longest running thread takes only <strong>16ms!!</strong>. This is an improvement of an order of magnitude over the <em>synchronized</em> keyword. 

The reason for this massive improvement is (as Brian Goetz describes <a href="http://www.ibm.com/developerworks/java/library/j-jtp11234/">here</a>) because they are implemented in a wait-free fashion (i.e. lock-free), using a native implementation not available on the JVM.  

There is a set of classes implemented with these atomic variables that give you built-in optimization for much better performance. As you can see, it may be worth trying them out next time you need to do some threading.
