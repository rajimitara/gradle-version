# gradle-version

Version is always the same. (meaning there is no versioning)
Artifacts are not published to Artifactory.
Deployments are done from branch using the source code.

Versioning:
----------------------

We should integrate release plugins to our pipelines see https://github.com/researchgate/gradle-release and https://maven.apache.org/maven-release/maven-release-plugin/
What we will have to do is basically add release plugin to repositories and define necessary settings like Artifactory URL etc. Plugins should take care of the operations like setting new version, tagging the repo and publishing to Artifactory etc.
What we need to keep in my is that we would need to tweak settings as Snapshot and Release. Snapshot `mod` should simply publish current artifact to Artifactory. However, `Release` mod should remove Snapshot from current version, tag repository and publish artifact to Artifactory and set next Snapshot version. (Plugins should handle this flow out of the box already)

Development & Build & Deployment using Jenkins:
----------------------------------------------
Jenkins job to deploy branch without merging master: 
This will be used to test our changes without merging to master. Anyone who wished to test changes can build and deploy to TEST environment anytime he/she likes. No artifact will be published to Artifactory.

Jenkins job to publish latest Snapshot: 
This will publish latest Snapshot of the current Master to Artifactory which we can deploy to TEST. The idea here is to keep Master green all the time and publish a Snapshot of the latest Master that we can use before releasing the next version.

Jenkins job to release next version: 
This will release the next version for the component. Job should simply trigger the release process provided by the build tool. Job may accept additional input parameters to set next version. (i.e. releaseMinor or releaseMajor etc.)

Jenkins Job to deploy to TEST: 
This will be used to deploy new version to TEST. Job should have input parameter to specify version (release or snapshot) which we wish to deploy. Job should pick up the artifact from Artifactory and deploy it on TEST instances.

Jenkins Job to deploy to PROD: (Job need some tweaks like adding version parameters etc)
This will be used to deploy new version to PROD. No source code will involve in this job. Job should have input parameter to specify version (only release version are allowed here) which we wish to deploy. Job should pick up the artifact from Artifactory and deploy it on PROD instances.

Example work flow:

Developer develops the new shinny feature and pushes to a branch.
Uses Jenkins job to deploy it on TEST environment to check if it works as expected.
After verifying that everything works as expected, developer merges changes to master. A Jenkins jobs starts automatically to build master branch and publishes latest snapshot.
After deciding that changes are good to go, developer releases the new version using the release Jenkins job. At this point, artifact with the new version is published to at Artifactory
The new version can be deployed to PROD using the dedicated Jenkins job.