[Release process](/content/release-process)

*   [Developer](/category/level/developer "This level is generally for users interested in further developing the Aegir system itself, or extending it with contributed modules.")

This page aims to document our release process. It documents the release cycle, but also the steps required to make a release.

# 1. The release cycle

In general, each major Aegir release comprises a simultaneous release of all the modules that are part of the project. We generally go through several testing releases (alphas, betas & RCs) before doing the first stable release on a branch.

*   First an **alpha** is released to test new functionalities and to accomplish the goals decided in the [Project Roadmap](/node/35) for that major version. Example: `0.4-alpha1`, [`0.4-alpha15`](/0.4-alpha15)
*   When we have covered most of the functionalities outlined in the [roadmap](/roadmap), we push out **beta releases** until no more critical issues show up. This is generally considered a _soft feature freeze_. Example: [`0.4-beta1`](/0.4-beta1)
*   Then we go into _full feature freeze_ and release a **first release candidate** (RC). Then a stable release branch is created, and the development branch is kept opened for development for the next stable release. This is generally considered a _soft API freeze_. Release candidates are made as long as critical bugs are found. Example: [`0.4-rc1`](/0.4-rc1), [freeze announcement](/node/360)
*   Once the development branch has no known critical bugs, the **stable release** is announced. From there on only critical fixes (security, critical performance and critical bugfixes) are committed to the stable branch, and stable releases are published (without alpha/beta/RC) directly on the stable branch. The stable branch is in **full API freeze**. New features are generally committed to the development branch.

See also [the tag and branch naming convention](/content/branch-and-tagging-conventions).


# 2. Steps for a release

For the specifics of release naming conventions and the cycle, see [the branch naming convention](/node/187).


## 2.1. Make sure Jenkins is all green

Look into [Jenkins](http://ci.aegirproject.org/) to see if all tasks have been performed without errors since the last commit. If there is an error, fix it before the release.


## 2.2. Verify that drupal-org.make specifies up-to-date versions

In the hostmaster project we maintain our own drupal-org.make file. Check that e.g. the ctools version specified is not out-dated.


## 2.3. Generating the release notes

We build complete release notes for every release. Those are made up of a summary of the release, an outline of key changes, of known issues, install and upgrade instructions and a full list of bugfixes and new features.

Using [Git Release Notes for Drush](https://www.drupal.org/project/grn)

`drush rn 7.x-3.0-beta1 HEAD`

or later

`drush rn 7.x-3.0-alpha1 7.x-3.0-alpha2`

The developers then proceed to format/edit the list of fixes as well as list other significant information/changes for this release. These notes end up becoming the Release Notes for the release, which are also entered in the `debian/changelog` file by the script below.

## 2.4. Running the release.sh script

Each time we make a new release, we run a script called `release.sh` in provision.

This script should only be used by the core dev team when doing an official release. If you are not one of those people, you probably shouldn't be running this.

This script does all the 'hard' work in that it doesn't forget all the very many places to edit version numbers etc of relevant documentation and other scripts. This includes install.sh.txt and upgrade.sh.txt.

_note from just after 7.x-3.0-beta1_: It's probably a good idea to disable the Jenkins jobs named 'D_aegir-debian*'. This could prevent later troubles in the 'Publish the Debian packages' section.

Paraphrasing from the script itself:

```
The following operations will be done:
 0. prompt you for a debian/changelog entry  
 1. change the makefile to download tarball  
 2. change the upgrade.sh.txt version  
 3. display the resulting diff  
 4. commit those changes to git  
 5. lay down the tag  
 6. revert the commit  
 7. (optionally) push those changes  

The operation can be aborted before step 7. Don't forget that as  
long as changes are not pushed upstream, this can all be reverted (see  
git-reset(1) and git-revert(1) ).
```

Next, we need to add the tag in hostmaster, and push it to the d.o repos. So in short, this sums up as:

```
cd provision
sh release.sh 1.10  
cd ../hostmaster  
git tag 6.x-1.10  
git push origin 6.x-1.10
```

Notice how we just provide the Aegir release number (`1.10`) to the release script, not the Drupal branch (`6.x`), which is hardcoded in the script to remove potential confusion.

**Special, for 2.x:** While we wait for core support in drush ([issue #1991764](https://drupal.org/node/1991764)) we need to manually modify the makefiles in hostmaster, drupal-org.make and hostmaster.make to point to the tags we lay down manually in hostmaster, hosting and provision (and maybe eldir). So this ends up being:

```
cd provision  
sh release.sh 2.0  
cd ../hostmaster  
git pull  
git tag 6.x-2.0  
git push origin 6.x-2.0  
cd ../hosting  
git pull  
git tag 6.x-2.0  
git push origin 6.x-2.0
```

### 2.4.1. Optional: new Eldir release

If we do a major release (say a point zero release), we may want to make a new release of the theme ([Eldir](https://drupal.org/project/eldir)). This can also be performed if there enough new changes on the theme to warrant a new release on its own.

To do a new release of Eldir:

1.  tag and push the tag
2.  update the version to fetch in hostmaster.make
3.  [create a release node](https://drupal.org/node/add/project-release/452774) for Eldir


## 2.5. Test the manual install in Jenkins

Before making a full release, test the release in Jenkins. To do so, start a build of the launch the [S_aegir_6.x-1.x_install job](http://ci.aegirproject.org/job/S_aegir_6.x-1.x_install/) (or [S_aegir_6.x-2.x_install job](http://ci.aegirproject.org/job/S_aegir_6.x-2.x_install/) for 2.x) with the following parameters:

*   the right release as the `AEGIR_VERSION` parameter
*   the `DRUSH_VERSION` to match what is required for this release. (Currently '4.5' for Aegir 1.x)

If the build fails, delete the remote tags (using `git push origin :6.x-1.7`, for example), fix the bugs and start again.


## 2.6. Build the Debian packages

Build the package and upload to [http://debian.aegirproject.org/](http://debian.aegirproject.org/ "http://debian.aegirproject.org/"). Jenkins can build and upload a Debian package for you with [the S_aegir-debian-official job](http://ci.aegirproject.org/job/S_aegir-debian-official/).

If you need to move the tags again, you will need to clear the testing archive using the [R clear repo job](http://ci.aegirproject.org/job/R%20clear%20repo/), with the testing argument.

You can also build and upload the package yourself as explained in these [detailed instructions](http://community.aegirproject.org/node/543). We first upload the package to the `testing` distribution, and it gets migrated down into `stable` after tests.

**Special, for 2.x**: build the package manually, see [detailed instructions](http://community.aegirproject.org/node/543). It should be something like:

    ./release.sh 2.0-rc5
    git reset --hard 6.x-2.0-rc5
    git-buildpackage -kanarcat@koumbit.org --git-builder=debuild
    dput aegir build-area/aegir2-provision_2.0~rc5_amd64.changes

See the [detailed instructions](http://community.aegirproject.org/node/543) for the dput configuration.


## 2.7. Test the upgrade in Jenkins

Once both of those tasks have executed successfully, you can test the upgrade of the Debian packages by running the following Jenkins job:

[http://ci.aegirproject.org/view/Upgrades/job/U%20aegir%206.x-1.x%20deb%2...](http://ci.aegirproject.org/view/Upgrades/job/U%20aegir%206.x-1.x%20deb%20upgrade/ "http://ci.aegirproject.org/view/Upgrades/job/U%20aegir%206.x-1.x%20deb%20upgrade/")

(Note that this test is currently non-functional)


## 2.8. Creating release nodes on Drupal.org

Once the tags are pushed and release notes published, we create a release node with an excerpt of (and a link to) the release notes so that tarballs are created and issue queue versions updated.

This needs to be done in the [hostmaster](https://drupal.org/node/add/project-release/195997) and [provision](https://drupal.org/node/add/project-release/196005) (and [hosting](https://drupal.org/node/add/project-release/196008) and (maybe) [eldir](https://drupal.org/node/add/project-release/452774) for 2.x) projects on Drupal.org.

Note: this could be [automated](http://drupal.org/node/1050618) with the right stuff on Drupal.org.

**Special, for 2.x**: make sure release nodes are created in order! The projects shipped with hostmaster (hosting, eldir, etc) need to be fully released before the hostmaster release node is created!


## 2.9. Build the release in Jenkins again

At this point, it's possible that the tarballs on Drupal.org were not created properly. We want to test the real procedure, so run a your build again, but choose 'package' as the `AEGIR_FETCH_MODE`. [S_aegir_6.x-1.x_install job](http://ci.aegirproject.org/job/S_aegir_6.x-1.x_install/)

Note: The link provided may reference the wrong test, since there doesn't appear to be an AEGIR_FETCH_MODE parameter.


## 2.10. Publish the Debian packages

Finally, when the Debian packages are tested you will need to pull them into the stable release channel:

[http://ci.aegirproject.org/view/Release%20scripts/job/R%20repo%20pull/](http://ci.aegirproject.org/view/Release%20scripts/job/R%20repo%20pull/ "http://ci.aegirproject.org/view/Release%20scripts/job/R%20repo%20pull/")

**2.x note**: we pull to testing (and stable since the betas), manually:

    reprepro copy testing unstable aegir2
    reprepro copy testing unstable aegir2-hostmaster
    reprepro copy testing unstable aegir2-provision
    reprepro copy testing unstable aegir2-cluster-slave

**3.x note**: we pull to stable (since the betas), manually:

    reprepro@zeus:~$ reprepro copy stable unstable aegir3
    reprepro@zeus:~$ reprepro copy stable unstable aegir3-hostmaster
    reprepro@zeus:~$ reprepro copy stable unstable aegir3-provision
    reprepro@zeus:~$ reprepro copy stable unstable aegir3-cluster-slave

If Jenkins has managed to build .debs and upload them before you've have a chance to pull them into testing/stable, you can manually remove them like so:

    reprepro remove unstable aegir2-cluster-slave aegir2 aegir2-provision aegir2-hostmaster

This should be prevented by disabling the Jenkins jobs named 'D_aegir-debian*' before starting to push the release tags.

You can then re-upload the new .debs you've generated, using the '-f' (force) flag:

    dput -f aegir build-area/aegir2-provision_2.3_amd64.changes

After doing that, you can re-run the 'copy' commands to publish the .debs to the appropriate releases.


## 2.11. Publish the release notes widely

Once all this is done and the tarballs are generated, the release notes are published in:

*   The [handbook](/node/66) (this is where the release notes live!)
*   A link to the release notes on the [frontpage block](/)
*   An [event in the calendar](/node/add/event)
*   A [discussion post](/node/add/discussion) (don't forget to make it 'sticky' & remove stickiness from the previous announcement)
*   Update the version in the [script upgrade page](/upgrading/script)
*   The topic of the IRC channel
*   The aegir-announce mailing list ([announce@lists.aegirproject.org](mailto:announce@lists.aegirproject.org).)
*   Twitter as @aegirproject

Optionally, blog posts on [koumbit.org](http://koumbit.org), [mig5.net](http://mig5.net), and elsewhere may go into further detail about significant changes, screencasts etc.