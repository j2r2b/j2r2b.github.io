---
title: "Storing draw.io decompressed XML"
---

If you work with the online version https://www.draw.io/ (coupled with google drive for example) or with the desktop draw.io application you can download your work as `*.drawio` file.

![draw.io Desktop](/images/2019-08-01-drawio-desktop.png)

This is an XML file, but for performance reasons (in online scenarios) the content of the graph is compressed.
Assuming you have only one tab "Page-1”, your `*.drawio` file looks like this:

```xml
<mxfile modified="2019-08-01T09:23:36.597Z" host="" agent=“..” etag="M34yXdgxU2q9AoIo9Xkg" version="10.9.5" type="device">
    <diagram id="sgW9pF4t5AKx6TWdV9Rh" name="Page-1"><!-- diagram content here --></diagram>
</mxfile>
```

The content inside the `<diagram>..</diagram>` can be decompressed with following steps:

1.	Base64 decode
2.	Inflate
3.	URL decode

jgraph provides [a conversion tool](https://jgraph.github.io/drawio-tools/tools/convert.html) that does the 3 steps inside a webpage.

![Conversion tool to decompress draw.io xml](/images/2019-08-01-drawio-conversion-tool.png)

Paste your content, and hit decode.

You get something like this:

```xml
<mxGraphModel dx="1106" dy="776" grid="1" gridSize="10" guides="1" tooltips="1" connect="1" arrows="1" fold="1" page="1" pageScale="1" pageWidth="413" pageHeight="583" math="0" shadow="0">
  <root>
    <mxCell id="0" />
    <mxCell id="1" parent="0" />
    <mxCell id="IgmaPHdbTIZ3v5u-N69z-1" value="" style="rounded=0;whiteSpace=wrap;html=1;fillColor=#fff2cc;strokeColor=#d6b656;" vertex="1" parent="1">
      <mxGeometry x="20" y="10" width="80" height="40" as="geometry" />
    </mxCell>
    <mxCell id="IgmaPHdbTIZ3v5u-N69z-2" style="edgeStyle=orthogonalEdgeStyle;rounded=0;orthogonalLoop=1;jettySize=auto;html=1;exitX=1;exitY=0.25;exitDx=0;exitDy=0;entryX=0;entryY=0.75;entryDx=0;entryDy=0;" edge="1" parent="1" source="IgmaPHdbTIZ3v5u-N69z-3" target="IgmaPHdbTIZ3v5u-N69z-5">
      <mxGeometry relative="1" as="geometry" />
    </mxCell>
    <mxCell id="IgmaPHdbTIZ3v5u-N69z-3" value="" style="rounded=0;whiteSpace=wrap;html=1;fillColor=#d5e8d4;strokeColor=#82b366;" vertex="1" parent="1">
      <mxGeometry x="141" y="10" width="80" height="40" as="geometry" />
    </mxCell>
    <mxCell id="IgmaPHdbTIZ3v5u-N69z-4" value="" style="endArrow=classic;html=1;entryX=0;entryY=0.25;entryDx=0;entryDy=0;exitX=1;exitY=0.75;exitDx=0;exitDy=0;" edge="1" parent="1" source="IgmaPHdbTIZ3v5u-N69z-1" target="IgmaPHdbTIZ3v5u-N69z-3">
      <mxGeometry width="50" height="50" relative="1" as="geometry">
        <mxPoint x="21" y="118" as="sourcePoint" />
        <mxPoint x="71" y="68" as="targetPoint" />
      </mxGeometry>
    </mxCell>
    <mxCell id="IgmaPHdbTIZ3v5u-N69z-5" value="" style="rounded=0;whiteSpace=wrap;html=1;fillColor=#f8cecc;strokeColor=#b85450;" vertex="1" parent="1">
      <mxGeometry x="262" y="10" width="80" height="40" as="geometry" />
    </mxCell>
  </root>
</mxGraphModel>
```

If you would like to store your draw.io sources inside a git repository, storing the graphs as decompressed and formatted XML is way better to understand the changes between the revisions.

draw.io Desktop is able to open a `*.drawio` file containing some decompressed xml content.
But as soon as the file is modified and saved, it is stored back in the compressed format.

---

By the way under "Menu > Extras > Edit Diagram…" inside the draw.io application you can access the same decoded xml content.

![Edit Diagram to see the xml sources in draw.io](/images/2019-08-01-drawio-edit-diagram.png)

---

References:

* [Extracting the XML from mxfiles](https://about.draw.io/extracting-the-xml-from-mxfiles/)
* [Rendering XML from draw.io on stackoverflow](https://stackoverflow.com/a/35322632/91497)
