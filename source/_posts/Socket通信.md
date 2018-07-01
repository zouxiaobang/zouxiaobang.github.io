---
title: Socket通信
date: 2016-09-27 10:35:33
tags: [android, java, Socket]
categories: [Android, Java]
---

## Socket与Http

说到Socket，就想到聊天室。对，Socket一个很大的优点就是聊天室的模式。同样都是网络通信，它与Http还是有很大的差别的：
	Http：请求 <--> 响应模式
	Socket：连接 --> 直接进行数据交换

![Http通信模式](/img/socket/socket1.png)

Http是先将客户端和服务端进行连接，然后由客户端发送请求，在服务端接收到请求之后进行逻辑处理，完毕之后返回响应给客户端。如果客户端没有发送请求，则服务端不会主动发送响应给客户端，即不会主动推送信息。

![Socket通信模式](/img/socket/socket2.png)

Socket的客户端是通过ServerSocket与服务端进行连接的，只要连接得上，就可以直接进行通信，而不用等待请求什么的。

## Socket的使用

Socket的使用虽然比较简单，但是其中有很多细节需要注意，一个不小心，就足以让你崩溃(我就是例子，弄了整整一天，换了不知多少种方式，到最后才发现问题，总是那么小)。

### 1、服务端

先来编写一个服务端，这里服务端是使用Java代码来实现

``` bash
public class MyServer{
	public static List<Socket> sockets = new ArrayList<Socket>();

	public static void main(String[] args) throws Exception{
		//这是服务端的ServerSocket，端口号为2121
		ServerSocket ss = new ServerSocket(2121);

		//通过循环来不断获取来自客户端的连接
		while(true){
			//在这里线程会阻塞，直到有客户端连接
			Socket socket = ss.accept();
			sockets.add(socket);
			//每当有一个客户端连接，则开启一个线程来为客户端服务
			new Thread(new ServerThread(socket)).start();
		}
	}
}

class ServerThread implements Runnable{
	private Socket socket;
	private BufferedReader br;

	public ServerThread(Socket socket){
		this.socket = socket;
		br = new BufferedReader(new InputStreamReader(socket.getInputStream()));
	}	

	public void run(){
		String line = null;
		try{
			while((line = br.readLine()) != null){
				System.out.println("FROM CLIENT: " + line);

				//向客户端发送信息
				OutputStream os = socket.getOutputStream();
				os.write("I'm server\n".getBytes("utf-8"));
			}
		} catch(Exception e){
			MyServer.sockets.remove(socket);
		}
	}
}
```

注意：
* Socket要作为List的泛型，只能改为SE5，这个Eclipse可以自动更改。
* sockets的作用是为了让服务端可以向所有连接的客户端发送信息。
* 在服务端发送数据给客户端时记得使用\n，否则客户端没法收到信息，因为客户端也是使用BufferedReader来接收信息，这个流的特点在于每次都需要读到换行符才会读取结束。

#### 1、如果你想不用添加\n这么麻烦的东西的话，你可以不用使用BufferedReader这个输入流

``` bash
InputStream is = socket.getInputStream();
byte[] bus = new byte[1024];
while (is.read(bus) != -1){
	System.out.println(new String(bus, 0, bus.length));
}
```

#### 2、如果想向所有的客户端都发送信息(例如聊天室聊天)，则在向客户端发送信息处修改代码

``` bash
for(Iterator<Socket> it = MyServer.sockets.iterator(); it.hasNext(); ){
	Socket s = it.next();
	OutputStream os = socket.getOutputStream();
	os.write("I'm server\n".getBytes("utf-8"));
}
```

### 2、客户端

客户端使用Socket来连接服务端

``` bash
public class MyClient{
	private InputStream is;
	private OutputStream out;

	public static void main(String[] args) throws UnknownHostException, IOException{
		while(true){
			Socket socket = new Socket("127.0.0.1", 2121);

			is = socket.getInputStream();
			out = socket.getOutputStream();

			//开启一个新线程来不断获取来自服务端的信息
			new Thread(){
				public void run(){
					try{
						byte[] bus = new byte[1024];
						while((is.read(bus)) != -1){
							System.out.println("FROM SERVER: " + new String(bus, 0, bus.length));
						}
					} catch(Exception e){

					}
				}
			}.start();

			String content = new Scanner(System.in).nextLine();
			os.write(content.getBytes("utf-8"));
		}
	}
}
```

Android端的客户端代码与Java的大同小异，就不贴出来了。主要是思想
















