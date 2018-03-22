---
title: Network services management
layout: default
---
A common problem in managing enterprise services is a failure in one or more services can cause the overall system to get wedged, due to many dependences between different services and server. After such a failure cascade, the network can only become fully functional by restarting the servers and services in the correct order. Network services management can coordinate the order of shutdown and restarts.

Using Protelis and the principles of aggregate programming, it is straightforward to write a daemon to monitor each service and communicate with other services to execute a coordinated shutdown and restart. We illustrate how dependency-directed coordination can be implemented with very little code.

<b>Code samples and illustrations from a simulation:</b>

![diagram of seven servers with service dependency arrows](/images/dependent-services-network.png)

This example scenario features an enterprise network for a small company, with dependencies between two key databases and the internal and external servers running web applications. The scenario was implemented on a network of EmuLab servers, with the services emulated as simple query-response networking programs in Java that entered a "hung" state either by being externally triggered to crash or after their queries began to consistently fail.

Each service was wrapped with an embedded Protelis execution engine, which was interfaced with the services by a small piece of monitoring glue code that inserted environment variables containing an indentifier for the <code>serviceID</code> running on that server, a tuple of identifier for <code>dependencies</code>, and the current <code>managedServiceStatus</code> of <code>stop</code>, <code>starting</code>, <code>run</code>, <code>stopping</code>, or <code>hung</code>. The glue code also provided <code>stopService</code> and <code>startService</code> methods to send signals to the service, track interactions between the services in order to maintain the set of neighbors for Protelis, and allowed an external monitoring applications to attach and receive status reports.

![diagram of seven servers with blue, red, and green status for run, hung, and stop](/images/restart.png)

In this screenshot of the emulated network of services on machines, we see that the supplies database has crashed, triggering a graceful shutdown cascade of dependent services. Upon restart of the supplies database, the rest of the services would automatically restart in the correct order, if each service was running the following Protelis network management code.

These variables track the services that should be reachable (<code>nbr_set</code>), the services that are needed but are not reachable (<code>nbr_missing</code>), the services that are required due to dependencies (<code>nbr_required</code>), and the nearby services that are down (<code>nbr_down</code>).

<pre>
<code style="color:purple">let</code><code style="color:green"> nbr_set </code><code>= </code><code style="color:purple">unionHood</code><code>(</code><code style="color:purple">nbr</code><code>([</code><code style="color:blue">serviceID</code><code>]));</code>
<code style="color:purple">let</code><code style="color:green"> nbr_missing </code><code>= </code><code style="color:blue">dependencies</code><code>.</code><code style="color:blue">subtract</code><code>(</code><code style="color:green">nbr_set</code><code>);</code>
<code style="color:purple">let</code><code style="color:green"> nbr_required </code><code>= </code><code style="color:blue">#contains</code><code>(</code><code style="color:blue">dependencies</code><code>, </code><code style="color:purple">nbr</code><code>(</code><code style="color:blue">serviceID</code><code>));</code>
<code style="color:purple">let</code><code style="color:green"> nbr_down </code><code>= </code><code style="color:purple">nbr</code><code>(</code><code style="color:blue">managedServiceStatus</code><code>=="hung" || </code><code style="color:blue">managedServiceStatus</code><code>=="stop");</code>
</pre>

The variable <code>problem</code> tracks whether this service is currently safe to run.

<pre>
<code style="color:purple">let</code><code style="color:green"> problem </code><code>= </code><code style="color:purple">anyHood</code><code>(</code><code style="color:green">nbr_down </code><code>&& </code><code style="color:green">nbr_required</code><code>) || !</code><code style="color:green">nbr_missing</code><code>.</code><code style="color:blue">isEmpty</code><code>();</code>
</pre>

This code takes this service down or up, depending on if it is safe to run.

<pre>
<code style="color:purple">if</code><code>(</code><code style="color:blue">managedServiceStatus</code><code>=="run" && </code><code style="color:green">problem</code><code>) {</code>
<code style="color:blue">  #stopProcess</code><code>(</code><code style="color:blue">managedService</code><code>);</code>
<code>} </code><code style="color:purple">else</code><code>  {</code>
<code style="color:purple">  if</code><code>(</code><code style="color:blue">managedServiceStatus</code><code>=="stop" && !</code><code style="color:green">problem</code><code>) {</code>
<code style="color:blue">    #startProcess</code><code>(</code><code style="color:blue">managedService</code><code>);</code>
<code>  } </code><code style="color:purple">else</code><code>  {</code>
<code style="color:blue">    managedServiceStatus</code><code>;</code>
<code>  }</code>
<code>}</code>
</pre>

For more information, see <a href="http://jakebeal.com/Publications/SAC2015-Protelis.pdf">Protelis: Practical Aggregate Programming</a>, Danilo Pianini, Mirko Viroli, Jacob Beal, ACM Symposium on Applied Computing 2015, April 2015.
