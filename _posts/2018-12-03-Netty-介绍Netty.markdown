---
layout: post
title:  "Netty-介绍Netty"
tags: Netty
---

## Netty是什么？

Netty是一个NIO客户端服务器框架，可以快速开发多协议服务器和客户端等网络应用程序。它简化了TCP,UDP网络编程。

Netty集成20多种丰富的协议库，如RTSP,MQTT,SMTP,HTTP以及各种二进制和基于文本的传统协议。可供开发者有选择性的添加使用。

这些框架底层都用Netty [dubbo、 Hadoop、Storm、Spark、 Elasticsearch、Springboot2.0,...] 

## 示例代码    
             
         //启动类
        public class MainServer {
            public static void main(String[] args) throws Exception {
                ServerBootstrap bootstrap = new ServerBootstrap();    // 创建Server-Bootstrap
                bootstrap.group(new NioEventLoopGroup()) // 创建Event-LoopGroup
                        .channel(NioServerSocketChannel.class) //指定所使用的NIO传输Channel
                        .childHandler(new ChannelInitializer(){
                            //我们平时会在这里添加处理数据协议, Netty会将add进去的Handler从上到下依次执行
                            @Override
                            protected void initChannel(Channel ch) throws Exception {
                                ch.pipeline().addLast(new HttpServerCodec());//Netty实现的 HTTP协议处理器
                                ch.pipeline().addLast(new HelloworldHandler()); //EchoServerHandler被标注为@Shareable，所以我们可以总是使用同样的实例
                            }
                        });
                ChannelFuture f = bootstrap.bind(8080).sync();  //异步地绑定服务器；调用sync()方法阻塞等待直到绑定完成
                f.channel().closeFuture().sync();//获取Channel的CloseFuture，并且阻塞当前线程直到它完成
            }
        }
        
         //业务处理器类
        public class HelloworldHandler extends SimpleChannelInboundHandler<HttpObject> {
                @Override
                protected void channelRead0(ChannelHandlerContext ctx, HttpObject msg) throws Exception {
                    System.out.println("Server received: " + msg);//请求数据
                    if (msg instanceof HttpRequest) {
                        HttpRequest request = (HttpRequest) msg;
                        FullHttpResponse response = new DefaultFullHttpResponse(HttpVersion.HTTP_1_1, HttpResponseStatus.OK);
                        response.content().writeBytes("hello word".getBytes());
                        response.headers().set("Content-Type", "text/plain");
                        response.headers().setInt("Content-Length", response.content().readableBytes());
        
                        if (HttpUtil.isKeepAlive(request)) {
                            response.headers().set("Connection", "keep-alive");
                            ctx.writeAndFlush(response);
                        } else {
                            ctx.writeAndFlush(response).addListener(ChannelFutureListener.CLOSE);
                        }
                    }
                }
            }
            

## 测试        
![成功](../../../images/postimg/nettyhallowoldsuccess.jpg)

## 总结
其实平时用的多的就是ChannelHandler类, 
