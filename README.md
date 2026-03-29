This repository contains informational material for the [luCT](https://github.com/Sawchord/luct) project.

The presentation is build using `marp`:
```
npm install -g @marp-team/marp-cli
marp prez.md -w --html
```

Also a useful tool to check that all links work:
```
pip3 install linkchecker
linkchecker prez.html --check-extern -o text
```

    https://nightly.link/Sawchord/luct/workflows/CI/main/luct-extension.zip