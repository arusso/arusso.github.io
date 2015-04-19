---
title: "Inexpensive Puppet Manifest Testing"
layout: post
date: 2015-04-18 21:16:02
comments: true
tags: [puppet,ci]
share: true
---

_...because "Poor Man's Puppet Testing" just sounded lame..._

## Disclaimer

There are better, more effective and automated ways of testing your puppet
manifests. However if you are in a position where setting up a CI server is not
in the time budget, this article is for you.

This article also assumes you do not have some sort of orchestration tool at
your disposal, and uses SSH[^1] to fake it until we make it. If this is not your
situation fret not! You should be able to tailor it to use your orchestration
tool of choice with minimal effort.

## Overview

This article discusses a fairly simple idea -- we will be updating some puppet
code, committing our changes and having a simple process run noop runs on hosts
we specify. The output of these runs will be displayed for us to inspect.

## Setup

First off, we will need two bash functions. The first is a helper function that
I came up with as a way to abstract away running arbirtray commands on a host
called ``runon``. Fancy, right? Fortunately, you should be able to refactor it
to use your tool of choice fairly easily.

{% highlight bash %}
runon () {
    CLIENT=$1;
    COMMAND=$2;
    SSH_OPTS="-o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no -a -T"

    if [[ "$1" == "" || "$2" == "" ]]; then
        echo "USAGE: runon <client> \"<command>\"    ";
    else
        ssh $SSH_OPTS ${CLIENT} "$2";
    fi
}
{% endhighlight %}

Next up, we are going to setup a bash function that is constantly checking a
file we specify for hosts to connect and test our manifests against. I use
environments to test all code changes[^2], so the function needs to know about
the environment as well as the filename to poll for changes.

{% highlight bash %}
test_puppet_env () {
    if [[ -f $1 && "$2" != "" ]]; then
        while [ 1 -eq 1 ]; do
            while read line; do
                for hst in $(eval "echo $line"); do
                    echo "#### HOST: ${hst} ####";
                    runon ${hst} "sudo puppet agent -t --noop --environment ${2}";
                    echo "#######################";
                done;
                eval "sed -i '' -e '/^$line\$/d'" $1;
            done < $1;
            sleep 1;
        done
    else
       echo "USAGE: test_puppet_env <filename> <environment>"
       return
    fi
}
{% endhighlight %}

With that, we have all the functions we need. Now, let's put it to use. We will
assume I am doing some refactoring in a puppet environment 'foo', and want to
test changes that would apply to hosts specified in ~/test-hosts

{% highlight bash %}
test_puppet_env ~/test-hosts foo
{% endhighlight %}

Now open up a new window (or hopefully a pane[^3]) and run the following command:

{% highlight bash %}
echo arusso-dev-0{1..9}.example.com >> ~/test-hosts
{% endhighlight %}

Suddenly, you will see output in the original pane where we ran
```test_puppet_env```. For a couple of hosts, this works reasonably well. But
when I am testing a large number of hosts (>5) I typically do the following so
I can grep through the files later rather than read them outright:

{% highlight bash %}
test_puppet_env ~/test-hosts foo | tee session-$(date +%Y%m%dT%H%M)
{% endhighlight %}

Now, in addition to having the output sent to stdout I can have a file I can
grep/awk/sed through to find interesting bits of information I'm looking for.

So that's it. Not too bad, right? Let me know what you think below.

[^1]: Technically this is SSH in a loop. Sleep well knowing I carry the burden of this terrible deed. Still, let's keep it between us and not say anything to Luke.
[^2]: Obviously some code cannot be tested in environments; the most obvious example being types. I assume that you know this, and use something like Vagrant to develop and modify such code in a sandbox. If not, I highly suggest this approach.
[^3]: tmux (or screen) if your friend here.
