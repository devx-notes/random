# Port

A port in networking is a software-defined number associated to a network protocol that receives or transmits communication for a specific service

- It is a way to multiplex IP addresses so that different applications can listen on the same IP address/protocol pair
- Well-known ports (the lowest numbered 1024 port numbers) are used for the most commonly used services. Higher-numbered ports are available for general use.

## 1. Socket

Both Socket and Port are the terms used in Transport Layer. A socket is an endpoint for data transfer between the devices, it is identified by  $\{src-ip, src-port\}$, $\{dest-ip, dest-port\}$ and $protocol$

- On startup, the server creates a listening socket bound to a specific port.
- When a client connects to a server's listening port, the server creates a new socket for that specific connection.
- Even though multiple connections share the same server port, they're distinguished by having different client IP/port combinations.

### TCP Socket

```c
int sockfd;
struct sockaddr_in servaddr;

sockfd = socket(AF_INET, SOCK_STREAM, 0); 
servaddr.sin_family = AF_INET; 
servaddr.sin_addr.s_addr = htonl(INADDR_ANY); 
servaddr.sin_port = htons(PORT); 

bind(sockfd, (SA*)&servaddr, sizeof(servaddr));
```

## 2. File Descriptor 

In Unix and Unix-like computer operating systems, a file descriptor (`fd`, less frequently `fildes`) is a process-unique identifier (handle) for a file or other input/output resource, such as a pipe or network socket.

## 3. References

- [OSI Model](https://en.wikipedia.org/wiki/OSI_model#Programming_interfaces)
- [How do Multiple TCP Clients Connect Simultaneously to a Single Port on a Server?](https://www.pico.net/kb/how-do-multiple-tcp-clients-connect-simultaneously-to-a-single-port-on-a-server/#:~:text=TCP%20connections%20are%20identified%20by,ports%20%2D%20making%20each%20connection%20unique.)