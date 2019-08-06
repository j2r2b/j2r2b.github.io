---
title: "Draw.io from the command line"
---

# Draw.io from the command line

[Draw.io Desktop](https://github.com/jgraph/drawio-desktop/) can be used from the command line.

On macOS, if the application is installed in the `Application` folder you can defined following alias to the `~/.bash_profile` file:

```
alias draw.io='/Applications/draw.io.app/Contents/MacOS/draw.io'
```

## Commands

With version `11.1.1`:

### Help

List all the commands:

```
$ draw.io --help

Usage: draw.io [options] [input file/folder]

Options:
  -V, --version                      output the version number
  -c, --create                       creates a new empty file if no file is passed
  -x, --export                       export the input file/folder based on the given options
  -r, --recursive                    for a folder input, recursively convert all files in sub-folders also
  -o, --output <output file/folder>  specify the output file/folder. If omitted, the input file name is used for output with the specified format as extension
  -f, --format <format>              if output file name extension is specified, this option is ignored (file type is determined from output extension) (default: "pdf")
  -q, --quality <quality>            output image quality for JPEG (default: 90)
  -t, --transparent                  set transparent background for PNG
  -e, --embed-diagram                includes a copy of the diagram (for PNG format only)
  -b, --border <border>              sets the border width around the diagram (default: 0)
  -s, --scale <scale>                scales the diagram size
  --width <width>                    fits the generated image/pdf into the specified width, preserves aspect ratio.
  --height <height>                  fits the generated image/pdf into the specified height, preserves aspect ratio.
  --crop                             crops PDF to diagram size
  -a, --all-pages                    export all pages (for PDF format only)
  -p, --page-index <pageIndex>       selects a specific page, if not specified and the format is an image, the first page is selected
  -g, --page-range <from>..<to>      selects a page range (for PDF format only)
  -h, --help                         output usage information
```

### Svg

To convert a `.drawio` file to `svg`:

```
draw.io -x -f svg -o Diagram.svg Diagram.drawio 
```

### Png

To convert a `.drawio` file to `png`:

```
draw.io -x -f png -o Diagram.png Diagram.drawio 
```

### Convert multiple files

The drawio CLI also works with folders:

Given this tree of file (multiple `.drawio` files in one folder) and an empty folder called `out/`:

```
.
├── files
│   ├── Diag.xml
│   ├── Diagram1.drawio
│   └── Diagram2.drawio
└── out
```

All the diagrams can be converted to PNG with:

```
draw.io -x -f png -o out/ files/
```

Console output:

```
files/Diag.xml -> out/Diag.xml.png
files/Diagram1.drawio -> out/Diagram1.drawio.png
files/Diagram2.drawio -> out/Diagram2.drawio.png
```

One png file is created for each diagram in the input folder.
