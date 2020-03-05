# Releasing InSpec

## Promote Dependency Gems

It's important that dependency gems be promoted (released to rubygems.org) BEFORE the last PR you intend to release is merged into inspec/inspec. This is because during the inspec merge process, software artifacts are created that embed the dependencies (such as omnibus packages, Habitat packages, and Docker images). During promotion, those artifacts are simply promoted, not rebuilt. If you need a later version of one of these dependencies and have no legitimate PR to merge, a dummy PR may be needed.

### Key Dependencies

Any change to these gems must be promoted to rubygems.org prior to releasing InSpec.

#### train / train-core

Train is the connectivity library for InSpec, and is the most critical dependency of InSpec.

train-core is a stripped-down version of train, with no cloud provider plugins and no gems that require compilation during installation.

You can promote train by running, in #inspec-notify,
```
/expeditor promote inspec/train:master 3.2.23
```

Note that the version you give to Expeditor is the plain version (not the tag, which includes a `v` prefix).

#### train-aws, train-winrm, and train-habitat

These are the transports (connectivity drivers) that have been fully plugin-ized.

train-aws in particular is critical because it carries the gem dependencies for all AWS libraries required by inspec-aws; when those change, train-aws must be updated, and so must train and inspec.

## Merging InSpec PRs

At the time of writing, a merge typically takes about 30-40 minutes of test time.

Watch the Slack channel #inspec-notify for messages about the success or failure of various steps.

Connect to the Chef VPN to fetch Expeditor logs in the event of a failure.

### Check Expeditor Labels

Most PRs shouldn't have any Expeditor control labels. The patchlevel (4.18.X) will be automatically incremented by the [expeditor configuration](https://github.com/inspec/inspec/blob/44fe144732e1e0abb2594957a880c5f1821e7774/.expeditor/config.yml#L117) and the [version bump script](https://github.com/inspec/inspec/blob/master/.expeditor/update_version.sh).

 * Expeditor: Bump Minor Version - Use when a significant new feature is being released.
 * Expeditor: Skip Changelog - Should only be used for dummy PRs.
 * Expeditor: Skip Version Bump - Use for non-code-change PRs, such as website or CI changes.
 * Expeditor: Skip Omnibus - Use for non-code-change PRs, such as website or CI changes.
 * Expeditor: Skip Habitat - Use for non-code-change PRs, such as website or CI changes.

### Click Merge

This should be straightforward, assuming the big merge button is green.

If the merge button is red, take a moment and reminisce about various regrets in your life, then make your choice. Merging a build that fails tests is counterproductive; we have further tests downstream during the post-merge process.

You'll see a message in #inspec-notify from Expeditor that it "performed actions for merged_inspec/inspec#1234" (for PR 1234, for example). Among the messages will be links to the Omnibus and Habitat builds.

### Watch Omnibus Build

The Omnibus build creates operating-system-specific packages for each platform for which we release InSpec. Its [expeditor configuration](https://github.com/inspec/inspec/blob/44fe144732e1e0abb2594957a880c5f1821e7774/.expeditor/config.yml#L133) drives a [BuildKite configuration](https://github.com/inspec/inspec/blob/master/.expeditor/release.omnibus.yml) which lists exactly which platforms to build.

The Omnibus build is generally reliable, if somewhat slow.

When the omnibus build succeeds, omnitruck delivers the packages to various package repos in `unstable` channels for public consumption.

### Watch Habitat Build

The Habitat build creates Habitat .hart packages for Linux and Windows. The [Expeditor configuration](https://github.com/inspec/inspec/blob/44fe144732e1e0abb2594957a880c5f1821e7774/.expeditor/config.yml#L138) drives a [BuildKite configuration](https://github.com/inspec/inspec/blob/master/.expeditor/build.habitat.yml).

The habitat build is unreliable, usually due to network timeouts on the Windows machines. It often requires manual retries by clicking the retry button in BuildKite (obtain the link from the message posted by Expeditor in #inspec-notify). If you do not retry, later steps will fail.

When the hab build succeeds, the packages will be placed on the Hab builder in the `unstable` channel for public consumption.

### Docker Image Built and Released

We also release a Docker image, which contains an Alpine system and InSpec installed from a gem, with the EXEC of the Docker image being `inspec`. It's a simple way to ship the dependencies of inspec.

When the Docker build succeeds, it is released directly to the `stable` channel.

### Gems Built and Placed on Artifactory

The inspec, inspec-bin, inspec-core, and inspec-core-bin gems are all built and placed on the internal Chef Artifactory server.  During promotion later they will be published to rubygems.org.

The difference between the gems is as follows:

 * inspec is a library gem, with full heavyweight dependencies, not encumbered by commercial licensing
 * inspec-bin contains an `inspec` executable and is encumbered by commercial licensing
 * inspec-core is a library gem, with lightweight dependencies and no compilation required at install time, not encumbered by commercial licensing
 * inspec-core-bin contains an `inspec` executable and is encumbered by commercial licensing

### Update Pending Release Notes

As each pull request is merged, update the [Pending Release Notes](https://github.com/inspec/inspec/wiki/Pending-Release-Notes). Some guidelines:
 * Don't include minor or non-customer visible changes, such as CI changes or test harness changes
 * Do include bug fixes, features, etc - things that impact the user.
 * Your words will be edited by the Docs team, but try help them along by keeping it human oriented; this isn't the technical-oriented CHANGELOG.
 * Add the PR number as a reference for the Docs team. They will typically remove them, but they may need to get more context by looking up the originating PR.
 * It's preferable to add notes as things merged, lest they be forgotten; sifting through a pile of merged PRs in a rush is a chore!

## Promoting InSpec
