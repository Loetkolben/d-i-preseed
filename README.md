# d-i-preseed

Preseeds for the Debian installer (d-i).

Upsteam documentation:
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
