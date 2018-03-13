---
title: Crowd danger estimation and warning
layout: default
---
Any time an event or attraction draws a crowd, individuals may continue to move into the most crowded areas and inadvertently obstruct egress, creating dangerously dense-packed locations vulnerable to stampedes. In addition to the motivation of avoiding danger, people may also want to avoid highly congested spots because the experience is unpleasant. Distributed sensing and computing can help solve this problem.

Using Protelis and the principles of aggregate programming, it is straightforward to write a scalable and robust distributed application to detect severe crowding and help individuals navigate around highly congested areas. We illustrate the use of several core Protelis operators and built-in functions.

<b>Code samples and illustrations from a simulation:</b>

Using some of Protelis’s special operators and syntax (purple), a small number of functions (blue) can be defined and applied to variables (green). A crowd safety service can use information about the number of nearby devices to issue warnings about areas with dangerous crowding levels. 

The function <code>dangerousDensity</code> flags whether a location has a high or low danger. The function determines danger by estimating the local density of people, using the values of <i>p</i> (proportion of people with a participating device running this app), <i>r</i> (the radius of interest), and checking large groups of people (>300) against a density danger cut-off value of 2.17 people per square meter.

<pre>
<code style="color:purple">def</code><code style="color:blue"> dangerousDensity</code><code>(</code><code style="color:green">p</code><code>, </code><code style="color:green">r</code><code>) {</code>
<code style="color:purple">  let</code><code style="color:green"> mr</code><code> = </code><code style="color:blue"> managementRegions</code><code>(</code><code style="color:green">r</code><code>*2, () -> { </code><code style="color:purple">nbrRange</code><code> });</code>
<code style="color:purple">  let</code><code style="color:green"> danger</code><code> = </code><code style="color:blue"> average</code><code>(</code><code style="color:green">mr</code><code>, </code><code style="color:blue">densityEst</code><code>(</code><code style="color:green">p</code><code>, </code><code style="color:green">r</code><code>)) > 2.17</code><code style="color:purple"> &&</code><code style="color:blue"> summarize</code><code>(</code><code style="color:green">mr</code><code>, </code><code style="color:green">sum</code><code>, 1 / </code><code style="color:green">p</code><code>, 0 ) > 300;</code>
<code style="color:purple">  if</code><code>(</code><code style="color:green">danger</code><code>) { high } </code><code style="color:purple">else</code><code> { low }</code>
<code>}</code>
</pre>

The function <code>crowdTracking</code> checks for a dangerous density in crowded areas, where 1.08 people per square meter is used as the cut-off to define crowded.

<pre>
<code style="color:purple">def</code><code style="color:blue"> crowdTracking</code><code>(</code><code style="color:green">p</code><code>, </code><code style="color:green">r</code><code>, </code><code style="color:green">t</code><code>) {</code>
<code style="color:purple">  let</code><code style="color:green"> crowdRgn</code><code> = </code><code style="color:blue"> recentTrue</code><code>(</code><code style="color:blue">densityEst</code><code>(</code><code style="color:green">p</code><code>, </code><code style="color:green">r</code><code>) > 1.08, </code><code style="color:green">t</code><code>);</code>
<code style="color:purple">  if</code><code>(</code><code style="color:green">crowdRgn</code><code>) { </code><code style="color:blue">densityEst</code><code>(</code><code style="color:green">p</code><code>, </code><code style="color:green">r</code><code>) } </code><code style="color:purple">else</code><code> { none }</code>
<code>}</code>
</pre>

The function <code>crowdWarning</code> alerts individuals who are near dangerously crowded spots.

<pre>
<code style="color:purple">def</code><code style="color:blue"> crowdWarning</code><code>(</code><code style="color:green">p</code><code>, </code><code style="color:green">r</code><code>, </code><code style="color:green">warn</code><code>, </code><code style="color:green">t</code><code>) {</code>
<code style="color:blue">  distanceTo</code><code>(</code><code style="color:blue">crowdTracking</code><code>(</code><code style="color:green">p</code><code>, </code><code style="color:green">r</code><code>, </code><code style="color:green">t</code><code>) == high) < </code><code style="color:green">warn</code>
<code>}</code>
</pre>

![map with green, yellow, and red dots](/images/marathon.png)

In this simulation of the 2013 Vienna marathon, dots represent devices running the crowd safety service. Red dots are devices in locations with potentially dangerous crowd density and yellow dots are devices providing crowd density warnings.

![map with blue, yellow, and red dots](/images/crowd-management.jpg)

In this simulation of public events near the Boston waterfront, dots represent devices running the crowd safety service. Red dots are devices in locations with potentially dangerous crowd density and yellow dots are devices providing crowd density warnings.

For more information, see <a href="http://web.mit.edu/jakebeal/www/Publications/QUANTICOL16-AggregateProgramming.pdf">Aggregate Programming: From Foundations to Applications</a>, Jacob Beal, Mirko Viroli, In: Bernardo M., De Nicola R., Hillston J. (eds) Formal Methods for the Quantitative Evaluation of Collective Adaptive Systems. SFM 2016. Lecture Notes in Computer Science, vol 9700. Springer, Cham
