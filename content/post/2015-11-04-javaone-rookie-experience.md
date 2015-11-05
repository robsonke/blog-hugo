---
author: Rob
date: 2015-11-04 20:00:23+00:00
layout: post
slug: javaone-rookie-experience
title: A JavaOne rookie experience
categories:
- java
--- 

Being part of JavaOne this year and including oracle open world as well was pretty special.  As a Java developer you know that JavaOne was always the conference to go to. With Oracle acquiring Sun this has changed according to people who experienced both. Nevertheless it was pretty excited and being there on invitation of Oracle is something I'm oracle grateful for. With an a absense of 20 years in the US being back here was another good thing. I enjoyed the politeness and kindness of the people. With a short touristic visit of San Francisco it was made complete. 

I decided to write a short blog on my experiences and my opinions on what I heard. There were many tracks to follow but looking at the subjects, the scope was pretty narrow. The following things were popular:
* Microservices
* Docker
* Lambas/streams
* Cloud computing
* Continuous delivery / integration

This didn't matter to me anyway as this has my interest too. And maybe not a total coincidence, they all cover the same benefits / promises:
* Auto scaling
* Isolation
* Easy and often deployment
* Cloud computing

This year Java reached the age of 20 years, pretty long for a language that shad its ups and downs but is, according to my opinion, fully alive and kicking. All cloud platforms support the JVM and on top of that they rollout support for other JVM and non JVM based languages as Ruby, Scala, Node, PHP, etcetera. 

## Microservices 
As being a fan of this relatively new architecture, I had some annoying moments in tracks covering this subject.  First of all it's brought as something revolutionary and sparkling new which is pretty untrue. I remember the coming up of SOA architecture and micro services is to me more version two of this approach. Being more focused on REST interfaces and with that statelessness. Where SOA was more based on older protocols as soap. The second thing was the way people want to bound/restrict micro services into certain boxes. With mentioning statements as "a service should not extend 2000 lines of code" I wondered how those people thing about breaking up the big monolithic legacy applications... You end up with a horrible product facing nasty issues as shared domain objects and transactions over a bunch of services. If you get s nice application with keeping such rules in mind, your monolithic application wasn't that bad either or you have a unique use case. 

So for myself I took a few conclusions. I hate the word micro from today and I'll call it services from now on, although isn't that what we already do since the last years? Kidding of course, nothing with the wrong micro but the main issue is that nobody should tell you how micro micro is. That can be different per use case. So what does put a service in a box then? I'd say you never put more then one business case in a service. And yes, that could mean that you end up with a service with easily 10.000 lines but maybe also with 500 lines of code. It's the same and simple rule of how you decide what to put in one class and what not, only then at a higher level. 

## Docker, cloud, continuous deployment strategies
Combining these subjects into one as they partially rely on each other.  Docker is around for a while but you get the feeling that it's now the one and only solution that everybody uses or wants to start using. That combined with cloud platforms, you get a powerful set of tools that makes deployment pretty easy. Nevertheless, I still feel that were pretty at the beginning. All tools and platforms around are popular but they are all lacking somewhere, either not providing a full set of features or being not production ready. I've seen some neat solutions but they required good knowledge and always some custom scripting or even developing extra applications. But with the popularity this must be changing soon and there are already, but often licensed, solutions available. It's something we don't use yet, but I'm looking forward to it!

## Typescript
And as I was already convinced by the use of Typescript for JavaScript development, Sander Mak's session proved that once more. It makes the language better, more safe and a more trusted place for Java developers. That combined with the fact that we today can start using the new ES2015 standards while the majority of browsers will need their time to support it. I'd call that the end of nightmare being happy with new feautures you cannot use. 
Learned also one new setting for tsconfig.json to become strict on any missing type:
```json
"compilerOptions": {
  "noImplicitAny": true
}
```

My homework for now is to dive deeper into technologies I found interesting, mainly:
* Kubernetes
* Open shift
* Cloud foundry
* New relic
* Cloud bees

The oracle appreciation event with a performance of Elton John and beck made it next to learning full week, also even more fun. And I enjoyed meeting several Dutch people on the NLJUG after party. Feeling a true member now with a NLJUG shirt even though I'm missing JFall this year :). 

Slides of sessions I found interesting and were published already:
* http://www.slideshare.net/SanderMak/typescript-coding-javascript-without-the-pain
* http://www.slideshare.net/ArjanSchaaf/zero-downtimejavadeploymentswithdockerandkubernetes

