---
title: "Universal gRPC client demonstration [Evans]"
date: 2020-08-08T09:19:03+00:00
# weight: 1
# aliases: ["/first"]
tags: ["grpc", "go", "client","server","cli","automated","mock-client"]
author: "mrturkmen"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: true
description: "Mock gRPC client"
disableHLJS: true # to disable highlightjs
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
cover:
    image: "../../images/github_actions.png" # image path/url
    alt: "<alt text>" # alt text
    caption: "<text>" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: true # only hide on current single page
editPost:
    URL: "https://github.com/mrtrkmn/mrtrkmn.github.io/tree/master/content"
    Text: "Suggest Changes" # edit text
    appendFilePath: true # to append file path to Edit link
---

In this post, I am going to write demo for a tool which I have just met, it is called **Evans**. It is basically universal gRPC client. What it means ? Basically when you have gRPC server and would like to test gRPC calls without creating client, you can test server side calls with **Evans**. It is known that gRPC is very common communication method between microservices, it can be used for internal and external communication. I do not have intention to explain what gRPC is in this post since it is not the purpose. 
If required [documentation of gRPC](https://grpc.io/docs/what-is-grpc/introduction/) can be investigated. 



# Create a simple gRPC server 

To demonstrate and see how **evans** works, a running gRPC should be exists, for this reason, I am going to provide a sample gRPC server. For this purpose, I will use Go programming language, however gRPC is supporting more [programming languages](https://grpc.io/docs/languages/) which you may more familiar than Go. 

gRPC server is basically an API endpoint where clients can make requests, since it is an API, first thing could be to define which methods will be used for this service. 

For simplicity and purpose of this post, I have created a basic microservice which will has four calls namely, Add,Delete,List and Find. Since the purpose is to understand how __evans__ works, the gRPC server does not need to be complex or includes lots of calls.  

## Sample repository

A sample microservice is created for demonstration purposes and all codes are available here : [https://github.com/mrturkmenhub/BookShelf](https://github.com/mrturkmenhub/BookShelf)

If you have already a running gRPC server, you can direcly pass to [demonstration of evans](#demonstration-of-evans), if not, you can clone [https://github.com/mrturkmenhub/BookShelf](https://github.com/mrturkmenhub/BookShelf) repository and test **evans** out.  

## Defining calls 

In my opinion, it is always nice to prepare *proto* file before hand, because it is like a contract which creates your main service. Let's imagine you would like to add, delete, list and find the books that you have read or wish to read. For this purpose, microservice should have at least four different calls which are Add, Delete, List and Find. There is no limit to have more calls however in order to do not get out of topic, I am keeping it small. 

Following proto file would be enough for BookShelf service. 

```proto

// it is important to declare syntax version
syntax = "proto3";
service BookShelf {
    rpc AddBook(AddBookRequest) returns (AddBookResponse) {}
    rpc ListBook (ListBooksRequest) returns (ListBooksResponse) {}
    rpc DelBook (DelBookRequest) returns (DelBookResponse){}
    rpc FindBook (FindBookRequest) returns (FindBookResponse){}
}


message AddBookRequest {
    BookInfo book = 1;
    message BookInfo {
        string isbn =1;
        string name =2;
        string author=3;
        string addedBy=4;
    }

}

message AddBookResponse {
    string message = 1;
}
message ListBooksRequest {
// no need to have anything
// could be extended to list books based on category ...
}

message ListBooksResponse {
    repeated BookInfo books =1;
    message BookInfo {
        string isbn =1;
        string name =2;
        string author=3;
        string addedBy=4;
    }
}

message  DelBookRequest {
    string isbn =1;
}

message DelBookResponse {
    string message =1;
}
message FindBookRequest {
    string isbn =1;
}

message FindBookResponse {
    Book book = 1;
    message Book {
        string isbn =1;
        string name =2;
        string author=3;
        string addedBy=4;
    }
}
```

Once **proto** file is declared, it becomes more easy to continue. For the demostration purposes, I will store information of books in memory. However, as you know, it is NOT acceptable for any production level application.


## Compile Proto file

**proto** files are great since once you have defined what you need, you can directly generate codes in available languages which are represented in gRPC [supported languages](https://grpc.io/docs/languages/). The generation of codes in your desired language is pretty straitforward, I am going to generate the code for Go programming language. 

```bash 
$  protoc -I proto/ proto/bs.proto --go_out=plugins=grpc:proto 
```

It will generate ready to use Go source code for your rpc calls which are defined in proto file. 

Afterwards, necessary piece of codes should be implemented, which are in memory store and book struct. For the aim of this post, I assumed that you have created all rest of the code as given in example gRPC server (-BookShelf-). 

**NOTE**: Generating source code through `protoc` requires to have `protoc`  tool to be installed before hand. Installation of protoc is [over here](https://developers.google.com/protocol-buffers/docs/downloads)

# Run gRPC server 

The post is not covering all aspects of gRPC, proto buffers, Go language and those are not the intention of this post. Therefore, I am assuming that you had gRPC server and would like to test out and see whether your proto contract is running correctly without creating client side codes. Once it is confirmed that your gRPC calls are running without encouraging any unseen problems, then creating client side code will be much easy without any problem. 

You can start gRPC server with: 

```bash 
 $ go run server/main.go
 BookShelf gRPC server is running ....
```

Once gRPC server is up and running, you can use **evans** tool for inspecting gRPC server for available inquires. 


# Demonstration of EVANS

Evans is an open source project which is available [at Github](https://github.com/ktr0731/evans) and I found it pretty useful, in particular, for people who have no idea what kind of calls are available in proto file, as it states its explanation, it is universal gRPC client. Installation of evans and more information is given its [readme file](https://github.com/ktr0731/evans/blob/master/README.md). 

It has plenty of features  which are very handy to use for automating and testing some stuff on top of existing gRPC server or new one. 

Let's make a demo, when you are using **evans** your gRPC server should be up and running, in order to make communication with universal gRPC client - **evans**. I assumed that you have followed [readme file](https://github.com/ktr0731/evans/blob/master/README.md) of **evans** and installed it correctly. 


_Note that port 9000 is given because it is port of gRPC server._


```bash 
❯ evans -r -p 9000

  ______
 |  ____|
 | |__    __   __   __ _   _ __    ___
 |  __|   \ \ / /  / _. | | '_ \  / __|
 | |____   \ V /  | (_| | | | | | \__ \
 |______|   \_/    \__,_| |_| |_| |___/

 more expressive universal gRPC client

BookShelf@127.0.0.1:9000> show services 

+-----------+----------+------------------+-------------------+
|  SERVICE  |   RPC    |   REQUEST TYPE   |   RESPONSE TYPE   |
+-----------+----------+------------------+-------------------+
| BookShelf | AddBook  | AddBookRequest   | AddBookResponse   |
| BookShelf | ListBook | ListBooksRequest | ListBooksResponse |
| BookShelf | DelBook  | DelBookRequest   | DelBookResponse   |
| BookShelf | FindBook | FindBookRequest  | FindBookResponse  |
+-----------+----------+------------------+-------------------+

BookShelf@127.0.0.1:9000> 

```

As you can observe all calls which are used in proto file can be used through **evans**, moreover its usage is pretty straitforward. 

You can check demonstration video below. 

⚠️ **_You may wish to change the video quality to 1080p60_**

<iframe width="810" height="500" src="https://www.youtube.com/embed/GnAUkPUXYCs" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen ></iframe>

If you have any questions, fix, or something else, do not hesitate to contact with me. 
