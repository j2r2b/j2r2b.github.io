---
title: "Reporting source code issues as xml or json"
---

## Reporting source code issues as xml or json

A lot of tools can report issues after source code analysis. In the java world we can mention [SonarLint](https://www.sonarlint.org/), [Checkstyle](http://checkstyle.sourceforge.net/), [SpotBugs](https://spotbugs.github.io/), [PMD](https://pmd.github.io/)…

Some of these tools can produce reports in a HTML, XML or JSON format. XML or JSON is great to be integrated with other tool (dashboards, build server…)

A simple example, produced by an analyzer tool written with the Jxlint framework looks like this:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<issues>
    <issue
        name="Lines should not be longer than 80 characters"
        severity="Warning"
        message="Line 2 is too long (length: 105)"
        category="STYLE"
        summary="The line length should be less than or equal to 80."
        explanation="In a text file (ending with `.txt`) the line should not be longer than 80 characters ..."
        location="/_absolute_path_to_/some-file.txt"
        lineNumber="2"
    />
    <issue
        name="Multiple new lines at the end of the document"
        severity="Warning"
        message="Line 9 is one of the multiple empty new lines at the end of the document. This is not allowed"
        category="STYLE"
        summary="Text document should end with zero or one new line. Additional new lines should be removed."
        explanation="In a text file (ending with `.txt`) there should be no multiple new line at ..."
        location="/_absolute_path_to_/some-file.txt"
        lineNumber="9"
    />
</issues>
```

I am wondering if there is some standard format to report this kind of issues.