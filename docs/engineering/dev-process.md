---
id: dev-process
title: Development Process
sidebar_label: Development Process
---

## Releasing Code
0. A release owner is delegated to follow the process below and work with the
   team to tie up any loose ends.  As of this writing, it's a volunteer chosen in our weekly team meeting.
0. The pre-flight checklist:
    0. Ensure there are no critical patches for this tag that haven't landed
       yet
    0. Fill out a new section in the [deployment doc][deployment-doc] for the Train you are tagging
    0. Ensure you've got appropriate qa signoffs
    0. Ensure you don't have any modified files or code laying around before
       you start the tag
0. Run [release.sh][release.sh] from the root of the repository.  Make sure
   there are no errors in the output.
0. Do some manual checks to make sure the generated tags are sane:
    0. Do the changelogs match expectations from `git log`?
    0. Have all the version strings been updated?
    0. Does the diff from `origin/master` (or `origin/train-xxx` if it’s a point release) look correct?
    0. Does the diff between the public and private tags look correct?
0. The release script will print some commands to run to push the public and private train branches to the remotes.  **It's best to copy and paste these so you don't mix them up.**
0. The release script will also print some URLs which you can use to open PRs to merge the train branches back to their respective master branches
0. Finally, the release script will also print out a bug template.  Copy that template and open a deployment bug in bugzilla under `Cloud Services :: Operations: Deployment Requests` ([example][example-deployment-bug]). Remember to include:
    0. Notes from the deploy doc, particularly any server side changes that
       need to happen as part of this deployment.
    0. Links to the needs:qa labels on GitHub.
    0. Links to the release tags on GitHub.
    0. Links to pertinent changelogs.
    0. Links to Circle builds that push to DockerHub.

#### Operations staff will take it from there…
0. Ensure that any configuration changes noted in the deployment bug land in [cloudops-deployment][cloudops-deployment].
0. Run any outstanding database migrations.  These are applied automatically for `dev` and `stage` but are reviewed manually for production since we may need to take care with the changes to avoid slow queries.  The migrations should be included in the [deployment doc][deployment-doc].
0. Build fxa-auth, fxa-content, fxa-oauth, fxa-profile, fxa-verifier in stage on Jenkins using git commit from cloudops-deployment PR and docker images referenced in deploy bug
0. Ensure that TeamCity tests are passing
0. Request QA on stage
    0. If major issues are found, a new patch is made and we’re back to step 3, running `release.sh patch`
        0. This command assumes that the relevant commit has been merged into the appropriate train branch. It bumps the minor rev on the last tag it finds in the tree from HEAD, so also requires that you checkout the appropriate train branch locally.
0. Deploy fxa-* to production
0. Initial deployment bug is closed

## FAQ

#### How are point releases (mid-sprint releases) handled?
Sometimes we need to push code out of our normal cycle.  These are point releases (eg. If Train 175 was our last release, this would be Train 175.1).  These follow the same process as above except instead of running `release.sh` you'll run `release.sh patch`.  Often the regular train's release bug can be re-used as well.  If you're doing an off-cycle release you should be communicating with the Engineering and Operations teams since it's, by definition, out of the normal flow.

[cloudops-deployment]: https://github.com/mozilla-services/cloudops-deployment/tree/master/projects/fxa
[deployment-doc]: https://docs.google.com/document/d/1lc5T1ZvQZlhXY6j1l_VMeQT9rs1mN7yYIcHbRPR2IbQ/edit
[example-deployment-bug]: https://bugzilla.mozilla.org/show_bug.cgi?id=1575233
[release.sh]: https://github.com/mozilla/fxa/blob/master/release.sh
