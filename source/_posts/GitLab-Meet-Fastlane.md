---
title: 'GitLab, Meet Fastlane'
tags: []
date: 2016-03-26 23:58:34
---

I configured GitLab on a Linode VPS today, and managed to successfully configure its CI features to build both an iOS app and this Jekyll blog using my iMac as a worker.

## tl;dr

1.  [Get a VPS](https://www.linode.com/?r=5fc6110467ead40f187656075e31a4b2fd52e3bd)
2.  [Install GitLab on your VPS](https://about.gitlab.com/downloads/)
3.  [Install the GitLab Runner on your Mac](https://gitlab.com/gitlab-org/gitlab-ci-multi-runner/blob/master/docs/install/osx.md) (choose `script` as the executor, unless you know better)
4.  (iOS) [Configure `fastlane` to build your project](https://github.com/fastlane/fastlane/blob/master/fastlane/docs/Guide.md)
5.  Add a `.gitlab-ci.yml` file to your projects to enable CI

## Details

### Xcode test coverage

I’ve configured my minimal `fastlane` test lane as follows:

<figure class="highlight">

    <span class="n">desc</span> <span class="s2">"Runs all the tests"</span>
    <span class="n">lane</span> <span class="ss">:test</span> <span class="k">do</span>
      <span class="nb">scan</span>
      <span class="n">xcov</span>
    <span class="k">end</span>`</pre></figure>

    `scan` runs my tests, which also generates Xcode coverage reports [if your scheme is properly configured](https://developer.apple.com/library/ios/documentation/ToolsLanguages/Conceptual/Xcode_Overview/CheckingCodeCoverage.html). `xcov` then takes the coverage output and renders it in a slightly more pretty fashion for archiving purposes; this step is totally optional.

    Here’s a simplified version of my iOS project’s `.gitlab-ci.yml` file to get you started:

    <figure class="highlight"><pre>`<span class="s">stages</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">build</span>

    <span class="s">build_project</span><span class="pi">:</span>
      <span class="s">stage</span><span class="pi">:</span> <span class="s">build</span>
      <span class="s">script</span><span class="pi">:</span>
        <span class="pi">-</span> <span class="s">fastlane test</span>
      <span class="s">tags</span><span class="pi">:</span>
        <span class="pi">-</span> <span class="s">ios</span> <span class="c1"># my iMac is tagged the same way in GitLab, </span>
              <span class="c1"># which allows it to process these builds</span>
      <span class="s">artifacts</span><span class="pi">:</span>
        <span class="s">untracked</span><span class="pi">:</span> <span class="s">true</span> <span class="c1"># archive all untracked files in the </span>
        <span class="s">paths</span><span class="pi">:</span>          <span class="c1"># fastlane path as build artifacts</span>
          <span class="pi">-</span> <span class="s">fastlane/</span>
</figure>

If you want to use your dev machine as a worker instead of dedicating a machine to that purpose, I recommend creating a separate account for this purpose and logging in via fast user switching to start the worker service running. You should also run all of your fastlane/jekyll steps manually as this user at least once, to make sure your environment is sane and that you’ve clicked through any permissions dialogs, etc. that Xcode might throw in your path. Once this is done, you can switch back to your main account (via fast user switching, so that both accounts remain logged in) and jobs will happily run in the background without affecting your actual working environment.

### Supplemental Reading

*   fastlane can do a whole lot more than just running your tests. Check out the list of actions [here](https://github.com/fastlane/fastlane/blob/master/fastlane/docs/Actions.md).
*   GitLab has [a recent post on their blog](https://about.gitlab.com/2016/03/10/setting-up-gitlab-ci-for-ios-projects/) about getting their CI set up for an iOS project. It has some details about getting things working without fastlane that I haven’t covered here, and might be worth a look regardless.
*   [`fastlane` example configurations](https://github.com/fastlane/examples)
*   [GitLab CI example configurations for other platforms](http://doc.gitlab.com/ee/ci/examples/README.html)