This page documents the WALA release process and availability of WALA snapshot builds.  It is mostly relevant to those who are hacking on WALA itself.

## Snapshots

### On Sonatype

WALA's Travis CI configuration is set up to upload a new snapshot build to Sonatype on each successful commit to master.  You can see the snapshot builds [here](https://oss.sonatype.org/content/repositories/snapshots/com/ibm/wala/).  As is usual, the version naming scheme is `X.X.X-SNAPSHOT`.

### Installing local snapshots

If you have local changes to WALA, you can install them as a snapshot build into your local Maven repository with the following command:
```
./gradlew install
```

You can then import this snapshot build from other repositories.  For Gradle, you can use the local Maven repository by writing:
```
repositories {
    mavenLocal()
}
```

You may need to put `mavenLocal` before other repositories while you are testing the local snapshot builds, as snapshots are also present on Maven Central.

## Releases

Here are the steps for pushing a new WALA release to Maven Central.

### Initial Setup

General documentation on using Maven Central is here:

http://central.sonatype.org/pages/ossrh-guide.html

You'll need to have Sonatype credentials and to have permission to push releases for the WALA namespace.

GPG key setup documentation is here:

http://central.sonatype.org/pages/working-with-pgp-signatures.html

WALA is configured to make use of the `gpg2` command-line tool and `gpg-agent`.  If on Mac, install `pinentry-mac` to get a GUI prompt for your secret key passphrase (otherwise signing archives won't work).  [These instructions](http://www.harpojaeger.com/2017/09/20/enigmail-gnupg-pinentry-on-mac-os-x-using-homebrew) are helpful.

Set up your `~/.gradle/gradle.properties` file with the following properties:
```
SONATYPE_NEXUS_USERNAME=xxxxxxxxxx
SONATYPE_NEXUS_PASSWORD=xxxxxxxxxxxxxx
signing.gnupg.keyName=xxxxxxxxxxx
```
The `signing.gnupg.keyName` property should be the ID of the public key you will use to sign the artifacts; run `gpg2 --list-keys` to find the ID of your public key.

### Tagging and uploading a release

**Important**: Before starting the release process, be sure that the latest WALA master is green on Travis CI.  Otherwise, you may ship a broken release.  Also, make sure others are not pushing to master during the process.  All the steps below should occur with the latest `master` branch checked out.

During development, WALA master uses a SNAPSHOT release version corresponding to the next release (e.g., `1.3.8-SNAPSHOT`).  When all other changes for the release are committed, the first step for tagging a release is to update this version number to a non-SNAPSHOT version, using the `change-version.py` script in the top-level directory.  Run the script as follows (substituting the current SNAPSHOT version number):
```
./change-version.py 1.3.8-SNAPSHOT 1.3.8
```
This will update various metadata files to have the new version number.  Once this is done, commit the changes:
```
git commit -am "Prepare for release X.Y.Z."
```

Then, tag the release:
```
git tag -a vX.Y.Z -m "Version X.Y.Z"
```

Upload the release files to the staging area for Maven Central:
```
./gradlew -Dorg.gradle.parallel=false clean uploadArchives
```
This step may take several minutes.  Also, it will sign the archives, so if you don't have gpg set up correctly, it will fail.  The `-Dorg.gradle.parallel=false` is important, to prevent parallel uploads of artifacts, which doesn't work.

Then, update the version to the next snapshot, e.g.:
```
./change-version.py 1.3.8 1.3.9-SNAPSHOT
```

Commit the change:
```
git commit -am "Prepare next development version."
```

Push to master:
```
git push && git push --tags
```

At this point, it is ok for others to push to master.

### Completing the release on Sonatype

Log in to [Sonatype Nexus](https://oss.sonatype.org/) with your Sonatype credentials.  Then, click "Staging Repositories" on the left.  Scrolling down the list, you should eventually see something like "comibmwala_xxxx", where xxxx is a number.  Select that repo, and click "Close" at the top of the list.  After this step completes successfully, click "Release" at the top of the repository.  After that completes, the staging repo should be dropped and no longer appear on the list.  After some further delay (usually minutes, possibly hours), the artifacts will become available on Maven Central.
