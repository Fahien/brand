---
toc: true
toc_sticky: true
---
This devlog introduces the fundamental functions of the [Berkeley socket API](https://en.wikipedia.org/wiki/Berkeley_sockets) by developing a simple client and server with the C language. The complete source code is available on this [github repository](https://github.com/Fahien/exsocket).

## Client

The first thing to do is to create a connection-oriented (`SOCK_STREAM`) socket of the IPv6 Internet protocol family (`AF_INET6`). This is going to use [TCP](https://en.wikipedia.org/wiki/Transmission_Control_Protocol), a reliable transport protocol.

```c
int sockfd;
if ((sockfd = socket(AF_INET6, SOCK_STREAM, 0)) < 0) {
    perror("socket");
    return -1;
}
```

The socket can be used to establish a connection with a server, and we can specify the address to connect to with a `sockaddr_in6` structure.

```c
struct sockaddr_in6 servaddr;
bzero(&servaddr, sizeof(servaddr));
servaddr.sin6_family = AF_INET6;
```

Both the port and the address data follow the host byte-ordering, which is not guaranteed to be the same as the ordering used by the network protocol. Accordingly, we need to convert them to a network representation and that can be done by making use of `htons()` for the integer port, and `inet_pton()` for the IPv6 address string.

```c
servaddr.sin6_port = htons(atoi(argv[2]));

if (inet_pton(AF_INET6, argv[1], &servaddr.sin6_addr) <= 0) {
    perror("inet_pton");
    return -1;
}
```

We can then pass the socket and the sockaddr structure to `connect()` which is going to reach out to the server.

```c
if (connect(sockfd, (struct sockaddr*) &servaddr, sizeof(servaddr)) < 0) {
    perror("connect");
    return -1;
}
```

Once the connection is established, we can receive data from the server by reading the socket. The bytes read will be stored into the `recvline` buffer, then a `0` is added at the end to finalize it as a string, and `fputs()` is used to print it to `stdout`.

```c
int n;
char recvline[MAXLINE + 1];
while ((n = read(sockfd, recvline, MAXLINE)) > 0) {
    recvline[n] = 0;
    fputs(recvline, stdout);
}
if (n < 0) {
    perror("read");
    return -1;
}
```

When the communication has finished, we need to close the socket, like every file descriptor.

```c
close(sockfd);
```

## Server

As well as the client, the first thing the server does is to create an IPv6 socket.

```c
int listenfd;
if ((listenfd = socket(AF_INET6, SOCK_STREAM, 0)) < 0) {
    perror("socket");
    return -1;
}
```

Then a `sockaddr` structure is initialized using `in6addr_any` which is a wildcard IPv6 address, letting the system to select the address for you.

```c
struct sockaddr_in6 servaddr;
bzero(&servaddr, sizeof(servaddr));
servaddr.sin6_family = AF_INET6;
servaddr.sin6_addr = in6addr_any;
servaddr.sin6_port = htons(atoi(argv[1]));
```

Before continuing we call `bind()` to assign this address to the socket.

```c
if ((bind(listenfd, (struct sockaddr*) &servaddr, sizeof(servaddr))) < 0) {
    perror("bind");
    return -1;
}
```

Then we convert the socket to a listening one, so that it could be used to accept incoming connection.

```c
if (listen(listenfd, BACKLOG) < 0) {
    perror("listen");
    return -1;
}
```

Here we can finally make server to wait for client requests. The `accept()` function is blocking and returns when a new incoming connection has been established. In order to generate a response from the server we get the date and time with the `time()` function, then we build the string to send to the client, and finally we write it onto the socket and close the connection once finished.

```c
int connfd;
int n;
time_t ticks;
char buff[MAXLINE];
while (true) {
    if ((connfd = accept(listenfd, (struct sockaddr*) NULL, NULL)) < 0) {
        perror("accept");
        return 1;
    }

    // Send the time to the client
    ticks = time(NULL);
    snprintf(buff, sizeof(buff), "%.24s\r\n", ctime(&ticks));
    while ((n = write(connfd, buff, strlen(buff))) < 0);

    // Close the connection
    close(connfd);
}
```

## Get client information

The first thing we can do to improve the server is to make a better use of the `accept()` function. By passing as an argument a `sockaddr` structure address, we could retrieve some information about the client.

```c
struct sockaddr_in6 cliaddr;
socklen_t clilen = sizeof(cliaddr);
bzero(&cliaddr, clilen);
accept(listenfd, (struct sockaddr *) &cliaddr, &clilen);
```

Please note that the data memory layout is still following the network order, and if we want to have meaningful address and port values on our system, we need to convert them into the appropriate _host representation_.

```c
inet_ntop(AF_INET6, &cliaddr.sin6_addr, buff, INET6_ADDRSTRLEN);
printf("Serving new client from %s:%d\n", buff, ntohs(cliaddr.sin6_port));
```

## Get local address

We have seen how the server can retrieve the address of a client. Let us see, on the other hand, how the host can query its own local address through `getsockname()`.

```c
struct sockaddr_in6 cliaddr;
socklen_t clilen = sizeof(cliaddr);
bzero(&cliaddr, clilen);
if (getsockname(sockfd, (struct sockaddr *) &cliaddr, &clilen) < 0) {
    perror("getsockname");
    return -1;
}
```

Now we have the information we need into `cliaddr`, the we convert the IPv6 address and port into the appropriate _host representation_ and finally we print it out.

```c
char clistr[INET6_ADDRSTRLEN];
inet_ntop(AF_INET6, &cliaddr.sin6_addr, clistr, INET6_ADDRSTRLEN);
printf("The current address is %s:%d\n", clistr, ntohs(cliaddr.sin6_port));
```

## Sending an acknowledgement

We would like the client to respond to the server with an acknowledgement message, and this can be done after the reading step by calling `write()`.

```c
char* ack = "Message received\n";
if ((n = write(sockfd, ack, strlen(ack))) < 0) {
    perror("write");
    return -1;
}
```

On the other side of the connection, the server should invoke `read()` to be able to receive the data sent by the client.

```c
char recvline[MAXLINE + 1];
if ((n = read(connfd, recvline, MAXLINE)) > 0) {
    recvline[n] = 0;
    fputs(recvline, stdout);
}
if (n < 0) {
    perror("read");
    return -1;
}
```

As you may notice, both `read()` and `write()` return the number of characters written into the socket buffer. In case of an error the return value is negative (`-1`) and the `errno` global variable is set appropriately. This means we can retrieve an informative message corresponding to the current value of `errno` through the `perror()` function.

## Recursive echo server

Let us see now two new functions useful for our network programs, `exso_readln()` and `exso_writen()`, based on some utilities functions presented by the [Unix Network Programming](https://www.informit.com/articles/article.aspx?p=169505&seqNum=9) book and used in a [networking course](https://sites.google.com/a/unisa.it/forciuoli-insegnamenti/home/reti-di-calcolatori) I attended at university.

- The `exso_readln()` function reads a line (a string ending with the newline character `'\n'`) from a socket buffer and automatically appends `'\0'` to mark the end of that string.
- The `exso_writen()` function writes `n` bytes into a socket buffer.

> More details can be found here: [libexso.c](https://github.com/Fahien/exsocket/blob/master/libexso.c).

The daytime server discussed so far features an _iterative_ behaviour, which means it can serve only one client at a time. In order to serve more clients concurrently, we could make it _recursive_ by spawning _child_ processes to manage new incoming connection. That is how it can be done with `fork()`.

```c
pit_t childpid;
if ((childpid = fork()) == 0) {
    close(listenfd);
    server_echo(connfd);
    return 0;
}
```

The child process must close the listening socket before executing its tasks. At the same time, the _father_ closes the connected socket and returns waiting for new requests. Here follows the `server_echo()` function which executes all the tasks of a server child process, also using the new read and write functions mentioned above.

```c
void server_echo(int sockfd) {
    ssize_t n;
    char line[MAXLINE];
    while (true) {
        if ((n = exso_readln(sockfd, line, MAXLINE)) == 0) {
            return; // connection closed by the other end
        }
        exso_writen(sockfd, line, n);
    }
}
```

## Kill the zombies

Good, now the server can handle multiple concurrent connections by spawning child processes. However, we should not forget to handle these processes when they end serving clients otherwise they will remain in memory as _zombies_.

When a child process terminates, the operating system sends a `SIGCHLD` signal to the father which by default does not do anything. If we want to remove zombies (dead child processes), the father can do that by registering a callback function for the `SIGCHLD` signal.

```c
if (signal(SIGCHLD, handle_zombies) < 0) {
    perror("signal");
    return -1;
}
```

Here is how the callback function works. By calling the `waitpid()` function with `-1` as first argument, the father process waits for the termination of any of its children. The `stat` output variable will contain information about a child's exit status.

```c
void handle_zombies(int signo) {
    pid_t pid;
    int stat;

    while ((pid = waitpid(-1, &stat, WNOHANG)) > 0) {
        printf("Child %d terminated with exit status %d\n", pid, stat);
    }
}
```

## UDP echo client

So far, we have been using TCP as the transport protocol. If we want to use the [UDP](https://en.wikipedia.org/wiki/User_Datagram_Protocol), we need to make some minor changes.

Change the socket type to `SOCK_DGRAM`.

```c
int sockfd;
if ((sockfd = socket(AF_INET6, SOCK_DGRAM, 0)) < 0) {
    perror("socket");
    return -1;
}
```

Data transmissions should be done with the `sendto()` function.

```c
char sendline[MAXLINE];
if (fgets(sendline, MAXLINE, fp) != NULL) {
    if (sendto(sockfd, sendline, strlen(sendline), 0, servaddr, sizeof(servaddr) < 0) {
        perror("sendto");
        return -1;
    }
}
```

Incoming data is collected with the `recvfrom()` function. Notice how, working with a connection-less protocol, the address of the sender is collected otherwise we would not know who sent these packets.

```c
struct sockaddr *replyaddr;
replyaddr = malloc(sizeof(servaddr));
len = sizeof(servaddr);
char recvline[MAXLINE + 1];

if ((n = recvfrom(sockfd, recvline, MAXLINE, 0, replyaddr, &len)) < 0) {
    perror("recvfrom");
    return -1;
}

recvline[n] = 0;
```

Consequently, we should verify the identity of the peer. Here we just make sure all the bytes of the `sockaddr` structures are equal.

```c
char buff[INET6_ADDRSTRLEN];
struct sockaddr_in6 *sa;

if ((len != sizeof(servaddr)) || memcmp(&servaddr, replyaddr, len) != 0) {
   sa = (struct sockaddr_in6 *) replyaddr;
   inet_ntop(AF_INET6, &sa->sin6_addr, buff, INET6_ADDRSTRLEN);
   printf("Ignoring reply from %s\n", buff);
   continue;
}
```

## Distributed calculator

Let us put our hands on something a bit more complex: a **distributed calculator**.

The communication takes place via UDP, so we use `sendto()` and `recvfrom()` introduced in the previous section.

```c
char buff[MAXLINE];
int a = 0;
int b = 0;
char op[OP_LEN] = "+";
while (fgets(buff, MAXLINE, stdin) != NULL) {
    if (sscanf(buff, "%d %s %d", &a, op, &b) == 3) {
        // ... do networking
    }
}
```

On the other end, the server receives the data, interprets it, computes the results and sends it back to the client.


```c
char buff[MAXLINE];
socklen_t len;

int a;
int b;
char op[OP_LEN];

while (true) {
    // Receive datagram
    len = clilen;
    if (recvfrom(sockfd, buff, MAXLINE, 0, cliaddr, &len) < 0) {
        perror("recvfrom");
        return -1;
    }
    
    // Interpret data and compute the result
    if (sscanf(buff, "%d %s %d", &a, op, &b) == 3) {
        if (strcmp(op, "+") == 0) {
            sprintf(buff, "%d", a + b);
        } else if (strcmp(op, "-") == 0) {
            sprintf(buff, "%d", a - b);
        } else if (strcmp(op, "*") == 0) {
            sprintf(buff, "%d", a * b);
        } else if (strcmp(op, "/") == 0) {
            if (b != 0) {
                sprintf(buff, "%d", a / b);
            } else {
                sprintf(buff, "Undefined");
            }
        } else if (strcmp(op, "mod") == 0) {
            sprintf(buff, "%d", a % b);
        } else {
            sprintf(buff, "Invalid operator");
        }
    } else {
        sprintf(buff, "Invalid message");
    }
    
    // Send result to the client
    if (sendto(sockfd, buff, strlen(buff), 0, cliaddr, len) < 0) {
        perror("sendto");
        return -1;
    }
}
```

## I/O multiplexing using select

The simple [echo client](https://github.com/Fahien/exsocket/blob/6c2f438cabaa80fbb6f4617ba9986f0bffafd732/echo/echocli.c) described previously suffers of some robustness problems. For example, if the server closes the connection while the client is blocked in a request for user input, the client would not be aware of anything until it tries to send data again after the user input has been read. I/O Multiplexing can help us solve this problem.

By means of the `select()` function we can work with more than one descriptor, and that requires preparing an `fd_set` object. In the example below, we reset `readset` with `FD_ZERO()` and then we add our file descriptors through `FD_SET()`.

```c
FILE *fp;
int sockfd;
// ...
fd_set readset;
FD_ZERO(&readset);
FD_SET(fileno(fp), &readset);
FD_SET(sockfd, &readset);
int maxfd = MAX(fileno(fp), sockfd) + 1;
if (select(maxfd, &readset, NULL, NULL, NULL) < 0) {
    perror("select");
    return -1;
}
```

When the `select()` returns, we can check whether the descriptors are ready by making use of `FD_ISSET()`.

```c
// Check if socket is ready to receive data from the server
if (FD_ISSET(sockfd, &readset)) {
    if ((n = exso_readln(sockfd, recvline, MAXLINE)) < 0) {
        perror("exso_readln");
        return -1;
    }
    if (n == 0) {
        printf("Server terminated prematurely\n");
        return -1;
    }
    fputs(recvline, stdout);
}

// Check if file is ready to send its content to the server
if (FD_ISSET(fileno(fp), &readset)) {
    if (fgets(sendline, MAXLINE, fp) == NULL) {
        return 0;
    }
    if (exso_writen(sockfd, sendline, strlen(sendline)) < 0) {
        perror("exso_writen");
        return -1;
    }
}
```

## Half closing the socket

Another way to improve the robustness of the client code is by shutting down the _write half_ of the connection once all reading operations from _standard input_ have finished. This can be done by calling the `shutdown()` function.

```c
char stdin_eof = 0;
// ...
if (FD_ISSET(fileno(fp), &readset)) {
    if (fgets(sendline, MAXLINE, fp) == NULL) {
        stdin_eof = 1;
        shutdown(sockfd, SHUT_WR);
        FD_CLR(fileno(fp), &readset);
        continue;
    }
    if (exso_writen(sockfd, sendline, strlen(sendline)) < 0) {
        perror("exso_writen");
        return -1;
    }
}
```

## Improved echo server

A recursive server is not the best choice if we want to avoid a proliferation of child processes, given that we fork every time a new client is connected. The server, as well as the client, could benefit of I/O Multiplexing via `select()` to handle both the listening socket and the other connection sockets.

```c
if ((ready = select(maxfd + 1, &rset, NULL, NULL, NULL)) < 0) {
    perror("select");
    return -1;
}

// Check for incoming connections
if (FD_ISSET(listenfd, &rset)) {
    if ((connfd = accept(listenfd, (struct sockaddr *) &cliaddr, &clilen)) < 0) {
        perror("accept");
        return -1;
    }
    // Save connected socket descriptor into an array
    for (i = 0; i < FD_SETSIZE; i++) {
        if (clients[i] < 0) {
            clients[i] = connfd;
            break;
        }
    }
    // ...
}

// Serve clients
for (i = 0; i < FD_SETSIZE; i++) {
    if ((sockfd = clients[i]) < 0) {
        continue;
    }
    if (FD_ISSET(sockfd, &rset)) {
        if ((n = exso_readln(sockfd, buff, MAXLINE)) < 0) {
            perror("exso_readln");
        }
        else if (n == 0) {
            printf("A client is gone\n");
            close(sockfd);
            FD_CLR(sockfd, &allset);
            clients[i] = -1;
        } else if (exso_writen(sockfd, buff, n) < 0) {
            perror("exso_writen");
        }
        // ...
    }
}
```
## Broadcast

What we have now is an array of all the connected clients, and that enable us to turn the server into a _chat room_! Here the server broadcasts every message sent by a client to all the other connected clients:

```c
for (i = 0; i < FD_SETSIZE; i++) {
    if (sockfd == client[i] || client[i] == -1) {
        continue;
    }
    if (exso_writen(client[i], buff, n) < 0) {
        perror("exso_writen");
    }
}
```

## Nickname command

After transforming the server into a chat room, it would definitely be a good idea to allow clients to set their nickname with a command. So, when a client is connected, the server is expecting to receive a message starting with `/nickname` followed by the actual nickname.

```c
#define MAXNICK 32
// ...
char nicknames[FD_SETSIZE][MAXNICK];
// Processing the i-th client
if (nicknames[i][0] == 0) {
    if (strncmp(buff, "/nickname", 9) == 0) {
        sscanf(buff, "/nickname %s\n", nicknames[i]);
        sprintf(buff, "%s has joined the chat\n", nicknames[i]);
    } else {
        sprintf(buff, "Error, expecting /nickname\n");
        exso_writen(client[i], buff, strlen(buff));
        continue;
    }
}
```

Once the nickname is set, we can use it as a prefix for the message and broadcast it to everyone.

```c
char temp[MAXLINE];
sprintf(temp, "%s: %s", nicknames[i], buff);
sprintf(buff, "%s", temp);
// ...
for (i = 0; i < FD_SETSIZE; i++) {
    if (sockfd == client[i] || client[i] == -1) {
        continue;
    }
    if (exso_writen(client[i], buff, strlen(buff)) < 0) {
        perror("exso_writen");
    }
}
```

To conclude, when a client logs out, we can reset its nickname.

```c
// If there are no more characters to read
if (n == 0) {
    close(sockfd);
    FD_CLR(sockfd, &allset);
    client[i] = -1;
    bzero(nicknames[i], MAXNICK);
}
```
