# Release procedure for golang


## shell

```sh
export GITHUB_TOKEN=YOURTOKEN

gobump patch -w
NOW_VERSION=`gobump show | jq '.["version"]' | sed 's/"//g'`

ghch --format=markdown --next-version=v${NOW_VERSION} >> CHANGELOG.md

if [ -e ./goxz ]; then
	rm -rf goxz
fi

goxz -pv ${NOW_VERSION} -os=linux,darwin -arch=amd64

ghr v${NOW_VERSION} goxz/
```


## Version Up

### install

```
go get -u github.com/motemen/gobump
cd `go env GOPATH`/src/github.com/motemen/gobump/cmd/gobump
go install
```

```
gobump patch -w
```

```
-const version = "0.0.1"
+const version = "0.0.2"
```

## ChangeLog

```
go get github.com/Songmu/ghch/cmd/ghch
```


```
ghch --format=markdown --next-version=v0.0.1 >> CHANGELOG.md
```

## Cross Compile

```
go get github.com/Songmu/goxz/cmd/goxz
```

```
goxz -pv 0.0.1 -os=linux,darwin -arch=amd64
```

## Release

```
go get -u github.com/tcnksm/ghr
```

```
export GITHUB_TOKEN=YOURTOKEN

ghr v0.0.1 goxz/
```

## See

+ http://memo.yuuk.io/entry/2018/03/31/171914
