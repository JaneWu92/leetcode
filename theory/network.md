### ConnectionTimeout vs ReadTimeout
ConnectionTimeout: time that wait for TCP connection set up.  
ReadTimeout: time that wait for read from socket. It can be:
1. time for network transfer
2. time for server to handle business  

Something to know about readtimeout:  
1. When client readtimeout, doesn't mean server will stop that service. It will continue.
2. If we configure readtimeout for too big, it may cause too many threads pending in clients and will make its load too heavy.

