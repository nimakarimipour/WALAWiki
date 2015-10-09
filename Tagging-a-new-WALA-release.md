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
**TODO: add instructions for pushing new release to Maven Central**

Now that we've tagged the release, we should update the version number on master to return to a SNAPSHOT build using the `change-version.py` script again, e.g.:
```
./change-version.py 1.3.8 1.3.9-SNAPSHOT
```
After running this script, commit and push the changes, and then development can continue on `master`.  Keeping these version numbers correct is important for automated builds and for pushing release jars to Maven Central.