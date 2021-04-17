---
title: "hot config reload with os signals "
date: 2020-04-17T13:30:03+00:00
# weight: 1
# aliases: ["/first"]
tags: ["golang", "os", "programming", "reload"]
author: "mrturkmen"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: true
hidemeta: false
comments: false
description: "Reloading config files with OS Signal"
disableHLJS: true # to disable highlightjs
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
cover:
    image: "<image path/url>" # image path/url
    alt: "<alt text>" # alt text
    caption: "<text>" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: true # only hide on current single page
editPost:
    URL: "https://github.com/mrtrkmn/mrtrkmn.github.io/tree/master/content"
    Text: "Suggest Changes" # edit text
    appendFilePath: true # to append file path to Edit link
---

Imagine a scenario where you have a monolithic application which uses a config file to store information about log directories, cert dirs and other service information. As an example to it following config file (- it is taken and modified from Haaukins project which I work on- ) can be considered:  


```yaml 

host:
    http: myapplication.mrturkmen.com
port:
    insecure: 8080
    secure: 8081

tls:
  enabled: false
  certfile: "/home/mrturkmen/certs/cert.crt"
  certkey: "/home/mrturkmen/certs/cert.key"
  cafile: "/home/mrturkmen/certs/ca.crt"

files:
  ova-directory: "/home/mrturkmen/ova"
  users-file: "/home/mrturkmen/configs/users.yml"
  exercises-file: "/home/mrturkmen/configs/exercises.yml"
  frontends-file: "/home/mrturkmen/configs/frontends.yml"

prodmode: true

vpn-service:
  grpc: "vpnservice.mrturkmen.com:4000"
  auth-key: random-auth-key
  sign-key: random-sign-key
  tls:
    enabled: true
    certfile: "/home/mrturkmen/certs/cert.crt"
    certkey: "/home/mrturkmen/certs/cert.key"
    cafile: "/home/mrturkmen/certs/ca.crt"

```

In this config file we have some set of keys which are defined to be used inside the application, however let's say we would like to update some values from the config file. Then in normal cases (-if no hot reload kind of function implemented- ), user needs to restart  entire application. It means application will have some down time, it may be less or more however it is not good way of doing it, in particular to update only a value from config file. 

In this point, os signals can be used to update config file without restarting or closing the application. There are some other libraries which it is possible to enable watch on config file mode. It means the library will immediately notify entire application when there is change on the config file. However in this scenario, I assume that there is no such a library or framework is integrated. 

Here I am considering the situation from Go language perspective, this may differ or not needed for some programming languages or freameworks. 

Channel and go routine will be used to listen any SIGHUP signal to the process of the application. 

First of all, it is nice to create the function which will re-assign config variable of the application when SIGHUP signal received. 

```go

func (a *application) ReloadConfig(confFile *string) error {
	conf, err := NewConfigFromFile(*confFile)
	if err != nil {
		return err
	}
	a.conf = conf  // re-assign applicaiton config file 
	return nil
}

```

When SIGNUP signal received by user given function above needs to be called to update configuration file. 

Here is the code which listens any SIGHUP signal to the application

```go 

func handleHotConfigReload(confFile *string, reload func(confFile *string) error) {

	c := make(chan os.Signal, 1)  // channel to wait os.Signal
	signal.Notify(c, syscall.SIGHUP)
	go func() {  // go routine to do not block other requests on the application
		<-c
		log.Info().Msgf("Hot reload for config file...")
		if err := reload(confFile); err != nil {
			log.Error().Msgf("Error on reloading config file: %s", err)
			os.Exit(1)
		}
		log.Info().Msgf("Config is updated !")
	}()
}

```

This function can be called before or after the application started. It can be called as shown below: 

```go 

handleHotConfigReload(confFilePtr, func(confFile *string) error {
		return a.ReloadConfig(confFilePtr)   // a is application struct 
	})

```
Then it can be tested with : 

```bash 
 $ kill -SIGHUP <process-id>
```

It will create SIGHUP signal on the process to call `ReloadConfig` function. 

This is how OS Signal can be used to update configuration file in an application which is written in Go, when you do not have already implemented library or framework. 






