---
title: "Reverse engineering Eclipse Oomph catalog"
---

## Reverse engineering Eclipse Oomph catalog

The source for all setups included in the Eclipse Installer are stored in the [`setups/` folder in the `org.eclipse.oomph` repository](http://git.eclipse.org/c/oomph/org.eclipse.oomph.git/tree/setups).

Example: if you want to find the `*.setup` file that contains the definition of the _"Eclipse Scout Demo App"_ project that you can select in step 2 of the Eclipse Installer (advanced mode)

<img width="400" alt="Step2 - Eclipse Installer: select projects" src="https://user-images.githubusercontent.com/1222165/47487847-86c0f080-d843-11e8-8384-acbf225757f9.png">

Back to the Eclipse Oomph repository, lets have a look at the at the [`org.eclipse.projects.setup` file](http://git.eclipse.org/c/oomph/org.eclipse.oomph.git/tree/setups/org.eclipse.projects.setup):

There is no detailed setup definition there. The file contains only _"includes"_ directive of other setup files. This way the problem of having up-to-date definitions is devided in smaller parts (this presents the advantage that each Eclipse project is able to update its setup file without having to submit a pull request to change something in the Oomph project).

For Eclipse Scout project, the _"include"_ directive is defined like this:

```
  <project href="https://git.eclipse.org/c/scout/oomph.git/plain/Scout.setup#/"/>
```

Now you can follow the link. For Eclipse Scout the file defining the projects setup is [`Scout.setup` file in the `scout/oomph.git` repository](https://git.eclipse.org/c/scout/oomph.git/tree/Scout.setup).
