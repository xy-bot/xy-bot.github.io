---
title: python socket理解
date: 2022-05-29 16:59:23
tags: 
- python
categories:
- python重点理解
---
## socket
所谓套接字(Socket)，就是对网络中不同主机上的应用进程之间进行双向通信的端点的抽象。一个套接字就是网络上进程通信的一端，提供了应用层进程利用网络协议交换数据的机制。从所处的地位来讲，套接字上联应用进程，下联网络协议栈，是应用程序通过网络协议进行通信的接口，是应用程序与网络协议栈进行交互的接口。
### 创建socket方式
#### 普通创建
```
socket.socket(family=-1, type=-1, proto=-1, fileno=None):
	family: 地址族：
		AF_UNIX:unix本机通信
		AF_INET: 默认值,ipv4
		AF_INET6: 当程序使用到ipv6时使用
	type: 套接字类型
		SOCK_STREAM：默认值；基于tcp,可靠传输，适用于对消息内容要求的情况，例如文件下载等
		SOCK_DGRAM：基于udp，广播消息，适用于对消息内容不是太准确的情况，例如视频通话、直播等
		SOCK_RAW：
	proto: 协议号编号
		协议编号通常为零，可以省略，
		或者在地址系列为AF_CAN的情况下，协议应为CAN_RAW、CAN_BCM、CAN_ISOTP或CAN_J1939之一。
	fileno: 指定文件描述符
		如果指定了fileno，则会从指定的文件描述符中自动检测family、type和proto的值，通常不会使用，此参数指定后，其他参数将失效
```
#### 创建一对已经连接的套接字对象
```
socketpair(family=None, type=SOCK_STREAM, proto=0)
	相当于创建一对已经连接的socket套接字,包含server和client
	默认是全双工模式,既可以发,也可以收
	默认是创建unix套接字(AF_UNIX)
	参数与socket基本一致
	使用场景：
		import socket
		from multiprocessing import Process, Pipe
		"""
		进程间双向通信, 相当于进程间通信的PIPE，只支持Linux
		"""
		socket1, socket2 = socket.socketpair()
		def process1():
			print(socket1)
			print(socket2)
			# 关闭不需要的socket
			socket2.close()
			socket1.send(b"hello")

		def process2():
			print(socket1)
			socket1.close()
			print(socket2)
			print(socket2.recv(1024))
		
		# pipe　简易使用
		conn1, conn2 = Pipe()
		def process3():
			print(conn1)
			conn1.send(b"hello")

		def process4():
			print(conn2)
			print(conn2.recv())
		if __name__ == "__main__":
			p1 = Process(target=process1)
			p2 = Process(target=process2)
			p1.start()
			p2.start()
			p3 = Process(target=process3)
			p4 = Process(target=process4)
			p3.start()
			p4.start()
－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－
		"""
		线程间双向通信:支持;Linux\Windows
		"""
		import threading
		socket1, socket2 = socket.socketpair()
		def thread1():
			print(socket1)
			socket1.send(b"hello")
		def thread2():
			print(socket2)
			print(socket2.recv(1024))

		t1 = threading.Thread(target=thread1)
		t2 = threading.Thread(target=thread2)
		t1.start()
		t2.start()
```
#### 创建连接基于tcp的socket对象(客户端)
```
create_connection(address, timeout=_GLOBAL_DEFAULT_TIMEOUT, source_address=None)
	address: 需要连接的tcp服务的地址(host, port)
	timeout: 连接的超时时间
	source_address: 将socket对象绑定到对应地址上(host, port),会在服务端上显示对应此地址
	只支持tcp,udp无法连接
	它将尝试为AF_INET和AF_INET6解析它，然后尝试依次连接到所有可能的地址，直到连接成功。
	这使得编写同时兼容IPv4和IPv6的客户端变得容易。
```
#### 创建基于tcp的socket对象(服务端)，python需在3.8以上
```
create_server(address, *, family=AF_INET, backlog=None, reuse_port=False, dualstack_ipv6=False)
	address: 绑定的地址
	family: 地址族: AF_INET or AF_INET6
	backlog: 监听的队列数，相当于 socket.listen()
	reuse_port: 指示是否使用SO_REUSEPORT套接字选项; SO_REUSEPORT: 重用端口
	dualstack_ipv6: 是否支持ipv6, 如果为True, 则family必须为AF_INET6,如果设备支持，则可以同事接收ipv4和ipv6的数据
	实例：
		with create_server(('', 8000)) as server:
		...     while True:
		...         conn, addr = server.accept()
		...         # handle new connection
不需要手动绑定监听，方便了不少
在POSIX平台上，设置了SO\u REUSEADDR socket选项，以便立即重用以前绑定在同一地址上并保持TIME\u WAIT状态的套接字。
```
#### 其他方法
```
# python3.8以上支持:  如果平台支持处理tcp的ipv4和ipv6的连接，则返回True, 否则返回False
socket.has_dualstack_ipv6()
# 从给定的文件副本中创建套接字对象，fd:文件副本, 其余与socket一致
socket.fromfd(fd, family, type, proto=0)
# 从socket.share()中创建套接字,此套接字默认是阻塞的 只支持windows
socket.fromshare()
# 关闭文件描述符, python 3.7新增;类似于os.close,但对于socket来说，在一些平台上，尤其是windows平台，无法正常关闭文件描述符
socket.close()
# 将主机和端口解析为地址信息条目列表。返回的为(family, type, proto, canonname, sockaddr)元组
socket.getaddrinfo(host, port, family=0, type=0, proto=0, flags=0)
# 从名称中获取完整的域名，name为空或为0.0.0.0则调用gethostname返回主机名，如果name不为空，则调用gethostbyaddr根据地址获取主机名
socket.getfqdn(name='')
socket.gethostname()
# 此处的name类似于address
socket.gethostbyaddr(name)
不常用方法更多详见官网
```
#### socket方法
```
# 接受连接。套接字必须绑定到地址并侦听连接。返回值是一对（conn，address），其中conn是一个新的套接字对象，可用于在连接上发送和接收数据，address是绑定到连接另一端的套接字的地址。　一般为服务端
socket.accept()
# 将套接字绑定到地址, address:(host, port)　一般为服务端
socket.bind(address)
# 关闭socket连接，注意在socket使用完后最好关闭socket
socket.close()
# 连接远程的地址, 如果超时，则会报超时TimeoutError，出现信号异常时会抛出InterruptedError, 一般为客户端
socket.connect(address)
# 连接远程的地址, 但对于C-level connect（）调用返回的错误，返回一个错误指示符，而不是引发异常（其他问题，例如“找不到主机”，仍然可能引发异常）。如果操作成功，错误指示器为0，否则为errno变量的值。这对于支持异步连接非常有用。一般为客户端
socket.connect_ex(address)
# 将套接字对象置于关闭状态，而不实际关闭底层文件描述符。文件描述符将被返回，并可用于其他目的。
socket.detach()
# 复制socket
socket.dup()
# 返回套接字的文件描述符（一个小整数），失败时返回-1。这对select.select()很有用
socket.fileno()
# 获取套接字的文件描述符或套接字句柄的可继承标志：如果套接字可以在子进程中继承，则为True；如果不能，则为False。
socket.get_inheritable()
# 返回套接字连接的远程地址。(host, port); 例如，这对于查找远程IPv4/v6套接字的端口号很有用。在某些系统上不支持此功能。一般为客户端
socket.getpeername()
# 返回套接字自己的地址。例如，这对于查找IPv4/v6套接字的端口号很有用。主机自身的
socket.getsockname()
# 返回给定套接字选项的值（请参阅Unix手册页getsockopt（2））。所需的符号常数（SO_*等）在本模块中定义。如果不存在buflen，则假定一个整数选项，其整数值由函数返回。如果存在buflen，它将指定用于在中接收选项的缓冲区的最大长度，该缓冲区将作为字节对象返回。由调用者来解码缓冲区的内容（请参阅可选的内置模块struct，了解解码编码为字节字符串的C结构的方法）。
socket.getsockopt(level, optname[, buflen])
# 如果套接字处于阻塞模式，则返回True；如果处于非阻塞模式，则返回False。python 3.7新增
socket.getblocking()
# 返回与套接字操作相关的超时（以秒为单位），如果未设置超时，则返回无。这反映了对setblocking（）或settimeout（）的最后一次调用。
socket.gettimeout()
# WSAIoctl系统接口的有限接口。只支持windows;有关更多信息，请参阅Win32文档。在其他平台上，通用fcntl。fcntl（）和fcntl。可以使用ioctl（）函数；它们接受套接字对象作为第一个参数。目前只支持以下控制代码：SIO_RCVALL、SIO_KEEPALIVE_VALS和SIO_LOOPBACK_FAST_PATH。
socket.ioctl(control, option)
# 允许服务器接受连接。如果指定了backlog，则它必须至少为0（如果较低，则设置为0）；它指定系统在拒绝新连接之前允许的未接受连接数。如果未指定，则选择默认的合理值。
socket.listen(backlog)；一般用于服务端
# 返回与套接字关联的文件对象。返回的确切类型取决于makefile（）的参数。这些参数的解释方式与内置的open（）函数相同，只是支持的模式值只有'r'（默认值）、'w'和'b'。插座必须处于阻塞模式；它可以有一个超时，但如果超时，文件对象的内部缓冲区可能会以不一致的状态结束。关闭makefile（）返回的文件对象不会关闭原始套接字，除非所有其他文件对象都已关闭，并且已打开套接字。已对套接字对象调用close（）。
socket.makefile(mode='r', buffering=None, *, encoding=None, errors=None, newline=None)
# 从套接字接收数据。返回值是一个字节对象，表示接收到的数据。一次接收的最大数据量由bufsize指定。有关可选参数标志的含，一般用于tcp连接
socket.recv(bufsize, flags);
# 从套接字接收数据。返回值是一对（字节，地址），其中字节是表示接收到的数据的字节对象，地址是发送数据的套接字的地址,一般用于udp连接的客户端
socket.recvfrom(bufsize, flags)

# 不太熟悉
socket.recvmsg(bufsize[, ancbufsize[, flags]])
socket.recvmsg_into(buffers[, ancbufsize[, flags]])
# 从套接字接收数据，将其写入缓冲区，而不是创建新的bytestring。返回值是一对（nbytes，address），其中nbytes是接收的字节数，address是发送数据的套接字的地址。
socket.recvfrom_into(buffer[, nbytes[, flags]])
socket.recv_into(buffer[, nbytes[, flags]])¶


# 发送字节数据到远程的socket，返回发送数据的字节数，一般用于客户端，可能会出现返回的字节数小于实际的发送数，当数据未发送完成时，需由应用程序检查是否完成发送，所以需要重复发送数据，
socket.send(bytes, flags)
# 发送字节数据到远程的socket,一般用于客户端，持续发送，无法确认发送了多少数据
socket.sendall(bytes[, flags])
# 发送字节数据，不需要连接远程的socket, 指定发送的地址，udp发送，点对点, 返回发送数据的字节数
socket.sendto(bytes, flags, address)

# 不太熟悉
socket.sendmsg(buffers[, ancdata[, flags[, address]]])
socket.sendmsg_afalg([msg, ]*, op[, iv[, assoclen[, flags]]])
socket.sendfile(file, offset=0, count=None)
socket.set_inheritable(inheritable)

# 设置套接字的阻塞或非阻塞模式：如果标志为false，则将套接字设置为非阻塞，否则设置为阻塞模式。
socket.setblocking(flag)
# 设置连接超时时间，作用同上, python3.7后socket type已添加SOCK_NONBLOCK flag 
socket.settimeout(value)
sock.setblocking(True) 相当于sock.settimeout(None)
sock.setblocking(False) 相当于sock.settimeout(0.0)

# 对socket的拓展，作用于套接字，设置套接字的相关属性
socket.setsockopt(level, optname, None, optlen: int)

# 关闭连接，发送和接收，how为SHUT_RD，接收不被允许；how为SHUT_WR，发送不被允许；how为SHUT_RDWR，发送接收都不被允许，一般建议先close在shutdown, 保证数据完全发送
socket.shutdown(how)
# 复制套接字并准备与目标进程共享。必须为目标进程提供进程id。然后，可以使用某种形式的进程间通信将生成的字节对象传递给目标进程，并使用fromshare（）在那里重新创建套接字。一旦调用了这个方法，就可以安全地关闭套接字，因为操作系统已经为目标进程复制了它。
socket.share(process_id)
```
#### 套接字创建注意事项
套接字对象可以采用三种模式之一：阻塞、非阻塞或超时。默认情况下，套接字始终在阻塞模式下创建，但可以通过调用setdefaulttimeout（）来更改。
　　在阻塞模式下，操作会阻塞直到完成或系统返回错误（例如连接超时）。
　　在非阻塞模式下，如果操作不能立即完成，操作将失败（不幸的是，错误取决于系统）：来自 的函数 select可用于了解套接字何时以及是否可用于读取或写入。
　　在超时模式下，如果无法在为套接字指定的超时时间内完成操作（它们引发timeout异常）或系统返回错误，则操作将失败。
　　在操作系统级别，超时模式下的套接字在内部设置为非阻塞模式。此外，阻塞和超时模式在引用同一网络端点的文件描述符和套接字对象之间共享。如果您决定使用套接字的fileno（），此实现细节可能会产生明显的后果。
##### 超时和connect方法
connect（）操作也受超时设置的影响，通常建议在调用connect之前调用settimeout，或传递超时参数到create_connection。然而，无论Python套接字超时设置如何，系统网络堆栈也可能返回自己的连接超时错误。
##### 超时和accept方法
如果getdefaulttimeout（）不是None，则accept（）方法返回的套接字将继承该超时。否则，行为取决于监听套接字的设置：
如果监听套接字处于阻塞模式或超时模式，则accept（）返回的套接字处于阻塞模式；
如果侦听套接字处于非阻塞模式，则accept（）返回的套接字是处于阻塞模式还是非阻塞模式取决于操作系统。如果要确保跨平台行为，建议手动覆盖此设置。
#### recv, send等相关携带flag参数展示
```
# 使用SCM_权限操作（如UNIX（7）所述），为通过UNIX域文件描述符接收的文件描述符设置close on exec标志。该标志的用途与open（2）的O_CLOEXEC标志相同。
MSG_CMSG_CLOEXEC (recvmsg() only; since Linux 2.6.23) 
# 启用非阻塞操作；如果操作会阻塞，则调用会失败，并出现错误EAGAIN或EWOLDBLOCK。
MSG_DONTWAIT (since Linux 2.2) 
# 此标志指定应从套接字错误队列接收排队的错误。错误在一条辅助消息中传递，其类型取决于协议（对于IPv4 IP_RECVERR）。用户应提供足够大小的缓冲区。
MSG_ERRQUEUE (since Linux 2.2) 
# 此标志请求接收正常数据流中不会接收到的带外数据。一些协议将加速数据放在正常数据队列的前端，因此该标志不能与此类协议一起使用。
MSG_OOB
# 此标志使接收操作从接收队列的开头返回数据，而不从队列中删除该数据。因此，后续的接收呼叫将返回相同的数据。
MSG_PEEK
# 对于原始（AF_数据包）、Internet数据报（自Linux 2.4.27/2.6.8起）、netlink（自Linux 2.6.22起）和UNIX数据报（自Linux 3.4起）套接字：返回数据包或数据报的实际长度，即使它比传递的缓冲区长。
MSG_TRUNC (since Linux 2.2)
# 此标志请求操作块停止，直到满足完整请求。然而，如果捕捉到信号、发生错误或断开连接，或者要接收的下一个数据与返回的数据类型不同，则呼叫返回的数据可能仍然少于请求的数据。此标志对数据报套接字无效。对于接收大数据
MSG_WAITALL (since Linux 2.2) 
```
### 例子
#### 一对一发送：只能接收一次：tcp，接收完自动停止
##### 客户端
```
import socket

HOST = "192.168.57.207"
PORT = 4003
BUFFERSIZE = 1024

with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as client:
    client.connect((HOST, PORT))
    client.sendall(b"hello")
    reply_data = client.recv(BUFFERSIZE)
    print(f"reply_data:{reply_data}")
```
##### 服务端
```
import socket

HOST = ""
PORT = 4003
BUFFERSIZE = 1024

def connect_one_once():
    """
    一对一发送:只能接收一次
    """
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as server:
        server.bind((HOST, PORT))
        server.listen(4)
        print("server start")
        conn, addr = server.accept()
        with conn:
            print(f"connect addr:{addr}")
            while True:
                data = conn.recv(BUFFERSIZE)
                if data:
                    break
                print(f"server receive data:{data}")
                conn.send(data)
```
#### 一对多发送：可以接收一次：tcp，接收完可以继续接收，不过默认socket会阻塞
##### 客户端
同上客户端
##### 服务端
```
import socket

HOST = ""
PORT = 4003
BUFFERSIZE = 1024

def connect_muti_more():
    """
    一对多发送:可接收多次
    """
    import time
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as server:
        server.bind((HOST, PORT))
        server.listen(4)
        print("server start")
        while True:
            conn, addr = server.accept()      
            with conn:
                print(f"connect addr:{addr}")
                data = conn.recv(BUFFERSIZE)
                time.sleep(20)      
                print(f"server receive data:{data}")
                conn.send(data)
```
#### socket发送大数据及接收数据
##### 客户端
同上，只是数据超过可buffersize
##### 服务端
```
import socket

HOST = ""
PORT = 4003
BUFFERSIZE = 1024

def connect_muti_large():
    """
    发送大数据
    """
    import time
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as server:
        server.bind((HOST, PORT))
        server.listen(4)
        print("server start")
        while True:
            conn, addr = server.accept()      
            print(f"connect addr:{addr}")
            while True:
                data = conn.recv(BUFFERSIZE, socket.MSG_WAITALL)
                if not data: break
                print(f"server receive data:{data}")
                conn.send(data)
```
#### udp通信
##### 客户端
udp无需连接对应地址，直接进行点对点发送即可
```
import socket

HOST = "192.168.57.207"
PORT = 4003
BUFFERSIZE = 1024

with socket.socket(socket.AF_INET, socket.SOCK_DGRAM) as client:
    client.sendto(b"world", (HOST, PORT))
```
##### 服务端
服务端无需监听端口，直接接收对应数据即可
```
import socket

HOST = ""
PORT = 4003
BUFFERSIZE = 1024

def connect_udp():
    with socket.socket(socket.AF_INET, socket.SOCK_DGRAM) as server:
        server.bind((HOST, PORT))
        data = server.recvfrom(BUFFERSIZE)
        print(f"receive data:{data}")
```
## 使用场景
### 局域网udp广播
实现同一局域网，广播传播消息，同一设备的客户端只能绑定一个，可通过server.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)实现多个文件同一设备绑定同一个端口
#### 服务端
```
import socket

HOST = "<broadcast>"
PORT = 4003
BUFFERSIZE = 1024

def udp_server():
    with socket.socket(socket.AF_INET, socket.SOCK_DGRAM) as client:
        client.setsockopt(socket.SOL_SOCKET, socket.SO_BROADCAST, 1)
        data = "hello"
        client.sendto(data.encode("utf-8"), (HOST, PORT))
        
udp_server()
```
#### 客户端
```
import socket

HOST = ""
PORT = 4003
BUFFERSIZE = 1024


def udp():
    with socket.socket(socket.AF_INET, socket.SOCK_DGRAM) as server:
　　# 一台机器可绑定同一个端口
        server.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
        server.setsockopt(socket.SOL_SOCKET, socket.SO_BROADCAST, 1)
        server.bind((HOST, PORT))
        while True:
            data, addr = server.recvfrom(BUFFERSIZE)
            print(f"addr:{addr}, data:{data}")

udp()
```
### 一（服务端）对多（客户端）通信
默认情况下，socket是阻塞的，意思就是服务端在接收到一个客户端的请求后必须要处理完，下一个客户端才可以连接，为了确保一对多的连接，可以采用多线程、socketServer、selectIO多路复用等方式实现一对多通信
### 客户端
可以使用多个客户端来测试服务端
```
import socket

HOST = "192.168.xx.xxx"
PORT = 4001
BUFFERSIZE = 1024
with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as client:
    client.connect((HOST, PORT))
    client.send(b"hello")
    data = client.recv(BUFFERSIZE)
    print(data)
```
#### 多线程方式
```
import socket, time,threading

HOST = ""
PORT = 4001
BUFFERSIZE = 1024

def recv_msg(server):
    conn, addr = server.accept()
    print(f"addr:{addr}")
    data = conn.recv(BUFFERSIZE)
    time.sleep(10)
    print(f"data:{data}")
    conn.send(data)

with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as server:
    server.bind((HOST, PORT))
    server.listen(10)
    while True:
        t = threading.Thread(target=recv_msg, args=(server,))
        t.daemon = True
        t.start()
```
#### 重写socketServer
```
import socketserver

class MySocketServer(socketserver.BaseRequestHandler):
    """
        def __init__(self, request, client_address, server):
        self.request = request
        self.client_address = client_address
        self.server = server
        self.setup()
        try:
            self.handle()
        finally:
            self.finish()

    def setup(self):
        pass

    def handle(self):
        pass

    def finish(self):
        pass

    """

    def setup(self) -> None:
        print(f"request:{self.request}")
        print(f"client_address:{self.client_address}")
        print(f"server:{self.server}")
        return super().setup()
    
    def handle(self) -> None:
        data = self.request.recv(BUFFERSIZE)
        print(f"data:{data}")
        time.sleep(10)
        self.request.send(data)
        return super().handle()
# 与默认的socket一样，也会阻塞
# server = socketserver.TCPServer((HOST, PORT), RequestHandlerClass=MySocketServer)
# 使用类似创建线程的方式，避免阻塞， Windows、Linux通用
server = socketserver.ThreadingTCPServer((HOST, PORT), RequestHandlerClass=MySocketServer)
# 使用创建进程的方式。避免阻塞，只适用于Linux
# server = socketserver.ForkingTCPServer((HOST, PORT), RequestHandlerClass=MySocketServer)
# serve_forever(self, poll_interval=0.5)：每隔固定时间进行轮训， 默认0.5
server.serve_forever()
```
#### select IO多路复用
```
import select, socket
from queue import Queue



HOST = ""
PORT = 4001
BUFFERSIZE = 1024

server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
# 设置共用地址端口，防止出现冲突
server.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
server.setblocking(False)
server.bind((HOST, PORT))
server.listen(10)

# 依次轮询readable、writeable、exceptional
# 期望去读的服务端socket以及服务端接收到的客户端的socket
inputs = [server]
# 期望去写的socket（客户端）发送数据
outputs = []
# 存放发送接收消息的字典
message_queue = {}

"""
readable, writeable, exceptional = select.select(inputs, outputs, inputs, timeout)
    inputs: 监听可读的套接字 外部发来的数据，服务端接收的客户端
    outputs: 监听可写的套接字 监控并接收所有发出去的数据
    inputs: 监听错误信息 
    timeout: 监听的时间限制 默认是0.05
    readable: 可读列表 服务端socket, 以及默认接收的客户端
    writeable: 可写列表 运行过程产生的socket 服务端接收的客户端
    exceptional: 错误信息  异常列表
readable对应inputs中数据
writeable对应outputs数据
"""

while inputs:
    # 服务端socket，运行过程产生的socket：客户端，异常列表
    readable, writeable, exceptional = select.select(inputs, outputs, inputs)
    # 处理服务器端的socket及包括接收到的客户端socket
    for s in readable:
        if s is server:
            # 服务端本身
            conn, addr = s.accept()
            # 设置非阻塞
            conn.setblocking(False)
            # 将接受到的socket添加到
            inputs.append(conn)
            message_queue[conn] = Queue()
        else:
            # 服务端接收的客户端的socket：inputs.append(conn)
            data = s.recv(BUFFERSIZE)
            print(f"{s.getpeername()}  recv data:{data}")
            if data:
                # 客户端与服务端保持连接，客户端向服务端发送数据
                message_queue[s].put(data)
                # 将接收的客户端添加到outputs用于服务端向客户端发送数据
                if s not in outputs:
                    outputs.append(s)
            else:
                # 无消息接收，客户端与服务端断开连接
                # 删除outputs中将要发送消息的socket
                if s in outputs:
                    outputs.remove(s)
                if s in inputs:
                    inputs.remove(s)
                # 关闭客户端socket
                s.close()
                # 清除消息字典中的对应socket
                del message_queue[s]

    # 连接到服务端的conn 服务端向客户端发送数据
    for s in writeable:

        try:
            message = message_queue.get(s, None)
            if message is not None:
                # 从消息字典中拿取数据
                data = message.get_nowait()
            else:
                # 消息字典中无对应数据, 客户端断开
                print(f"socket has no data , close")
                # if s in outputs:
                #     outputs.remove(s)
        except Exception as e:
            # 客户端断开
            print(f"send data error")
            print(f"{s.getpeername()} close")
        else:
            # 如果存在对应消息字典数据则发送
            if message is not None:
                s.send(data)
            else:
                print(f"socket has no data , close")

    # 异常处理
    for s in exceptional:
        print("exist exception")
        # 出现异常，从inputs删除socket
        inputs.remove(s)
        # 如果在outputs中存在则删除
        if s in outputs:
            outputs.remove(s)
        if s in inputs:
            inputs.remove(s)
        # 关闭socket
        s.close()
        # 删除socket相应的消息字典数据
        del message_queue[s]
        
    # print(readable)
    # print(writeable)
    # print(exceptional)
```
#### select服务端接收客户端的一次流程
```
select接收一次客户端流程
第一次轮询：服务端
    inputs:[服务端socket]
    outputs:[]
	readable:[服务端socket]
	writeable:[]
	exceptional:[]
	轮询监听的服务端socket，将接收到的客户端添加至监听可读列表 ,并对客户端创建消息队列并添加至消息字典中
    inputs:[服务端socket, 接收到的客户端socket]
第二次轮询：服务端接收到的客户端， 接收消息
    inputs:[服务端socket, 接收到的客户端socket]
    outputs:[]
    readable:[接收到的客户端socket]
    writeable:[]
    exceptional:[]
    轮询监听接收到的客户端的socket， 接收客户端的数据并将数据添加至对应消息队列中
    outputs:[接收到的客户端socket]
    readable:[]
第三次轮询：服务端接收到的客户端， 服务端发送消息给客户端
    inputs:[服务端socket, 接收到的客户端socket]
    outputs:[接收到的客户端socket]
    readable:[]
    writeable:[接收到的客户端socket]
    exceptional:[]
    接收到的客户端socket获取消息队列中的数据并向客户端发送数据

第四次轮询：
    inputs:[服务端socket, 接收到的客户端socket]
    outputs:[接收到的客户端socket]
    readable:[接收到的客户端socket]
    writeable:[接收到的客户端socket]
    exceptional:[]
    继续接收数据，由于没有数据则从可写（发送数据）列表、可读列表（服务端接收）中移除并关闭客户端socket、删除对应消息字典中的队列
    inputs:[服务端socket]
    outputs:[]
    readable:[接收到的客户端socket(closed)]
    writeable:[接收到的客户端socket(closed)]

```
### 检测端口是否被占用
通过连接对应端口，是否出现目标计算机拒绝来判断端口是否被占用，被占用则返回True，反之则为False，无需在执行命令查看端口是否被占用
Windows查看端口占用命令
```
netstat -ano | findstr port
```
Linux查看端口占用命令
```
lsof -i:port
```
```
def check_port_used(port):
    try:
        client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        client.connect(("127.0.0.1", port))
        return True
    except Exception as e:
        print(f"check_port_used error:{e}")
        return False
    finally:
        client.close()
```

### 跨语言通信（同一台设备），不同设备的采用RPC
需要注意的是如果使用bufferedReader.readLine()读取服务端发送的数据时默认回阻塞，只有读到\n或者\r时才会返回
#### python服务端
```
import socket
import time
HOST = ""
PORT = 4001
BUFFERSIZE = 1024

def socket_demo_tcp():
    """
    socket.socket(family=-1, type=-1, proto=-1, fileno=None):
    """
    server_socket = socket.socket(family=socket.AF_INET, type=socket.SOCK_STREAM)
    server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    server_socket.bind((HOST, PORT))
    server_socket.listen(1)
    while True:
        conns, address = server_socket.accept()
        print(f"conns:{conns}, address:{address}")
        msg = conns.recv(BUFFERSIZE)
        time.sleep(5)
        print(f"msg:{msg}")
        conns.send("reply\n".encode("utf-8"))

socket_demo_tcp()
```
#### java 客户端
```
import java.io.BufferedReader;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.io.OutputStream;
import java.io.PrintWriter;
import java.net.Socket;
import java.text.SimpleDateFormat;
import java.util.Date;

public class socket{

    private static final String HOST_ADDRESS = "127.0.0.1";
    private static final Integer PORT = 4001;


    public static void main(String[] args) {
        try {
        
            Socket socket = new Socket(HOST_ADDRESS, PORT);
            //获取输出流，向服务器端发送信息
            OutputStream os = socket.getOutputStream();
            //字节输出流
            PrintWriter pw = new PrintWriter(os);
            String data = "hello";
            //将输出流包装为打印流
            pw.write(data + "\n");
            // 刷新缓冲区，发送数据
            pw.flush();
            //接收服务器的输入流
            InputStream inputStream = socket.getInputStream();
            //读取服务器的输入流
            BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(inputStream));
            System.out.println("read start");
            //读取服务器输入流并输出
            String info = bufferedReader.readLine();
            System.out.println("read finish");
            if (info != null) {
                System.out.println("接收数据: " + ": 返回数据为：" + info);
            } 
            bufferedReader.close();
            inputStream.close();
            socket.close();
        } catch (Exception e) {
            System.out.println("SendMsgThread: exceptionMsg: " + e.getMessage());
        }
    }

}
```
### 
## 补充
### close与shutdown问题
```
shutdown():参数
socket.SHUT_RD：关闭读操作，即recv
socket.SHUT_WR：关闭写操作，即send
socket.SHUT_RDWR：关闭读和写操作，即send recv
# 如下内容发出消息后就关闭了读，因此无法获取到服务端返回给客户端的数据
def shutdown():
    client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    client.connect((HOST, PORT))
    client.send(b"hello")
    client.shutdown(socket.SHUT_RD)
    data = client.recv(BUFFERSIZE)
    print(f"client data:{data}")

# 如下客户端发送数据后关闭了写，因此还可以获取到服务端返回给客户端的数据
def shutdown():
    client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    client.connect((HOST, PORT))
    client.send(b"hello")
    client.shutdown(socket.SHUT_WR)
    data = client.recv(BUFFERSIZE)
    print(f"client data:{data}")
    client.close()
```
建议先调用shutdown再调用close，避免数据丢失
### listen真的是最大连接数吗？
启用服务器以接受连接。如果指定backlog，则必须至少为0（如果低于0，则设置为0）；它指定系统在拒绝新连接之前将允许的未接受连接的数量。如果未指定，则选择默认的合理值。
简单来说，这里的nt表示socket的”排队个数“, 但目前存在疑问？不同系统，是否是同一台主机貌似会影响这个结果
一般情况下，一个进程只有一个主线程（也就是单线程），那么socket允许的最大连接数为: n + 1如果服务器是多线程，比如上面的代码例子是开了2个线程，那么socket允许的最大连接数就是: n + 2换句话说：排队的人数(就是那个n) + 正在就餐的人数（服务器正在处理的socket连接数) = 允许接待的总人数（socket允许的最大连接数）
#### 服务端
```
import socket
import time
HOST = ""
PORT = 4001
BUFFERSIZE = 1024

def socket_demo_tcp():
    """
    socket.socket(family=-1, type=-1, proto=-1, fileno=None):
    """
    server_socket = socket.socket(family=socket.AF_INET, type=socket.SOCK_STREAM)
    server_socket.bind((HOST, PORT))
    server_socket.listen(1)
    while True:
        conns, address = server_socket.accept()
        print(f"conns:{conns}, address:{address}")
        msg = conns.recv(BUFFERSIZE)
        time.sleep(60)
        print(f"msg:{msg}")
        conns.send(b"reply")
```
#### 客户端
```
import socket

HOST = "192.168.XX.XXX"
HOST = "127.0.0.1"
PORT = 4001 
BUFFERSIZE = 1024

def client_tcp():
    client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    client.connect((HOST, PORT))
    client.send(b"hello")
    data = client.recv(BUFFERSIZE)
    print(f"client data:{data}")
```
前提：listen为1
Windows-->Windows： 当client进行第4个时会出现ConnectionRefusedError: [WinError 10061] 由于目标计算机积极拒绝，无法连接。
Linunx-->Linux: 目前发现client是100个也不会出现Windows类似的连接拒绝
Linux--> Windows、Windows--> Linux:与Linux-->Linux类似。不会出现连接拒绝的情况， 但如果开启线程的情况下，会出现连接失败情况
综上，listen在Windows上确实为排队连接的最大数量， 但在其他平台或者平台交互时表现却不是如此，望大佬指教
### socket.setsockopt(level, optname, value, optlen: int)详解
#### 相关参数
setsockopt()是对于socket的扩展，帮助设置socket的相关属性
level: 选项所在协议层
		SOL_SOCKET:通用套接字选项.
		IPPROTO_IP:IP选项.
		IPPROTO_TCP:TCP选项.
optname：具体的选项（socket中SO_*）
```
SO_DEBUG，打开或关闭调试信息。
SO_REUSEADDR，打开或关闭地址复用功能。
SO_DONTROUTE，打开或关闭路由查找功能。
SO_BROADCAST，允许或禁止发送广播数据。
SO_SNDBUF，设置发送缓冲区的大小。其上限为256 * (sizeof(struct sk_buff) + 256)，下限为2048字节。
SO_RCVBUF，设置接收缓冲区的大小。上下限分别是：256 * (sizeof(struct sk_buff) + 256)和256字节。
SO_KEEPALIVE，套接字保活。
SO_OOBINLINE，紧急数据放入普通数据流。
SO_NO_CHECK，打开或关闭校验和。
SO_PRIORITY，设置在套接字发送的所有包的协议定义优先权。
SO_LINGER，如果选择此选项, close或 shutdown将等到所有套接字里排队的消息成功发送或到达延迟时间后才会返回. 否则, 调用将立即返回。
SO_PASSCRED，允许或禁止SCM_CREDENTIALS 控制消息的接收。
SO_TIMESTAMP，打开或关闭数据报中的时间戳接收。
SO_RCVLOWAT，设置接收数据前的缓冲区内的最小字节数。
SO_RCVTIMEO，设置接收超时时间。
SO_SNDTIMEO，设置发送超时时间。
SO_BINDTODEVICE，将套接字绑定到一个特定的设备上。
SO_ATTACH_FILTER和SO_DETACH_FILTER。
```
value: 缓冲区大小
optlen: 

返回说明：
	成功执行时，返回0。失败返回-1，errno被设为以下的某个值
	EBADF：sock不是有效的文件描述词
	EFAULT：optval指向的内存并非有效的进程空间
	EINVAL：在调用setsockopt()时，optlen无效
	ENOPROTOOPT：指定的协议层不能识别选项
	ENOTSOCK：sock描述的不是套接字
#### 3.设置缓冲区（发送缓冲区，接收缓冲区）
当创建tcp和udp时默认会创建发送缓冲区和接收缓冲区
目前发现tcp服务端设置接收缓冲区大小会影响recv()中buffersize的作用，接收的数据由设置的接收缓冲区大小决定，其余则不会影响
tcp服务端
```
with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as server:
    server.setsockopt(socket.SOL_SOCKET, socket.SO_SNDBUF, 222)
　#设置接收缓冲区大小为７
    server.setsockopt(socket.SOL_SOCKET, socket.SO_RCVBUF, 7)
    server.bind((HOST, PORT))
    server.listen(4)
    while True:
        conn, addr = server.accept()
        print(f"conn:{conn}, addr:{addr}")
        data = conn.recv(BUFFERSIZE)
        # 接收到的数据则为缓冲区大小数据
        print(f"data:{data}")
        conn.send(data)
```
客户端
```
with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as client:
    client.setsockopt(socket.SOL_SOCKET, socket.SO_SNDBUF, 4)
    client.setsockopt(socket.SOL_SOCKET, socket.SO_RCVBUF, 4)
    # 设置发送的缓冲区大小
    client.connect((HOST, PORT))
    client.send(b"helloasdadddddddddddd")
    time.sleep(3)
    data = client.recv(BUFFERSIZE)
    print(f"data:{data}")
```
#### 用法
##### 1.并发连接时端口冲突的问题，是由于在socket连接正常close后，会有一定的等待时间端口仍被占用，因此可以移除关闭后的等待时间 SO_REUSEADDR:可以重用端口
```
setsockopt(SOL_SOCKET ,SO_REUSEADDR,1);
```
##### 2.设置收发时间，可能由于网络波动导致无法正常发送
```
setsockopt(SOL_SOCKET ,SO_SNDTIMEO,timeout);
setsockopt(SOL_SOCKET ,SO_RCVTIMEO,timeout);

import struct
with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as client:
    # 需要使用struct将python数据转换为字节流
    interval = struct.pack("ll", 10, 0)
    print(interval)
    client.setsockopt(socket.SOL_SOCKET, socket.SO_SNDTIMEO, interval)
    client.connect((HOST, PORT))
    client.send(b"hello")

```
##### 3.取消socket默认缓冲区，提高性能，默认是从socket缓冲区读到系统缓冲区，然后由系统缓冲区发送
```
setsockopt(socket.SOL_SOCKET, socket.SO_SNDBUF, 0)
setsockopt(socket.SOL_SOCKET, socket.SO_RCVBUF, 0)
```
##### 4.设置socket缓冲区，避免send/recv重复发送，send返回的是实际发送出去的数据(同步)或发送到socket缓冲区的数据(异步)，默认系统发送大小为8.8k，当发送数据量较大时则需要多次发送
```
setsockopt(socket.SOL_SOCKET, socket.SO_SNDBUF, buffersize)
setsockopt(socket.SOL_SOCKET, socket.SO_RCVBUF, buffersize)
```
#### 5.设置非阻塞方式
##### 直接在创建时设置
```
socket.socket(socket.AF_INET, socket.SOCK_STREAM | socket.SOCK_NONBLOCK, proto=socket.IPPROTO_TCP)
```
##### 创建完socket设置
```
server.setblocking(False)
```
##### 使用fcntl设置
```
# 获取当前文件描述符的状态
flag = fcntl.fcntl(server, fcntl.F_GETFL, 0)
# 设置为非阻塞
fcntl.fcntl(server, fcntl.F_SETFL, flag | socket.SOCK_NONBLOCK)
```
需要注意的是如果服务端在accept时设置非阻塞，由于当前没有客户端连接最好在accept时捕捉没有客户端的异常，防止流程中断
```
def tcp_non_block():
    try:
        with socket.socket(socket.AF_INET, socket.SOCK_STREAM | socket.SOCK_NONBLOCK, proto=socket.IPPROTO_TCP) as server:
            server.bind((HOST, PORT))
            # server.setblocking(False)
            server.listen(3)
            while True:
                try:
                    conn, addr = server.accept()
                    print(f"addr:{addr}")
                    conn.setblocking(False)
                    time.sleep(10)
                    data = conn.recv(BUFFERSIZE)
                    print(f"data:{data}")
                except Exception as e:
                    print(f"no client connect")
                    time.sleep(4)
    except Exception as e:
        print(f"tcp_non_block error:{e}")
```
### socket缓冲区
一个 Socket 会带有两个缓冲区，一个用于发送、另一个用于接收。因为这是个先进先出的结构，所以有时也叫它们发送、接收队列
对于send来说，发送成功不代表已经发送到对应接收端，只是发送到发送缓冲区，由操作系统决定什么时候发送
由于涉及到缓冲区，因此缓冲区的状态及socket的状态会影响到send/write或recv/read的状态，详细可见socket缓冲区文章
![image](https://img2022.cnblogs.com/blog/2850640/202204/2850640-20220429103232761-19348255.webp)
### socket相关流程图
#### socket位置、常见socket通信方式，socket缓冲区
![image](https://img2022.cnblogs.com/blog/2850640/202204/2850640-20220429103125732-751616889.png)
#### select IO多路复用
![image](https://img2022.cnblogs.com/blog/2850640/202204/2850640-20220429103449922-1468225405.png)

引用内容
[sockopt](https://www.cnblogs.com/clschao/articles/9588313.html "sockopt")
[sockopt_option](https://www.cnblogs.com/yuyutianxia/p/4908600.html "sockopt")
[socket缓冲区](https://www.cnblogs.com/traditional/p/11806454.html "socket缓冲区")
[shutdown与close区别](http://t.zoukankan.com/leijiangtao-p-11957234.html "shutdown与close区别")
[listen理解](https://www.jb51.net/article/209798.htm "listen理解")
以上内容均来自python官网:
[socket文档](https://docs.python.org/3/library/socket.html "socket文档")