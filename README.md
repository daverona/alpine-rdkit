# daverona/alpine/rdkit

[![pipeline status](https://gitlab.com/daverona/alpine/rdkit/badges/master/pipeline.svg)](https://gitlab.com/daverona/alpine/rdkit/-/commits/master)

This is a repository of Alpine APK packages of [RDKit](https://www.rdkit.org/).

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

## Docker Installation

```dockerfile
ARG ALPINE_VERSION=3.12
FROM alpine:${ALPINE_VERSION}

ARG RDKIT_VERSION=Release_2020_03_5
ARG ALPINE_VERSION
RUN _version=${RDKIT_VERSION#Release_} && _version=${_version//_/.} \
  && _release=r0 \
  && wget -qO /etc/apk/keys/daverona.rsa.pub https://github.com/daverona/alpine-rdkit/releases/download/${_version}-${_release}-alpine${ALPINE_VERSION}/daverona.rsa.pub \
  && for apk in rdkit py3-rdkit; do \
       wget -q https://github.com/daverona/alpine-rdkit/releases/download/${_version}-${_release}-alpine${ALPINE_VERSION}/${apk}-${_version}-${_release}.apk; \
     done \
  && apk add --no-cache *.apk \
  && rm -rf *.apk /etc/apk/keys/daverona.rsa.pub

CMD ["python3"]
```

## References

* RDKit repository: [https://github.com/rdkit/rdkit](https://github.com/rdkit/rdkit)
* RDKit Installation: [https://www.rdkit.org/docs/Install.html](https://www.rdkit.org/docs/Install.html)
