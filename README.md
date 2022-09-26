# server-client-communication
simulating a small scale social network.
## assignment descreption
you will implement a simple social network server and
client. The communication between the server and the client(s) will be
performed using a binary communication protocol. A registered user will be
able to follow other users and post messages. Please read the entire
document before starting.
The implementation of the server will be based on the Thread-Per-Client
(TPC) and Reactor servers taught in class.
The servers, as seen in class, only support pull notifications.
Any time the server receives a message from a client it can replay back to
the client itself. But what if we want to send messages between clients, or
broadcast an announcement to a group of clients? We would like the server to
send those messages directly to the client without receiving a request to do
so. this behaviour is called push notifications.
The first part of the assignment will be to replace some of the current
interfaces with new interfaces that will allow such a case.
Note that this part changes the servers pattern and must not know the
specific protocol it is running. The current server pattern also works that way
(Generics and interfaces).
Once the server implementation has been extended you will have to
implement an example protocol. The BGS (Ben Gurion Social) Protocol will
emulate a simple social network.
Users need to register to the service. Once registered, they will be able to
post messages and follow other users. It is a binary protocol that uses pre
defined message length for different commands. The commands are defined
by an opcode, a short number at the start of each message. For each
command, a different length of data needs to be read according to it’s
specifications. In the following sections we will define the specifications of the
commands supported by the BGS protocol.
Unlike real social network you will not work with real databases. You will need
to save data (Users, Passwords, Messages, ect...). You only need to save
information from the time the server starts and keep it in memory until the
server closes.

REGISTER Messages:

Messages that appear only in a Client-to-Server communication.
A REGISTER message is used to register a user in the service. If the username is
already registerd in the server, an ERROR message is returned. If successful an
ACK message will be sent in return. Both string parameters are a sequence of
bytes in UTF-8 terminated by a zero byte (also known as the ‘\0’ char).
Parameters:
- Opcode: 1.
- Username: The username to register in the server.
- Password: The password for the current username (used to log in to the
server).
- Birthday: the birthday of the user (should be in the format DD-MM-YYYY)
Command initiation:
• This command is initiated by entering the following text in the client command
line interface: REGISTER <Username> <Password> <Birthday>

  LOGIN Messages:
Messages that appear only in a Client-to-Server communication.
A LOGIN message is used to login a user into the server. If the user doesn’t exist
or the password doesn’t match the one entered for the username, sends an
ERROR message. An ERROR message should also appear if the current client has
already succesfully logged in.
An ERROR message should appear also if the captcha byte is 0.
Both string parameters are a sequence of bytes in UTF-8 terminated by a zero
byte.
Parameters:
- Opcode: 2.
- Username: The username to log in the server.
- Password: The password for the current username (used to log in to the
server).
- Captcha: is to simulate captcha (must be 1, for successful login)
Command initiation:
• This command is initiated by entering the following text in the client command
line interface: LOGIN <Username> <Password> <Captcha>
  
LOGOUT Messages:
Messages that appear only in a Client-to-Server communication. Informs the
server on client disconnection. Client may terminate only after reciving ACK
message in replay. If no user is logged in, sends an ERROR message.
Parameters:
• Opcode: 3.
Command initiation:
• This command is initiated by entering the following text in the client command
line interface: LOGOUT
• Once the ACK command is received in the client, it must terminate itself.
  
FOLLOW Messages:
Messages that appear only in a Client-to-Server communication. A FOLLOW
message allows a user to add/remove other user to/from his follow list.
If the FOLLOW command failed an ERROR message will be sent back to the
client.
The user must be logged in, otherwise an ERROR message will be sent.
For a follow command to succeed, a user on the list must not already be on the
following list of the logged in user. (The opposite also applys for the unfollow
command).
The ack for this command will contain the user name of the followed/unfollowed
user.
The ACK message will have the following form:
ACK-Opcode FOLLOW-Opcode <username>
Parameters:
• Opcode: 4.
• Follow/Unfollow: This parameter has a value of 0 when a user wants to follow,
otherwise it has a value of 1(Unfollow).
• UserName: The requested user name to follow/unfollow.
Command initiation:
• This command is initiated by entering the following texts in the client
command line interface:
FOLLOW <0/1 (Follow/Unfollow)> <UserName>
  
POST Messages:
Messages that appear only in a Client-to-Server communication. A post message
allows a user to share messages with other users.
The Content parameter is a sequence of bytes in UTF-8.
All posts should be saved to a data structure in the server, along with PM
messages. A post message will be sent to users who are listed with a
“@username” inside the message (if username is registered in the system) and
to users following the user who posted the message.
In order to send a POST message the user must be logged in, otherwise an
ERROR message will be returned to the client.
Parameters:
• Opcode: 5.
• Content: The content of the message a user wants to post. The message may
contain @<username> in order to send it to specific users other then those
following the poster.
Command initiation:
• This command is initiated by entering the following texts in the client
command line interface:
POST <PostMsg>
  
PM Messages:
Messages that appear only in a Client-to-Server communication.
PM message is used to sent private messages to another user.
All string parameters are a sequence of bytes in UTF-8 terminated by a zero
byte.
In order to send a PM message the sending user must be logged in, otherwise an
ERROR message will be returned to the client.
When sending a message to the client add the Date to the content in the end of
the string
If the reciepient user isn’t registered an ERROR message will be returned to the
client.
@<username> isn’t applicable for private messages.
If the user is not following the reciepient user, ERROR message will be returned
to the client.
All PM messages should be saved to a data structure in the application, along
with post messages.
Before showing and saving the message, the server should filter the message
from words provided in the server, and every filtered word should be replaced
with ‘<filtered>’.
The data structure which includes the words need to be filtered, is hard coded in
the server.
For example: if the server has a list of words [‘war’, ‘Trump’], and the message
Which some client had send was: ‘Trump is planning to declare a war on the
republic of Lala-land’
Then the message that will be saved in the server is: ‘<filtered> is planning to
declare a <filtered> on the republic of Lala-land’
Parameters:
- Opcode: 6.
- UserName: The user to send the message to.
- Content: The content of the message the logged in user wants to send to the
other user.
- Sending date and time: the date and time when the message has sent
(Format: DD-MM-YYYY HH:MM)
Command initiation:
• This command is initiated by entering the following texts in the client
command line interface:
PM <Username> <Content>
  
LOGSTAT Message:
Messages that appear only in a Client-to-Server communication.
A LOGSTAT message is used to recieve data on a logged in users (the age of
every user , number of posts every user posted, number of every user’s
followers, number of users the user is following).
In order to send a LOGSTAT message the user must be logged in, otherwise an
ERROR message will be returned to the client.
If user is not registered, an error message will be returned.
The returned ACK message will contain (for every single username) user’s age,
number of posts a user posted (not including PM’s), number of followers, number
of users the user is following in the optional section of the ACK message.
Clarifications: if user a blocks user b → then user a will not get information about
user b when sending a logstat MSG and user b will not get information about
user a
Example:
ACK-Opcode LOGSTAT-Opcode <Age><NumPosts> <NumFollowers>
<NumFollowing>
ACK-Opcode LOGSTAT-Opcode <Age><NumPosts> <NumFollowers>
<NumFollowing>
Parameters:
• Opcode: 7.
Command initiation:
• This command is initiated by entering the following texts in the client
command line interface: LOGSTAT
  
STAT Messages
A STAT message is used to recieve data on a certain users (the age of every user
, number of posts every user posted, number of every user’s followers, number
of users the user is following).
The ‘List of usernames’ parameter are a sequence of bytes in UTF-8 terminated
by a zero byte,
With the following format ‘ usernam1|username2|username3|...’ for
simplicity, you can assume that ‘username’ doesn’t contain the symbol
‘|’.
Messages that appear only in a Client-to-Server communication.
In order to send a STAT message the user must be logged in, otherwise an
ERROR message will be returned to the client.
If user is not registered, an error message will be returned.
The returned ACK message will contain (for every single username) user’s age,
number of posts a user posted (not including PM’s), number of followers, number
of users the user is following in the optional section of the ACK message.
Clarifications: if user a blocks user b → and user b is in the user list then user a
will get an error MSG
if user b blocks user a → and user b is in the user list then user a will get an error
MSG
Example:
ACK-Opcode STAT-Opcode <Age><NumPosts> <NumFollowers>
<NumFollowing>
ACK-Opcode STAT-Opcode <Age><NumPosts> <NumFollowers>
<NumFollowing>
Parameters:
• Opcode: 8.
• List of usernames: The list of users whose stats will be returned to the client.
Command initiation:
• This command is initiated by entering the following texts in the client
command line interface: STAT <UserNames_list>
  
NOTIFICATION Messages:
Messages that appear only in a Server-to-Client communication. This message
will be sent from the server for any PM sent to the user, post sent by someone
the user is following, or a post that contained @<MyUsername> in the content of
a message.
Both string parameters are a sequence of bytes in UTF-8 terminated by a zero
byte.
A user will recive any POST/PM notification sent after follow (for users the current
user is following) that he didn’t see. I.e. wasn’t logged in when the other user
posted/sent the message. (Clue: for each user, save timestamp of last message
recieved from each of the other users / timestamp of the follow command)
Parameters:
• Opcode: 9.
• NotificationType : indicates whether the message is a PM message (0) or a
public message (post) (1).
• PostingUser: The user who poster/sent the message.
• Content: The message that was posted/sent.
Client screen output:
• Any NOTIFICATION message received in client sould be written to the screen:
NOTIFICATION <”PM”/”Public”> <PostingUser> <Content>
  
ACK Messages
Messages that appear only in a Server-to-Client communication.
ACK Messages are used to acknowledge different Messages. Each ACK contains
the message number for which the ack was sent. In the optional section there
will be additional data for some of the Messages (if a message uses the optional
section it will be specified under the message description).
All Messages that appear in a Client-to-Server communication require an
ack/error message in response.
Parameters:
• Opcode: 10.
• Message Opcode: The message opcode the ACK was sent for.
• Optional: changes for each message.
Client screen output:
• Any ACK message received in client sould be written to the screen:
ACK <Message Opcode> <Optional>
• Printing of the optional part:
- Multi-parameter optional sections should be split by space and printed by
order of the ack response
- Short should be printed as numbers
- String lists should be seperated by a space
  
ERROR Messages
Messages have the following format:
Messages that appear only in a Server-to-Client communication.
An ERROR message may be the acknowledgment of any other type of message.
In case of error, an error message should be sent.
Parameters:
• Opcode: 11.
• Message Opcode: The message opcode the ERROR was sent for.
Error Notification:
• Any error message received in client should be written to screen:
ERROR <Message Opcode>
  
BLOCK Message
Messages that appear only in a Client-to-Server communication.
Message send to block a specific user.
Blocked user can’t follow, PM, or show any information about the user who
blocked him.
Once user blocking acknowledged, both users (the blocking and the blocked)
stop following each other.
If ‘username’ doesn’t exist, and ERROR message will be sent back.
Clarifications: if user a blocks user b → then user a will not get information about
user b when sending a logstat MSG and user b will not get information about
user a
Parameters:
- Opcode: 12
- Username: the username to block
Command initiation:
- This command is initiated by entering the following texts in the client
command line interface: BLOCK <UserNames>
