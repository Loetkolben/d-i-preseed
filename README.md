<!--
SPDX-FileCopyrightText: 2020 Loetkolben
SPDX-License-Identifier: MIT
-->

# d-i-preseed

Preseeds for the Debian installer (`d-i`).

Upstream documentation:
- [Debian Wiki](https://wiki.debian.org/DebianInstaller/Preseed)
- [Installer Guide](https://www.debian.org/releases/stable/amd64/apbs03.en.html)
- [Sample Preseed File](https://www.debian.org/releases/stable/example-preseed.txt)

## Documentation
- Documentation for the preseed parameters is available in the form of the
  `PACKAGE_NAME/debian/PACKAGE_NAME.templates` files in the `d-i` source
  repository [here](https://salsa.debian.org/installer-team/d-i)
  - Remember to checkout the "sub repos" with `mr` (`myrepos` tool)
  - If using a "smart" search tool (IDE, ripgrep), remember that
    the "sub repos" are gitignored in the main repo, so configure your
    search tool to ignore the ignore files:
    `rg --no-ignore partman-auto/cap-ram` 


## Updating preseed files for new releases
As the documentation and the sample preseed files are part of the
[install guide](https://www.debian.org/doc/user-manuals#install)
whose source (Docbook XML) is in
[git](https://salsa.debian.org/installer-team/installation-guide)
(to be specific, in `en/appendix/preseed.xml` in the repo),
one can (hopefully) find (at least the important) changes with `git diff`:
```
git diff origin/bullseye..origin/bookworm en/appendix/preseed.xml
```
