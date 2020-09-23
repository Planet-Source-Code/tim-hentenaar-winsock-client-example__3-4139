<div align="center">

## Winsock Client Example


</div>

### Description

This is a simple Client to go along with my Winsock Server Example.
 
### More Info
 
It takes 2 arguments:

1) server to connect to

2) port


<span>             |<span>
---                |---
**Submitted On**   |
**By**             |[Tim Hentenaar](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByAuthor/tim-hentenaar.md)
**Level**          |Beginner
**User Rating**    |5.0 (10 globes from 2 users)
**Compatibility**  |C, C\+\+ \(general\), Microsoft Visual C\+\+
**Category**       |[Internet/ Browsers/ HTML](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByCategory/internet-browsers-html__3-9.md)
**World**          |[C / C\+\+](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByWorld/c-c.md)
**Archive File**   |[](https://github.com/Planet-Source-Code/tim-hentenaar-winsock-client-example__3-4139/archive/master.zip)

### API Declarations

```
// Programming with Sockets: Win32 Client Example
// (C) 2002 Matriark TerVel
// http://opengraal.com
// This example is free software distributed under
// the GNU General Public Licence (http://www.gnu.org/licenses/gpl.txt)
#include <windows.h>
#include <winsock.h> // Windows Sockets
#include <stdio.h> // printf() and standard I/O
#include <stdlib.h> // Standard C/C++ Functions
#include <sys/time.h> // timeval
#include <signal.h> // signals
```


### Source Code

```
// Programming with Sockets: Win32 Client Example
// (C) 2002 Matriark TerVel
// http://opengraal.com
// This example is free software distributed under
// the GNU General Public Licence (http://www.gnu.org/licenses/gpl.txt)
#include <windows.h>
#include <winsock.h> // Windows Sockets
#include <stdio.h> // printf() and standard I/O
#include <stdlib.h> // Standard C/C++ Functions
#include <sys/time.h> // timeval
#include <signal.h> // signals
// Define signals for error trapping
#define SIGHUP 1	// Hang-Up
#define SIGINT 2	// User pressed CTRL+C
#define SIGQUIT 3	// Quit Process
#define SIGKILL 9	// Kill signal (End task sends this)
#define SIGSEGV 11	// Seg Fault (crash)
#define SIGPIPE 13
unsigned int sock;	// Client socket
void sighandle(int signum) { // Our familliar signal handler
	if (sock) closesocket(sock); // Close socket
	WSACleanup();	// Clean up after winsock :P
	printf("Exiting due to signal(%d)\n",signum);
	exit(0);
	// Exit normally ("man 3 exit" displays this man page in linux)
}
int main(int argc, char **argv) {
	// Declare the signals to trap
	// This is used to make sure the socket isn't left open in the event
	// of a crash, kill, or user exit
	signal(SIGHUP, SIG_IGN); // Ignore
	signal(SIGPIPE, SIG_IGN);
	signal(SIGQUIT, sighandle); // send to sighandle() (defined above)
	signal(SIGINT, sighandle);
	signal(SIGTERM, sighandle);
	signal(SIGSEGV, sighandle);
	signal(SIGKILL, sighandle);
	// The main portion of our program starts here
	WSADATA winsock_data; // Winsock Info
	WSAStartup(MAKEWORD(1,1), &winsock_data); // Start Winsock
	/* NOTE:
	 * the &winsock_data passes the address of winsock_data instead of
	 * winsock_data itself. (i.e. a pointer to winsock_data)
	 *
	 * MAKEWORD(major,minor) makes a WORD (basically a long) out of
	 * major and minor. in this case it returns 1.1
	 */
	// Print out Winsock Data
	printf("Matriark TerVel's Example Winsock Client\n\n");
	printf("\tWinsock Info\n\tVersion: %d.%d\n\tDescription: %s\n\tStatus: %s\n\n",winsock_data.wVersion,winsock_data.wHighVersion,winsock_data.szDescription,winsock_data.szSystemStatus);
	if (argc<1) {
	 printf("Too Few Arguments.. Exiting..\n\n");
	 WSACleanup();
	 exit(0);
	}
	// Create the socket
	printf("Creating Socket..\n");
	sock = socket(AF_INET, SOCK_STREAM, IPPROTO_TCP);
	// Make sure socket is OK
	if (sock == INVALID_SOCKET || sock == SOCKET_ERROR) {
		printf("Invalid Socket. Winsock Error: %d\n",WSAGetLastError());
		if (sock) closesocket(sock);
		WSACleanup();
		return 0;
	} else printf("Socket Established!\n");
	// Resolve the server to connect to
	sockaddr_in addr; // for connecting to the server
	// Argument 1 = remote server
	// Argument 2 = port
	char *server = argv[1]; // server name
	int port = atoi(argv[2]); // port (ascii-to-integer)
	addr.sin_family = AF_INET; // An Internet Socket
	addr.sin_port = htons(port); // port must be in network order
	// Resolve the server
	hostent *r_host;
	r_host = gethostbyname(server); // Resolve by hostname
	if (r_host == NULL) {
		printf("Unable to resolve host: %c!\n\n",server);
		if (sock) closesocket(sock);
		WSACleanup();
	}
	// Assign IP Address to addr
	memcpy(&addr.sin_addr.s_addr, r_host->h_addr, r_host->h_length);
//	addr.sin_addr.s_addr = r_host->h_addr; // Server's IP
/* NOTE: this is useful in GUIs but not needed in this example
	// Don't block the program waiting on the socket
	// (e.g. make it a "non-blocking" socket)
	// 0 = on (block), 1 = off (don't block)
	unsigned long block = 1;
	ioctlsocket(sock, FIONBIO,&block);
*/
	// Connect to server
	connect(sock,(struct sockaddr *)&addr,sizeof(addr));
	printf("Connected to Server!\n");
	struct timeval tm;
	tm.tv_sec = 0;
	tm.tv_usec = 0;
	fd_set *set;
	FD_ZERO(set);
	int bytesread;
	char xbuffer[8193];	// for recv()
	char data[8193]; 	// for send()
	memset(data,'\0',sizeof(data)); // clear buffer
	while (true) { // Endless Loop
/*NOTE: select() is used to get the status of a socket set.
 * int select(unsigned int s, fd_set *read, fd_set *write, fd_set *except,
 *
 */
		// we use the read set because we want to see if there is data
		// waiting to be read
		if (select(sock,set,NULL,NULL,&tm) == SOCKET_ERROR) {
			// If select() had problems
			printf("select() returned SOCKET_ERROR. aborting.\n");
			closesocket(sock);
			WSACleanup();
			exit(0); // exit
		} else { // No problems
		 if (FD_ISSET(sock,set)) { // If our socket has data
			bytesread = recv(sock,xbuffer,8192,0); // read data
			if (bytesread == SOCKET_ERROR) {
			 printf("read() returned SOCKET_ERROR! Winsock Error: %d\n\n",WSAGetLastError());
			 if (sock) closesocket(sock);
			 WSACleanup();
			 exit(0);
			}
			// print the data to STDOUT (the console)
			printf("Recieved %d bytes: %s\n",bytesread,xbuffer);
			// Clear the buffer
			memset(xbuffer,'\0',sizeof(xbuffer));
		 }
		}
		printf("Enter Data to Write to Server: ");
		scanf("%s",data); // get data from STDIN (Console)
		send(sock,data,sizeof(data),0); // Send data to server
		memset(data,'\0',sizeof(data));
	} // end the endless loop :P
	if(sock) closesocket(sock);
	WSACleanup();
	return 0;
}
```

