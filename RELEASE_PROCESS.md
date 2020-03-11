# Releasing InSpec

This document describes the release process for InSpec as used internally at Chef Software.

## Promote Dependency Gems

It's important that dependency gems be promoted (released to rubygems.org) BEFORE the last PR you intend to release is merged into inspec/inspec. This is because during the inspec merge process, software artifacts (such as omnibus packages, Habitat packages, and Docker images) are created that embed the dependencies. During promotion, these artifacts are simply promoted, not rebuilt. If you need a later version of one of these dependencies and have no legitimate PR to merge, a dummy PR may be needed.

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

You can promote each of these gems by running, in #inspec-notify,
```
/expeditor promote inspec/GEMNAME:master VERSION
```

## Merging InSpec PRs

At the time of writing, a merge typically takes about 30-40 minutes of test time.

Watch the Slack channel #inspec-notify for messages about the success or failure of various steps.

Connect to the Chef VPN to fetch Expeditor logs in the event of a failure.

### Check Expeditor Labels

Most, though not all, PRs shouldn't have any Expeditor control labels. The patchlevel (4.18.X) will be automatically incremented by the [expeditor configuration](https://github.com/inspec/inspec/blob/44fe144732e1e0abb2594957a880c5f1821e7774/.expeditor/config.yml#L117) and the [version bump script](https://github.com/inspec/inspec/blob/master/.expeditor/update_version.sh).

Here are the Expeditor control labels, and the circumstances under which they should be used:

 * Expeditor: Bump Minor Version - Use when a significant new feature is being released.
 * Expeditor: Skip Changelog - Should only be used for dummy PRs.
 * Expeditor: Skip Version Bump - Use for non-code-change PRs, such as website or CI changes.
 * Expeditor: Skip Omnibus - Use for non-code-change PRs, such as website or CI changes.
 * Expeditor: Skip Habitat - Use for non-code-change PRs, such as website or CI changes.

### Click Merge

This should be straightforward, assuming the big merge button is green.

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

We also release a Docker image (see[expeditor config](https://github.com/inspec/inspec/blob/44fe144732e1e0abb2594957a880c5f1821e7774/.expeditor/config.yml#L150)), which contains an Alpine system and InSpec installed from a gem, with the ENTRYPOINT of the Docker image being `inspec` (see [Dockerfile](https://github.com/inspec/inspec/blob/master/Dockerfile)). It's a simple way to ship the dependencies of inspec.

When the Docker build succeeds, it is labeled `current`.  Currently, there is nothing that promotes the Docker image to the `stable` label ([#4952](https://github.com/inspec/inspec/issues/4952)).

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

Promotion should only be attempted if you have a set of green artifacts in #inspec-notify.

### Notify Docs Team

Let the Docs team know that a release is planned and that editing of the release notes is needed.  They will polish up the words and remove any unused sections.

Wait until they let you know it is ready.

### Run promote command

The promote command is:

```
/expeditor promote inspec/inspec:master 4.18.100
```

Note that the version you give to Expeditor is the plain version (not the tag, which includes a `v` prefix).

A lot happens when you promote:

```
inspec/inspec:master 4.18.100 was promoted to stable by cwolfe
3:49
inspec/inspec:master performed the following actions for inspec/inspec:master (4.18.100) (logs)
• Promoted version 4.18.100 of inspec to stable.
3:49
inspec/inspec:master performed the following actions for inspec/inspec:master (4.18.100) (logs)
• Promoted version 4.18.100 of inspec to stable.
3:50
:metal: :ice_cream: inspec 4.18.100 has been promoted to the stable channel!
The artifacts can be downloaded from:
  • stable channel of https://packages.chef.io
  • stable channel of https://omnitruck.chef.io
  • https://downloads.chef.io/inspec/stable/4.18.100
3:50
inspec/inspec:master performed the following actions for inspec 4.18.100 (stable) (logs)
• Executed .expeditor/update_dockerfile.sh
• Updated CHANGELOG.md to reflect the promotion of 4.18.100
• Published inspec-4.18.100, inspec-core-4.18.100, inspec-bin-4.18.100, and inspec-core-bin-4.18.100 to RubyGems.org
• Created GitHub release Chef InSpec v4.18.100
• Tagged chef/inspec:4.18.100 with 4, 4.18, and latest
• Promoted chef/inspec/4.18.100/20200303200836 for x86_64-windows to the stable channel on the Habitat Depot
• Promoted chef/inspec/4.18.100/20200303200418 for x86_64-linux to the stable channel on the Habitat Depot
• Promoted chef/inspec/4.18.100/20200303200623 for x86_64-linux-kernel2 to the stable channel on the Habitat Depot
• Executed .expeditor/publish-release-notes.sh
• Executed .expeditor/purge-cdn.sh
• Executed .expeditor/announce-release.sh
• Notified Slack channels of the artifact_published event
```

Among other things, this automatically generates [release notes](https://github.com/inspec/inspec/blob/master/.expeditor/publish-release-notes.sh) and [publishes them](https://github.com/inspec/inspec/blob/master/.expeditor/announce-release.sh) to Discourse.

