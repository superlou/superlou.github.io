+++
title = "checkinstall vs. make install"
date = 2012-12-09
[taxonomies]
tags = ["linux"]
+++

Install checkinstall via:

```sh
sudo apt-get install checkinstall
```

<!-- more -->

Then, wherever you would do `make install`, run `checkinstall` instead. This tracks where the installation places files to allow easy removal via `dpkg -r my_built_package`.

More details available from the [Ubuntu documentation](https://help.ubuntu.com/community/CheckInstall).
