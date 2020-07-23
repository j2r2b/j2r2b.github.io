---
title: "Representing a tree with text"
---

# Representing a tree with ASCII or Unicode characters

Representing trees with text and a monospaced font is something I need often need to do (documentation, raising issues, …).

If your tree consists of folders and files on the filesystem, there are utilities to do it (check the `tree` command).

Different sets of characters can be used to represent the tree.

## Unicode characters

```
mySite
├──css
│   └──page.css
├──images
│   └──img.svg
├──index.html
├──js
│   └──empty.js
├──other
│   └──folder
│       └──page.html
├──page1.html
└──sub
    └──page2.html
```

* A node is using: `├` ([Box Drawings Light Vertical](https://www.compart.com/en/unicode/U+251c)) combined with two `─` ([Box Drawings Light Horizontal](https://www.compart.com/en/unicode/U+2500))
* The last node is using: `└` ([Box Drawings Light Up and Right](https://www.compart.com/en/unicode/U+2514)) combined with two `─` ([Box Drawings Light Horizontal](https://www.compart.com/en/unicode/U+2500))
* To indent lines, if more nodes are coming: `│` ([Box Drawings Light Vertical](https://www.compart.com/en/unicode/U+2502))

## ASCII characters

```
mySite
+---css
|   \---page.css
+---images
|   \---img.svg
+---index.html
+---js
|   \---empty.js
+---other
|   \---folder
|       \---page.html
+---page1.html
\---sub
    \---page2.html
```

* A node is using: `+---`
* The last node is using: `\---`
* To indent lines, if more nodes are coming: `|`


## See also

* [Generate ASCII folder structures on Windows with Tree](https://cmatskas.com/generate-ascii-folder-structures-for-windows-with-tree/)
* If you are looking for a Java implementation, the code in this [StackOverflow answer](https://stackoverflow.com/a/33438475) helped me to start.