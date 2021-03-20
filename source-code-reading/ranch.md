https://github.com/ninenines/ranch

Core concepts
- listener: is an entity whose role is listen on a given port and handle new connections. Each listener will specify its own handler to handle incoming connection request
- protocols: protocol is how you handle incoming connection. After a connection is establish, the specified protocol will be dispatch. A protocol can be in the form of MFA (module, function, arity)

Several modules of ranch
- ranch_listener_sup: a supervisor for ranch listener process. Có nhiệm vụ start ranch listener được chỉ định lên
- ranch_server: a gen server holding miscellaneous info like opts, pid, ets table. Other processes can fetch info from this process
- ranch_conns_sup_sup: supervisor of ranch_conns_sup. Only one for each listener started
- ranch_acceptors_sup: this process will start one acceptor and try to listen on the configured port
- ranch_acceptor: this process will represent a acceptor. It loops to accept from the socket. Each ranch_acceptor will be monitor by one ranch_conns_sup. After a socket has been establish by acceptor, the acceptor will set the ranch_conns_sup as the pid receive the messages from socket.
- ranch_conns_sup: represent a connection. This process will be start and enter a receive loop (why not use a gen server though). This process will receive messages from socket. After a connection has been created, this process will start the protocol we specify in listener. After it starts the protocol successful, it will handover the messages from socket to the protocol pid.
- ranch_tcp: implement of transport layer. Doing stuff like listen or perform handshake.

Other details:
- Each acceptor can only serve on connection at a time. When an acceptor establish successfully a connection, the acceptor then tell the ranch_conn_sup to start the configured protocol. If the max connections has not yet reach, the ranch_conns_sup will allow the acceptor the resume immediately, ready to accept another connection. But if the max_connections is reach, the acceptor will be put in a list of sleepers. After a protocol is done running, the ranch_conns_sup will check if there any sleeper left in the queue. If it has then that sleeper will be allow to resume its execution, ready to accept another connection.

Question:
- What exactly does `handshake` do?
- Why a ranch_acceptor has to be monitor by a ranch_conns_sup?