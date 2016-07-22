Here are the steps for tagging a new WALA release.

During development, WALA master uses a SNAPSHOT release version corresponding to the next release (e.g., `1.3.8-SNAPSHOT`).  When all other changes for the release are committed, the first step for tagging a release is to update this version number to a non-SNAPSHOT version, using the `change-version.py` script in the top-level directory.  Run the script as follows (substituting the current SNAPSHOT version number):
```
./change-version.py 1.3.8-SNAPSHOT 1.3.8
```
This will update various metadata files to have the new version number.  Once this is done, commit the changes, push to github, and make sure the tests pass on Travis CI.  Assuming the tests pass, this version should be tagged as the next release, using `git tag`, e.g.:
```
git tag R_1.3.8
git push origin R_1.3.8
```
Now that we've tagged the release, we should update the version number on master to return to a SNAPSHOT build using the `change-version.py` script again, e.g.:
```
./change-version.py 1.3.8 1.3.9-SNAPSHOT
```
After running this script, commit and push the changes, and then development can continue on `master`.  Keeping these version numbers correct is important for automated builds and for pushing release jars to Maven Central.

### Maven Central

General documentation is here:

http://central.sonatype.org/pages/ossrh-guide.html

GPG key setup documentation is here:

http://central.sonatype.org/pages/working-with-pgp-signatures.html

Info on Maven is here:

http://central.sonatype.org/pages/apache-maven.html

You need to set up `~/.m2/settings.xml` to look like this:

```xml
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0
                      http://maven.apache.org/xsd/settings-1.0.0.xsd">
  <localRepository/>
  <interactiveMode/>
  <usePluginRegistry/>
  <offline/>
  <pluginGroups/>
  <servers>
    <server>
      <id>ossrh</id>
      <username>msridhar</username>
      <password>*************</password>
    </server>    
  </servers>
  <mirrors/>
  <proxies/>
  <profiles>
    <profile>
      <id>gpg-profile</id>
      <properties>
        <gpg.executable>/usr/local/bin/gpg2</gpg.executable>
        <gpg.useagent>true</gpg.useagent>        
      </properties>
    </profile>    
  </profiles>
  <activeProfiles>
    <activeProfile>gpg-profile</activeProfile>
  </activeProfiles>
</settings>
```

This uses gpg2 with the gpg agent, to avoid re-typing certificate password repeatedly.

**NOTE**: To build release jars for deployment to Maven Central, JDK 7 should be used.  JDK 8 works, but that will force anyone using the jars to also have JDK 8.

We now have a script `build-maven-jars.py` that builds the jars in the right order.  For deployment to the local Maven repository on the machine, do (in the root WALA directory):
```
./build-maven-jars.py "install -Dgpg.skip"
```
If you're a WALA developer and want to do a trial run of building release jars (including GPG signing), you can do:
```
./build-maven-jars.py install
```
Finally, when you're ready to deploy the jars (to Maven Central or to the [Sonatype staging repos](http://oss.sonatype.org/content/repositories/snapshots/) for SNAPSHOT builds), run:
```
./build-maven-jars.py deploy
```
