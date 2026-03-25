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