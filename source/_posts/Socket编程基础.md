---
title: Socket编程基础
date: 2018-12-02 22:12:21
tags:
	- Linux
	- Socket
categories:
	- 技术
	- linux
	- 网络
---
# Socket编程之Socket基础

## 前言
Socket编程主要分为TCP／UDP／SCTP三种，每一种都有各自的优点，所以会根据实际情况决定选用何种Socket。
之后会抽取空余时间来开始深入学习Socket编程。

今天学习的是TCP编程。

## 基本流程：
TCP基本客户端与服务端的套接字函数：
* Client: 
		* socket
		* connect
		* write/read
		* close.
<!--more-->
* Server: 
		* socket
		* bind
		* listen
		* accept
		* read/write
		* close.

整个TCP过程，可以参见下图。
{% asset_img socket_process.png %}

## 三次握手
三次握手是指客户端调用connect函数发起连接请求，服务端accept函数响应时，发生的3次数据交换过程。

{% asset_img shake_hands.png %} 

## 例子
简单客户端code例子：https://github.com/SadCPPCoder/Socket/blob/master/2018.12.02/tcp/add_client.c
```c
// simple client: send two number to server, and the server reply the add result.
// author: SanCPPCoder
// date: 2018.12.02

#include <sys/types.h>
#include <sys/socket.h>
#include <stdio.h>
#include <stdlib.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <fcntl.h>
#include <string.h>
#include <unistd.h>

#define ADD_SERVER_PORT 	( 8888 )
#define ADD_SERVER_IP		( "127.0.0.1" )
#define INPUT_COUNT		( 3 )

int main(int argc, char **argv)
{
    // get the input
    if( INPUT_COUNT != argc )
    {
        perror( "Error input!\nPlease input like:\n\t\t client numberA numberB\n" );
        return -1;
    }

    double num[2] = { 0.0 };
    num[0] = atof( &argv[1][0] );
    num[1] = atof( &argv[2][0] );

    printf( "The input data is [%f] and [%f].\n", num[0], num[1] );

    // socket
    int client_sock = socket( AF_INET, SOCK_STREAM, 0 ); // IPV4, Stream, TCP

    // define the socket address of server
    struct sockaddr_in server_addr;
    bzero( &server_addr, sizeof(server_addr) );
    server_addr.sin_family = AF_INET; // IPV4
    server_addr.sin_port = htons( ADD_SERVER_PORT ); // server port
    server_addr.sin_addr.s_addr = inet_addr( ADD_SERVER_IP ); // server ip

    // connect
    if( connect( client_sock, (struct sockaddr*)&server_addr, sizeof(server_addr) ) < 0 )
    {
        perror( "Connect to server failed!\n" );
        // TODO: output error information.
        close( client_sock );
        return -1;
    }
    
    // send
    int send_len = 0;
    if( (send_len = send( client_sock, (char*)&num, sizeof(num), 0 ) ) <= 0 )
    {
        perror( "Send data failed!\n" );
        close( client_sock );
        // TODO: resend data
        return -1;
    }

    // recv
    double result = 0.0;
    if( (send_len = recv( client_sock, (char*)&result, sizeof(result), 0 ) ) < 0 )
    {
        perror( "receive data failed!\n" );
        close( client_sock );
        return -1;
    }

    printf( "Receive the result of [%f] + [%f] = [%f].\n", num[0], num[1], result );

    // close
    close( client_sock );
}
```
简单服务端code例子：https://github.com/SadCPPCoder/Socket/blob/master/2018.12.02/tcp/add_server.c
```c
// simple server: the server get two number from clients, and then reply the add result of the
// two numbers to client.
// author: SadCPPCoder
// date: 2018.12.02
#include <sys/types.h>
#include <sys/socket.h>
#include <stdio.h>
#include <stdlib.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <fcntl.h>
#include <string.h>
#include <unistd.h>

#define ADD_SERVER_PORT 	( 8888 )
#define ADD_SERVER_IP		( "127.0.0.1" )
#define INPUT_COUNT		( 3 )

int main(int argc, char **argv)
{
    // socket
    int server_sock = socket( AF_INET, SOCK_STREAM, 0 );

    // define socket address
    struct sockaddr_in server_addr;
    bzero( (char*)&server_addr, sizeof(server_addr) );
    server_addr.sin_family = AF_INET;
    server_addr.sin_port = htons( ADD_SERVER_PORT );
    server_addr.sin_addr.s_addr = htonl( INADDR_ANY );

    // bind
    if( bind( server_sock, (struct sockaddr*)&server_addr, sizeof(server_addr)) == -1 )
    {
        perror( "bind failed!\n" );
        close( server_sock );
        return -1;
    }

    // listen
    while( 1 )
    {
	if( listen( server_sock, 5 ) == -1 )
	{
	    perror( "listen failed!\n" );
	    close( server_sock );
	    return -1;
	}

	// client socket
	double num[2] = { 0.0 };
	struct sockaddr_in client_addr;
	bzero( (char*)&client_addr, sizeof(client_addr) );
	socklen_t length = sizeof(client_addr);

	int connect_fd = 0;
	if( (connect_fd = accept( server_sock, (struct sockaddr*)&client_addr, &length ) ) < 0 )
	{
	    perror( "Conncet failed!\n" );
	    close( server_sock );
	    return -1;
	}

	int len = 0;
	if( (len = recv( connect_fd, (char*)&num, sizeof(num), 0 ) ) < 0 )
	{
	    perror( "receive data failed!\n" );
	    close( connect_fd );
	    close( server_sock );
	    return -1;
	}

	double result = num[0] + num[1];

	// send data to client
	if( (len = send( connect_fd, (char*)&result, sizeof(result), 0 ) ) < 0 )
	{
	    perror( "send result failed!\n" );
	    close( connect_fd );
	    close( server_sock );
	    return -1;
	}

	close( connect_fd );
    }
    close( server_sock );

    return 0;
}
```