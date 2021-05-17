# Release Process

### Overview

This article describes the sequence of steps and methodology of preparing annd executing the release process.  The management of individual feature branches is beyond the scope of this topic.  The release process begins once all features for a given version of the sofware has been wrapped up and the codebase is frozen for new features until after the release is complete.

### Pre-release procedures

Once the features for a given relase have been merged into master, a release candidate branch (rc-{V.v}) is created and all feature PRs are frozen until after the release is complete.  Once the RC branch has been created, a draft release is created on github referencing the rc branch, and indicating the tag for the next release version (v1.1.0 in the case of releasing a new release after 1.0).  

The branch structure may look like the following:

````
hotfix ->          O - (1.0.1) - O - O - (1.0.2)
                  /
master ->  O - (1.0) - O - O   
                            \  
      release candidate:     O 
````
The hotfix branch is there to illustrate that the hotfix branch is managed independently of the release candidate.

### Making the Relaase Candidate branch.

Once the RC branch is created, final testing can commence, and any issues found should be applied in the RC branch.  These fixes will eventually merge back into master after the release.  Once thesting has been completed, a final commit is made to set the final versioning label to the sourcecode (ie: remove -SNAPSHOT from versions).

The branch structure may look like this:

````
master-1.0 ->          O - (1.0.1) - O - O - (1.0.2)
                  /
master ->  O - (1.0) - O - O 
                            \   
      release candidate:     O - O {remove -snapshot / update version label}
````

### Publishing the release

With the final commit for release added to the RC branch completed, the software is ready for release.  Finalize the contents of the release notes in Github, and publish the release. This will apply the tag to the head of the RC branch:

````
master-1.0 ->          O - (1.0.1) - O - O - (1.0.2)
                  /
master ->  O - (1.0) - O - O        
                            \       
      release candidate:     O - (1.1) 
````

### Post-release steps

With the release formally published, the RC branch can be merged into master, and the versions can be updated to reflect the next development cycle:

````
master-1.0 ->          O - (1.0.1) - O - O - (1.0.2)
                  /
master ->  O - (1.0) - O - O         O - O {1.2.0-SNAPSHOT}
                            \       /
      release candidate:     O - (1.1)
````

Once the new version is set in `master` a new hotfix branch is created to support the hotfixes for that minor version release:

````
master-1.0 ->          O - (1.0.1) - O - O - (1.0.2)
                  /
master ->  O - (1.0) - O - O         O - O {1.2.0-SNAPSHOT}
                            \       /
      release candidate:     O - (1.1)
                                    \
master-1.1 ->                        O
````

Note: if the prior hotfix is no longer necessary (ie: we no longer post hotfixes to 1.0, and new hotfixes will be released under 1.1.x), the previous master-{version} can be deleted.


### Updating Public Websites with Latest Release

atlas-demo.ohdsi.org is updated when the `released` branch is updated to a new commit.  The following commands are executed in git bash to move the `released` pointer and push the new pointer to github:

```
[1] : git update-ref refs/heads/released {release tag}
[2] : git push origin +released
```

[1] Will update the released branch pointer to the commit of the release tag, and [2] will force push the released branch, effectively re-assigning the released branch to the release commit.  A Jenkins job (managed by OHDSI) will perform a build and deploy of the WebAPI/Atlas application to the target web hosting platform.

