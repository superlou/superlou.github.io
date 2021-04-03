+++
title = "Open folder with Atom from Nemo"
date = 2015-12-12
[taxonomies]
tags = ["atom.io", "nemo", "linux"]
+++

To add a context menu action, add `open_with_atom.nemo_action` to `~/.local/share/nemo/actions`.

<!-- more -->

```
[Nemo Action]
Active=true
Name=Open with Atom
Comment=Open with Atom

Exec=atom %P
Icon-Name=folder
Selection=any
Quote=double
Extensions=dir
```
