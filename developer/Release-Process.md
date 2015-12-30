---
layout: default
title: CAS - Release Process
---

# Release Process
This page documents the steps that a release engineer should take for cutting a CAS server release.

##Environment Review

- Set up your environment:
	- Load your SSH key and ensure this SSH key is also referenced in Github.
	- Load your `~/.gradle/gradle.properties` file with the following:

{% highlight bash %}
signing.keyId=
signing.password=
signing.secretKeyRingFile=
sonatypeUsername=
sonatypePassword=
{% endhighlight %}

- Checkout the CAS project: `git clone git@github.com:Jasig/cas.git cas-server`

##Preparing the Release

- Create a branch for the release version, if necessary (i.e `4.2.x`).
- Switch to the release branch. 
- In the project's `gradle.properties`, change the project version to the release version (i.e. `4.2.0-RC1`)
- Build the project using the following command:

{% highlight bash %}
./gradlew clean build -x test --parallel -DskipCheckstyle=true -DskipFindbugs=true
{% endhighlight %}

- Release the project using the following commands:

{% highlight bash %}
./gradlew uploadArchives -DpublishReleases=true
{% endhighlight %}

- In the project's `gradle.properties`, change the project version to the *next* release version (i.e. `4.2.0-RC2`) 
- Create a tag for the released version and push the tag to the upstream Jasig repository. (i.e. `v4.2.0-RC1`).
- Switch to the `master` branch and in the project's `gradle.properties`, change the project version to the *next* development version (i.e. `4.3.0-SNAPSHOT`). Push your changes to the upstream Jasig repository. 

##Performing the Release

Follow the process for [deploying artifacts to Maven Central](https://wiki.jasig.org/display/JCH/Deploying+Maven+Artifacts) via Sonatype OSS repository.  

- Log into [https://oss.sonatype.org](https://oss.sonatype.org).
- Find the staged repository for CAS artifacts
- "Close" the repository.
- "Release" the repository.  Both c and d should be accompanied by email confirmation.

## Housekeeping

- Close [the milestone](https://github.com/Jasig/cas/milestones) for this release.
- Find [the release](https://github.com/Jasig/cas/releases) that is mapped to the released tag, update the description with the list of resolved/fixed issues and publish it as released. 
- Mark the release as pre-release, when releasing RC versions of the project. 

To generate the changelog and release notes, use the below steps:

1. Download and install [this tool](https://github.com/lalitkapoor/github-changes)
2. Generate a github access token [here](https://github.com/settings/tokens)
3. Execute the following command:

{% highlight bash %}
github-changes -o Jasig -r cas -b x.y.z -k <TOKEN> -a --use-commit-body
{% endhighlight %}

Note that `x.y.z` is the name of the branch that is released. The output will be saved in `ChangeLog.md` file. Comb
through the file, edit, format and paste the final content under the release tag. 

- Send an announcement message to @cas-announce, @cas-user and @cas-dev mailing lists. A template follows:

{% highlight bash %}
CAS Community,

CAS x.y.z is available for testing and evaluation. We encourage adopters to grab 
this release from Maven Central, integrate into your environment, and provide feedback.

Regards,
John Smith

{% endhighlight %}

##Update Maven Overlay
Update the following overlay projects to point to the newly released CAS version. This task is only relevant when dealing with GA releases.

- [CAS WebApp Overlay](https://github.com/Jasig/cas-overlay-template)
- [CAS Services Management WebApp Overlay](https://github.com/Jasig/cas-services-management-overlay)


##Docker Image
Release a new CAS [Docker image](https://github.com/Jasig/cas/tree/dockerized-caswebapp).
This task is only relevant when dealing with GA releases.
