# Releasing InSpec

## Promote Dependency Gems

It's important that dependency gems be promoted (released to rubygems.org) BEFORE the last PR you intend to release is merged into inspec/inspec. This is because during the inspec merge process, software artifacts are created that embed the dependencies (such as omnibus packages, Habitat packages, and Docker images). During promotion, those artifacts are simply promoted, not rebuilt. If you need a later version of one of these dependencies and have no legitimate PR to merge, a dummy PR may be needed.

### Key Dependencies

Any change to these gems must be promoted to rubygems.org prior to releasing InSpec.

#### train / train-core

Train is the connectivity library for InSpec, and is the most critical dependency of InSpec.

train-core is a stripped-down version of train, with no cloud provider plugins and no gems that require compilation during installation.

#### train-aws, train-winrm, and train-habitat

These are the transports (connectivity drivers) that have been fully plugin-ized.

train-aws in particular is critical because it carries the gem dependencies for all AWS libraries required by inspec-aws; when those change, train-aws must be updated, and so must train and inspec.

## Merging InSpec PRs

### Click Merge

### Watch Omnibus Build

### Watch Habitat Build

### Update Pending Release Notes

As each pull request is merged, update the [Pending Release Notes](https://github.com/inspec/inspec/wiki/Pending-Release-Notes). Some guidelines:
 * Don't include minor or non-customer visible changes, such as CI changes or test harness changes
 * Do include bug fixes, features, etc - things that impact the user.
 * Your words will be edited by the Docs team, but try help them along by keeping it human oriented; this isn't the technical-oriented CHANGELOG.
 * Add the PR number as a reference for the Docs team. They will typically remove them, but they may need to get more context by looking up the originating PR.
 * It's preferable to add notes as things merged, lest they be forgotten; siftingg through a pile of merged PRs in a rush is a chore!

## Promoting InSpec
