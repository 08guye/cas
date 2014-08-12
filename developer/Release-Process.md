---
layout: default
title: CAS - Release Process
---


# Release Process
This page documents the steps that a release engineer should take for cutting a CAS server release.


##Environment Review

- Sanity test pending CAS release by testing protocol and major features.
- Ensure criteria for a release has been met and that there are no outstanding JIRA issues, etc.
- Ensure all outstanding changes are committed.
- Ensure all tests pass on CI 
- Set up your environment :
	- Load your SSH key on your local computer and ensure this SSH key is also referenced in Github
	- Ensure your `settings.xml` file has the Sonatype repository defined with the appropriate credentials:
	
{% highlight xml %}
<servers>
	<server>
	  <id>sonatype-nexus-snapshots</id>
	  <username>uid</username>
	  <password>psw</password>
	</server>
	<server>
	  <id>sonatype-nexus-staging</id>
	  <username>uid</username>
	  <password>psw</password>
	</server>
</servers>
{% endhighlight %}
	
- Checkout the CAS project :
{% highlight bash %}
mkdir casrelease
cd casrelease
git clone git@github.com:Jasig/cas.git
{% endhighlight %}

- Ensure licensing conformity before release by executing the following goals from the project root:
{% highlight bash %}
mvn -o notice:check
mvn -o license:check
mvn -o checkstyle:check
{% endhighlight %}

If either of the above, the `notice:generate` and `license:format` goals [may be used](https://wiki.jasig.org/display/LIC/maven-notice-plugin) to help remedy the situation.  


##Preparing the Release

Prepare for release by running prepare goal of Maven Release Plugin, which prompts for version numbers:
{% highlight bash %}
export MAVEN_OPTS="-Xmx2048m"
mvn release:prepare
{% endhighlight %}

Alternatively, you may specify the release version number directly on the command line:
{% highlight bash %}
export MAVEN_OPTS="-Xmx2048m"
mvn -DreleaseVersion=x.y.z -DdevelopmentVersion=a.b.c release:prepare
{% endhighlight %}


##Performing the Release

{% highlight bash %}
mvn release:perform
{% endhighlight %}

Follow the process for [deploying artifacts to Maven Central](https://wiki.jasig.org/display/JCH/Deploying+Maven+Artifacts) via Sonatype OSS repository.  

- Log into [https://oss.sonatype.org](https://oss.sonatype.org).
- Find the staged repository for CAS artifacts created by the `mvn release:perform` goal.
- "Close" the repository.
- "Release" the repository.  Both c and d should be accompanied by email confirmation.

Send an announcement message to cas-announce, cas-user and cas-dev. A template follows:

{% highlight bash %}
CAS Community,

CAS x.y.z is available for testing and evaluation. We encourage adopters to grab 
this release from Maven Central, integrate into your environment, and provide feedback.

Regards,
John Smith

{% endhighlight %}

Finally, trigger the release in [JIRA](https://issues.jasig.org/secure/Dashboard.jspa). You will need to
have proper permissions in order to mark a given CAS version as "Released". 

* Mark the version released, moving remaining open issues to the next version.
* Do a query for all unresolved issues that affect the previous release and release candidates. Add the new release version as an affected version for each issue in the results.
* Batch transition all resolved issues for the version to closed.

##Post Release

<div class="alert alert-warning"><strong>GA Releases</strong><p>The following steps should only be executed for public GA releases.</p></div>

###Publishing Artifacts

- Check out source from generated branch/tag
- Build the assembly using the following command:
{% highlight bash %}
mvn -DskipTests clean package assembly:assembly && mvn -N antrun:run
{% endhighlight %}
- [Upload assembly](https://wiki.jasig.org/display/JCH/Publishing+Project+Downloads) to Jasig Download Servers. 
- Generate release note/message in JIRA
    - Log into JIRA at https://issues.jasig.org/secure/Dashboard.jspa
    - Under the "Projects" menu at the top, select "CAS Server"
    - From the TOC on the left, select "Versions"
    - From the list, select the CAS server version under release
    - Click on "Release Notes" on the top right-hand corner and copy the release notes
- Add entry to download section of the [website](http://jasig.org/cas) with release notes.
    - Go to https://www.jasig.org/user and log in
        - Note: CAS download block is available at http://www.jasig.org/admin/build/block/configure/block/21
    - Navigate to [Content Management] - [Create Content] - [Product Release] at https://www.jasig.org/node/add/product-release
    - Give the page a title of `CAS Server x.y.z Release`
    - Select `CAS Product Release` under Site Sections
    - Add download links for the CAS Server version under release
    - Copy over the content of the release notes page from JIRA into the body
    - Under the URL path settings section set a URL of: `cas/download/CAS-XYZ`
    - Under Publishing options check Published, Promoted to front page, and Sticky at top of lists
    - Click Save
    - Edit the page for the previous release and under Publishing options uncheck Promoted to front page, and Sticky at top of lists.

###Documentation

- Build the documentation site locally once. Artifacts will appear in the `_site` directory.
- Copy over the `_site` folder to the same directory
- Rename the folder to match the CAS release version (i.e `4.0.0`)
- Run a global search and replace on all generated artifacts to replace all instances of the development version number to the GA release number. For example, replace `4.0.0-RC4-SNAPSHOT` to `4.0.0`.
- Navigate to `Older-Version.md` page and include a link to the new directory that points to the new release.
- Modify the root `index.html` file of the `current` folder to point to the latest stable release such that `location.href = "../4.0.0/index.html";`
- Push the changes to the repository.

##Common Issues
If you're preparing and cutting the release, please be wary of the following possible issues:

###SSH Authentication: private key's passphrase prompt hangs
SSH authentication with git on Windows is unable to cache the private key's password regardless of any authorized SSH agent running in the background. The maven-release-plugin currently is unable to actually present the password prompt. 

To mitigate the problem, you will need to modify the release.properties file, and switch the authentication scheme from SSH to HTTPS. 


###"Directory is outside the repository" error
When the maven-release plugin attempts to add/commit/push back files to the repository from the project directory, you might receive the error that the directory is outside the repository! This has to do with the fact that the plugin and git consider path names to the project directory to be case-sensitive while Windows does not. In other words, git might consider the project repository directory to be available at "c:\project", while the current directory is pointed to "C:\project". 

To mitigate the problem, simply "CD" into the directory path of the working directory given by git, based on the console output. 

###"Dependencies cannot be found/resolved" error
Once you have prepped the release and start to execute `release:perform`, the maven-release plugin will attempt to checkout the prepped release from source first and run through the build to upload artifacts. You may at this stage encounter the error that artifact dependencies (that are based on the new tagged release) cannot be found. For example, if you have prepped the release V4-RC1 and are attempting to cut it,  you may receive the error that certain modules for V4-RC1 cannot be resolved during the build. 

To mitigate the issue, navigate to the `target\checkout` directory and run `mvn install` once to locally capture the newly tagged artifacts. You may alternatively, checkout the CAS project again, switch to the tag and perform `mvn -o install -Pnocheck`


###Java heap space "Out of memory" errors
Depending on settings, the build might run out of Java heap space. You can tweak settings through `JAVA_OPTS` and `MAVEN_OPTS` and allow more via the `-Xmx` parameter. Usually, `2g` should suffice on a 64-bit Java VM:

`export MAVEN_OPTS="-Xmx2048m"`


###The build is sluggish

The current CAS build runs a number of checks and processes as it progresses forward in the release process. Disabling these checks will help speed up the build because, the maven-release plugin attempts to run all said checks for all modules prior to processing the active module during the build. In other words, if the build is attempting to process module C, it will run the checks for modules A and B...and then it moves on to C. Similarly, if the build is at module D, it will yet again run all checks for modules, A, B and C before it starts with D. 

You can disable some of these checks in the Maven's `settings.xml` file. Here are some of the steps you can through configuration ignore by setting them to *true*:

<div class="alert alert-danger"><strong>Caution!</strong><p>You must ensure that all above checks do actually pass, separately and independently, if you do decide to disable them in the build process.</p></div>

- Skip running tests (Tests still would have to be compiled): `<skipTests>true<skipTests>`
- Checkstyle checks: `<checkstyle.skip>true</checkstyle.skip>`
- License checks: `<license.skip>true</license.skip>`
- Notice file checks: `<notice.skip>true</notice.skip>`
- Dependency version checks: `<versions.skip>true</versions.skip>`

The current Maven build contains a `nocheck` profile that encapsulates the above settings. The profile may be invoked via `-Pnocheck` parameter.







