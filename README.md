# daverona/alpine/rdkit

[![pipeline status](https://gitlab.com/daverona/alpine/rdkit/badges/master/pipeline.svg)](https://gitlab.com/daverona/alpine/rdkit/-/commits/master)

This is a repository of Alpine APKs of [RDKit](https://www.rdkit.org/).

* GitLab repository: [https://gitlab.com/daverona/alpine/rdkit](https://gitlab.com/daverona/alpine/rdkit)
* GitHub releases: [https://github.com/daverona/alpine-rdkit/releases](https://github.com/daverona/alpine-rdkit/releases)

## Installtion

Download APK files and `daverona.rsa.pub` from the [release page](https://github.com/daverona/alpine-rdkit/releases).
And run these commands (as root or sudoer) in a terminal:

```bash
cp daverona.rsa.pub /etc/apk/keys
apk add *.apk
rm -rf /etc/apk/keys/daverona.rsa.pub
```

## References

* RDKit repository: [https://github.com/rdkit/rdkit](https://github.com/rdkit/rdkit)
* RDKit Installation: [https://www.rdkit.org/docs/Install.html](https://www.rdkit.org/docs/Install.html)
