---
title: "My contribution to Bnd 5.3.0"
---

# My contribution to Bnd 5.3.0

While I was trying out the [enRoute Quick Start tutorial](https://enroute.osgi.org/tutorial/020-tutorial_qs.html) with the newest version of Bnd, I was not understanding the error message I got:

```
[INFO] --- bnd-export-maven-plugin:5.2.0:export (default) @ app ---
[ERROR] Error   : Fail on changes set to true (--xchange,-x) and there are changes
[ERROR] Error   :    Existing runbundles   [fr.jmini.bnd.impl;version='[1.0.0,1.0.1)', org.apache.felix.scr;version='[2.1.10,2.1.11)']
[ERROR] Error   :    Calculated runbundles [fr.jmini.bnd.impl;version='[1.0.0,1.0.1)', org.apache.felix.scr;version='[2.1.10,2.1.11)']
```

The error is raised because the `app.bndrun` file is incomplete: it does not contain the required `runbundles` entry.
If you read the guide carefully, everything is explained in the "[Resolving the Application](https://enroute.osgi.org/tutorial/020-tutorial_qs.html#resolving-the-application)" section.
With following maven command:

```
mvn -pl app -am  bnd-indexer:index \
    bnd-indexer:index@test-index \
    bnd-resolver:resolve package
```

The file `app.bndrun` is updated and everything works fine.

But this did not explain the error message I got.

It turned out I was not the only one complaining about this, it has been already raised by Neil Bartlett as [bndtools/bnd#4082](https://github.com/bndtools/bnd/issues/4082). Finding somebody describing the problem you face in an issue, a blog post, or a StackOverflow question is always a good sign.

Even if I am impressed by widely used projects like Bnd, I thought that I could have a look at the code.
One checkout later, I was able to find the corresponding code location by just searching for the error message I saw in the maven log `"Fail on changes set to true (--xchange,-x) and there are changes"`.
I am glad I tried because the code was easy to understand and I could figure out what was wrong.
I could also adjust one of the existing unit test (make the assertion stronger) to have one test failing before my change and passing after my change.

I tried to submit a change in form of a PR and I was really happy to see it got merged in less than 3 hours. This is open-source the way I like it!

And now we see everything works as expected with version 5.3.0 of Bnd:

```
[INFO] --- bnd-export-maven-plugin:5.3.0:export (default) @ app ---
[ERROR] Error   : Fail on changes set to true (--xchange,-x) and there are changes
[ERROR] Error   :    Existing runbundles   null
[ERROR] Error   :    Calculated runbundles [fr.jmini.bnd.impl;version='[1.0.0,1.0.1)', org.apache.felix.scr;version='[2.1.10,2.1.11)']
``` 

I have moved my test code to the folder "[enroutexample/](https://github.com/jmini/bnd-experiments/tree/master/enroutexample)" in a public repository on GitHub.
