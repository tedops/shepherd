shepherd
========

Deployment Framework for Java, PHP, Grails, and more.

Borne from the desire and necessity for consistent, reliable deployments that DON'T require
endless incarnations of scripts. Shepherd aims to provide the best concepts from convention
over configuration, while remaining flexible enough to allow configuration where needed.

Shepherd aims to eliminate the phrases:
"Did the deployment break?"
"I think the deploy broke production."
"Something went wrong during the deploy."
etc.

The syntax in Shepherd is designed to be familiar and intuitive, taking cues from RPM,
Bash, Python, Java, Maven, HTTP, and MIME.

## usage

To deploy with Shepherd, create a 'plan' file (.pln) that defines:
- Type of project (Java, Grails, etc)
- Project version dependencies, if any
- Type of artifact (JAR, WAR, TGZ (PHP))
- Servlet container (Tomcat, Glassfish)
- Web server (Apache, Jetty)
- System dependencies, including versions (mysql, Python, etc)
- Artifact location and target path

You can also insert custom actions that allow flexibility using hooks.

Designed to be run in tandem with a continuous integration (CI) system, Shepherd
accepts command line parameters, so artifact names, paths, or anything else can
be passed from a build plan.

## example

Here is a sample plan file for a fictitious Tomcat web app called 'Platform' (platform.pln):

	# PLATFORM PLAN
	project-type: java/tomcat
	artifact-type: war
	artifact-location: http://my-nexus-instance.com/repo/platform/${args.1}
	artifact-target: /var/artifacts/platform
	servlet-location: /var/tomcat7
	destination-servers: qaz07web qaz07web
	log-level: debug

	# HOOKS
	%Postinstall
	ln -nfs /var/artifacts/platform/${args.1} /var/tomcat/webapps/${args.1}

This plan will deploy the artifact (defined on the first command line parameter)
to artifact.target, run the pre-install section, then restart tomcat. Note that this
deployment will not take the *version* of Tomcat into consideration, however it will
ensure that Tomcat is on the system before it attempts deployment.

To run Shepherd:

	$ shep [plan file] [optional parameters]
	Example:
	$ shep platform plat-1012_15a.war

## hooks

Hooks allow custom actions to be inserted within Shepherd's native deploy process. Hooks
are defined by a percent sign (%) at the beginning of a line. There are 4 points at
which custom actions can be inserted into the deploy process:
- `%Prep`: this will be run before Shepherd begins running anything, similar to RPM
- `%Preinstall`: this will be run before deploying a payload to the target server/location
- `%Postinstall`: runs after the artifacts have been copied, but before service restarts
- `%Finally`: runs after deployment is complete
Hooks are what give Shepherd the edge over other deployment frameworks. They allow the
flexibility of configuration where needed, while leaving the heavy lifting to Shepherd.

## plan file

The plan file contains all of the information specific to your artifact that's necessary
for a successful deployment. If Shepherd is a short-order cook at a restaurant, the plan
file is the customer's order. Here is a list of the parameters that can be used in a plan:
<table>
<tr>
<th>project-type</th><td>determines what deployment template Shepherd uses (e.g. java/tomcat)</td>
</tr>
<tr>
<th>artifact-type</th><td>packaging type of artifact (e.g. war, jar, tgz)</td>
</tr>
<tr>
<th>artifact-location</th><td>URI of file location (e.g. http://myserver.com/file.war)</td>
</tr>
<tr>
<th>artifact-target</th><td>path to artifact location (e.g. /var/tomcat/webapps)</td>
</tr>
<tr>
<th>servlet-location</th><td>path to servlet container (e.g. /var/tomcat)</td>
</tr>
<tr>
<th>destination-servers</th><td>space-separated list of servers where deployment should go</td>
</tr>
<tr>
<th>log-level</th><td>sets the logging level output</td>
</tr>
</table>
Lines that begin with a hash symbol (#) will be ignored and so can be used to make comments.

## requirements

- Java 1.6 or greater
- MySQL version 5.0.37 or greater


## q&a

##### *How does Shepherd know the difference between a Grails war and a Java war?*
Based on what's defined in the plan, Shepherd will initiate a deployment template accordingly.
Because these things are crucial to a successful deployment, most fields in the plan file
are required, not optional.

##### *What kind of logging solution does Shepherd use?*
Shepherd makes use of slf4j with log4j, so full, robust logging is offered for every line
during the deploy process, controllable through the log-level line in the project's plan file.

##### *What happens when there's an error?*
Shepherd maintains a transaction log for every non-custom action performed during the deploy
process. When there is an error, Shepherd will provide a detailed message about the error and
initiate rollback when applicable. The auto-rollback feature can be disabled within the
project's plan file (auto-rollback: false).