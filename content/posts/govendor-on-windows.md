---
title: Installing govendor on windows
date: 2018-02-26 01:01:01
---

If you are using go ans specially if you come from a .Net background you might be missing a tool like [Nuget](https://www.nuget.org/) of course `go get` solves a lot of those problems but you might be missing some configuration and control on top of that. [Govendor](https://github.com/kardianos/govendor) helps you to do that. You can check how to use govendor [here](https://zerokspot.com/weblog/2017/04/23/getting-started-with-govendor/)

This are the instructions to install in case you are using windows.

<!--more-->

Make sure your go binaries are in the path.
You should have `<GOPATH>/bin` you your windows `PATH` instructions on how to install go easy on windows can be found [here](http://www.wadewegner.com/2014/12/easy-go-programming-setup-for-windows/).

On any shell with access to go (powershell or cmd for example)

Get govendor

```shell
go get -u github.com/kardianos/govendor
```

Install govendor

```shell
go install github.com/kardianos/govendor
```

After that you should be able to check which version you are using from govendor using:

```shell
govendor --version
```

now you just need to use in your go projects

```shell
govendor init
```

and for the dependencies that you want to add

```shell
govendor fetch <Your-Dependency>
```

(this info was based on my experience and on [this answer in stack overflow](https://stackoverflow.com/a/42170134))