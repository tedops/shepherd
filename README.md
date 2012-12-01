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