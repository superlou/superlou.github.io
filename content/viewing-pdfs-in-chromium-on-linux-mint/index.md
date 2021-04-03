+++
title = "Viewing PDFs in Chromium on Linux Mint"
date = 2013-03-17
[taxonomies]
tags = ["linux"]
+++

Since the PDF library is not free software, it isn’t included with the distribution of Chromium. To get PDF support in Chromium:

<!-- more -->

Download the appropriate DEB package for your system from the google chrome download page.  Using the archive manager, extract libpdf.so from /opt/google/chrome/.  I extracted the file to my desktop.  Copy the library into your chromium installation libs:

```sh
sudo cp ~/Desktop/libpdf.so /usr/lib/chromium-browser
```

Restart Chromium if you have it open, and verify that the Chrome PDF Viewer is enabled via about:plugins in the URL bar.

Source: http://askubuntu.com/questions/12584/why-doesnt-chromium-have-chrome-pdf-viewer-plugin
