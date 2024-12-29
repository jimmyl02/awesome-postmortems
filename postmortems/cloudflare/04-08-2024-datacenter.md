[Source](https://blog.cloudflare.com/major-data-center-power-failure-again-cloudflare-code-orange-tested)

Here's a post we never thought we'd need to write: less than five months after one of our major data centers lost power, it happened again to the exact same data center. That sucks and, if you're thinking "why do they keep using this facility??," I don't blame you. We're thinking the same thing. But, here's the thing, while a lot may not have changed at the data center, a lot changed over those five months at Cloudflare. So, while five months ago a major data center going offline was really painful, this time it was much less so.

This is a little bit about how a high availability data center lost power for the second time in five months. But, more so, it's the story of how our team worked to ensure that even if one of our critical data centers lost power it wouldn't impact our customers.

On November 2, 2023, one of our critical facilities in the Portland, Oregon region lost power for an extended period of time. It happened because of a cascading series of faults that appears to have been caused by maintenance by the electrical grid provider, climaxing with a ground fault at the facility, and was made worse by a series of unfortunate incidents that prevented the facility from getting back online in a timely fashion.

If you want to read all the gory details, they're available [here](https://blog.cloudflare.com/post-mortem-on-cloudflare-control-plane-and-analytics-outage/).

It's painful whenever a data center has a complete loss of power, but it's something that we were supposed to expect. Unfortunately, in spite of that expectation, we hadn't enforced a number of requirements on our products that would ensure they continued running in spite of a major failure.

That was a mistake we were never going to allow to happen again.

### Code Orange

The incident was painful enough that we declared what we called Code Orange. We borrowed the idea from Google which, when they have an existential threat to their business, reportedly declares a Code Yellow or Code Red. Our logo is orange, so we altered the formula a bit.

Our conception of Code Orange was that the person who led the incident, in this case our SVP of Technical Operations, Jeremy Hartman, would be empowered to charge any engineer on our team to work on what he deemed the highest priority project. (Unless we declared a Code Red, which we actually ended up doing due to a hacking incident, and which would then take even higher priority. If you're interested, you can read more about that [here](https://blog.cloudflare.com/thanksgiving-2023-security-incident/).)

After getting through the immediate incident, Jeremy quickly triaged the most important work that needed to be done in order to ensure we'd be highly available even in the case of another catastrophic failure of a major data center facility. And the team got to work.

![](https://blog.cloudflare.com/content/images/2024/04/image2-15.png)

### How'd we do?

We didn’t expect such an extensive real-world test so quickly, but the universe works in mysterious ways. On Tuesday, March 26, 2024, — just shy of five months after the initial incident — the same facility had another major power outage. Below, we'll get into what caused the outage this time, but what is most important is that it provided a perfect test for the work our team had done under Code Orange. So, what were the results?

First, let’s revisit what functions the Portland data centers at Cloudflare provide. As described in the November 2, 2023, [post](https://blog.cloudflare.com/post-mortem-on-cloudflare-control-plane-and-analytics-outage/), the control plane of Cloudflare primarily consists of the customer-facing interface for all of our services including our website and API. Additionally, the underlying services that provide the Analytics and Logging pipelines are primarily served from these facilities.

Just like in November 2023, we were alerted immediately that we had lost connectivity to our PDX01 data center. Unlike in November, we very quickly knew with certainty that we had once again lost all power, putting us in the exact same situation as five months prior. We also knew, based on a successful internal cut test in February, how our systems should react. We had spent months preparing, updating countless systems and activating huge amounts of network and server capacity, culminating with a test to prove the work was having the intended effect, which in this case was an automatic failover to the redundant facilities.

Our Control Plane consists of hundreds of internal services, and the expectation is that when we lose one of the three critical data centers in Portland, these services continue to operate normally in the remaining two facilities, and we continue to operate primarily in Portland. We have the capability to fail over to our European data centers in case our Portland centers are completely unavailable. However, that is a secondary option, and not something we pursue immediately.

On March 26, 2024, at 14:58 UTC, PDX01 lost power and our systems began to react. By 15:05 UTC, our APIs and Dashboards were operating normally, all without human intervention. Our primary focus over the past few months has been to make sure that our customers would still be able to configure and operate their Cloudflare services in case of a similar outage. There were a few specific services that required human intervention and therefore took a bit longer to recover, however the primary interface mechanism was operating as expected.

To put a finer point on this, during the November 2, 2023, incident the following services had at least six hours of control plane downtime, with several of them functionally degraded for days.

API and Dashboard  
Zero Trust  
Magic Transit  
SSL  
SSL for SaaS  
Workers  
KV  
Waiting Room  
Load Balancing  
Zero Trust Gateway  
Access  
Pages  
Stream  
Images

During the March 26, 2024, incident, all of these services were up and running within minutes of the power failure, and many of them did not experience any impact at all during the failover.

The data plane, which handles the traffic that Cloudflare customers pass through our data centers in over 300 cities worldwide, was not impacted.

Our Analytics platform, which provides a view into customer traffic, was impacted and wasn’t fully restored until later that day. This was expected behavior as the Analytics platform is reliant on the PDX01 data center. Just like the Control Plane work, we began building new Analytics capacity immediately after the November 2, 2023, incident. However, the scale of the work requires that it will take a bit more time to complete. We have been working as fast as we can to remove this dependency, and we expect to complete this work in the near future.

Once we had validated the functionality of our Control Plane services, we were faced yet again with the cold start of a very large data center. This activity took roughly 72 hours in November 2023, but this time around we were able to complete this in roughly 10 hours. There is still work to be done to make that even faster in the future, and we will continue to refine our procedures in case we have a similar incident in the future.

![](https://blog.cloudflare.com/content/images/2024/04/Incident-inspection.png)

### How did we get here?

As mentioned above, the power outage event from last November led us to introduce Code Orange, a process where we shift most or all engineering resources to addressing the issue at hand when there’s a significant event or crisis. Over the past five months, we shifted all non-critical engineering functions to focusing on ensuring high reliability of our control plane.

Teams across our engineering departments rallied to ensure our systems would be more resilient in the face of a similar failure in the future. Though the March 26, 2024, incident was unexpected, it was something we’d been preparing for.

The most obvious difference is the speed at which the control plane and APIs regained service. Without human intervention, the ability to log in and make changes to Cloudflare configuration was possible seven minutes after PDX01 was lost. This is due to our efforts to move all of our configuration databases to a Highly Available (HA) topology, and pre-provision enough capacity that we could absorb the capacity loss. More than 100 databases across over 20 different database clusters simultaneously failed out of the affected facility and restored service automatically. This was actually the culmination of over a year’s worth of work, and we make sure we prove our ability to failover properly with weekly tests.

Another significant improvement is the updates to our Logpush infrastructure. In November 2023, the loss of the PDX01 datacenter meant that we were unable to push logs to our customers. During Code Orange, we invested in making the Logpush infrastructure HA in Portland, and additionally created an active failover option in Amsterdam. Logpush took advantage of our massively expanded Kubernetes cluster that spans all of our Portland facilities and provides a seamless way for service owners to deploy HA compliant services that have resiliency baked in. In fact, during our February chaos exercise, we found a flaw in our Portland HA deployment, but customers were not impacted because the Amsterdam Logpush infrastructure took over successfully. During this event, we saw that the fixes we’d made since then worked, and we were able to push logs from the Portland region.

A number of other improvements in our Stream and Zero Trust products resulted in little to no impact to their operation. Our Stream products, which use a lot of compute resources to transcode videos, were able to seamlessly hand off to our Amsterdam facility to continue operations. Teams were given specific availability targets for the services and were provided several options to achieve those targets. Stream is a good example of a service that chose a different resiliency architecture but was able to seamlessly deliver their service during this outage. Zero Trust, which was also impacted in November 2023, has since moved the vast majority of its functionally to our hundreds of data centers, which kept working seamlessly throughout this event. Ultimately this is the strategy we are pushing all Cloudflare products to adopt as our data centers in [over 300 cities worldwide](https://www.cloudflare.com/network) provide the highest level of availability possible.

![](https://blog.cloudflare.com/content/images/2024/04/image1-12.png)

### What happened to the power in the data center?

On March 26, 2024, at 14:58 UTC, PDX01 experienced a total loss of power to Cloudflare’s physical infrastructure following a reportedly simultaneous failure of four Flexential-owned and operated switchboards serving all of Cloudflare’s cages. This meant both primary and redundant power paths were deactivated across the entire environment. During the Flexential investigation, engineers focused on a set of equipment known as Circuit Switch Boards, or CSBs. CSBs are likened to an electrical panel board, consisting of a main input circuit breaker and series of smaller output breakers. Flexential engineers reported that infrastructure upstream of the CSBs (power feed, generator, UPS & PDU/transformer) was not impacted and continued to act normally. Similarly, infrastructure downstream from the CSBs such as Remote Power Panels and connected switchgear was not impacted – thus implying the outage was isolated to the CSBs themselves.

Initial assessment of the root cause of Flexential’s CSB failures points to incorrectly set breaker coordination settings within the four CSBs as one contributing factor. Trip settings which are too restrictive can result in overly sensitive overcurrent protection and the potential nuisance tripping of devices. In our case, Flexential’s breaker settings within the four CSBs were reportedly too low in relation to the downstream provisioned power capacities. When one or more of these breakers tripped, a cascading failure of the remaining active CSB boards resulted, thus causing a total loss of power serving Cloudflare’s cage and others on the shared infrastructure. During the triage of the incident, we were told that the Flexential facilities team noticed the incorrect trip settings, reset the CSBs and adjusted them to the expected values, enabling our team to power up our servers in a staged and controlled fashion. We do not know when these settings were established – typically, these would be set/adjusted as part of a data center commissioning process and/or breaker coordination study before customer critical loads are installed.

![](https://blog.cloudflare.com/content/images/2024/04/Incident-inspection-3.png)

### What’s next?

Our top priority is completing the resilience program for our Analytics platform. Analytics aren’t simply pretty charts in a dashboard. When you want to check the status of attacks, activities a firewall is blocking, or even the status of Cloudflare Tunnels - you need analytics. We have evidence that the resiliency pattern we are adopting works as expected, so this remains our primary focus, and we will progress as quickly as possible.

There were some services that still required manual intervention to properly recover, and we have collected data and action items for each of them to ensure that further manual action is not required. We will continue to use production cut tests to prove all of these changes and enhancements provide the resiliency that our customers expect.

We will continue to work with Flexential on follow-up activities to expand our understanding of their operational and review procedures to the greatest extent possible. While this incident was limited to a single facility, we will turn this exercise into a process that ensures we have a similar view into all of our critical data center facilities.

Once again, we are very sorry for the impact to our customers, particularly those that rely on the Analytics engine who were unable to access that product feature during the incident. Our work over the past four months has yielded the results that we expected, and we will stay absolutely focused on completing the remaining body of work.