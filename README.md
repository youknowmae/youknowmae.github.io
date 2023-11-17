# Creating A Simple Real-Time Chat-App Using Python Sockets and Tkinter

***Sockets*** play a vital role in the domain of network programming, functioning as a flexible and effective method of communication among processes, devices, or applications across a network. In this context, a socket serves as a fundamental endpoint for the transmission and reception of data over a computer network. 

It presents a potent and versatile system for networking, granting developers the ability to construct a variety of networked applications seamlessly and efficiently. Whether the project entails crafting a straightforward chat application or a sophisticated distributed system, Python sockets act as the foundational framework for facilitating seamless communication in the digital landscape.

while ***Tkinter*** is widely recognized as the primary GUI ***(Graphical User Interface)*** toolkit for Python, offering developers a clear and potent method for crafting desktop applications that incorporate graphical interfaces. it is constructed based on the Tk GUI toolkit, which has its origins as an integral component of the Tcl (Tool Command Language) scripting language. 

In this blog you will be able to understand the process of how communication works in sockets, as well as the benefit of using Tkinter.

## Now, let's begin ##

Before we proceed in coding, I want to discuss the relationship of the client and server, which are very essential in our chat application. 

Imagine a server as a waiter in a restaurant and a client as a customer. The waiter (server) is prepared to take orders and deliver food. The customer (client) chooses what they want, requests it from the waiter, and the waiter satisfies the request.

![picture alt](ClientandServer.png)

In the context of computers, the server is similar to the waiter, offering a service, and the client is akin to the customer, making requests to the server. Sockets act as the language through which the waiter and customer communicate – a shared understanding. The waiter remains attentive for requests (listens for customers), and upon a customer's arrival (when the client connects), they communicate to fulfill the customer's needs (whether it's data or a service). Following that, the customer departs (the connection is closed). Various guidelines (protocols like TCP or UDP) ensure a seamless conversation.

## **TCP and UDP** ##

![picture alt](TCP-vs-UDP.jpeg)

TCP or Transmission Control Protocol is a connection-oriented protocol that ensures the reliability and an ordered data delivery making it suitable for applications prioritizing data integrity like file transfers. If you're going to use TCP, you have to create a socket using *socket.socket()*, and specifying the socket type by using *socket.SOCK_STREAM*.

On the other hand, UDP or User Diagram Protocol is a connectionless protocol that focuses on speed more than reliability. This type of socket is used in real-time application like gaming and streaming. It is created using *socket.SOCK_DGRAM*.

## Chat Application ##
Now that the introduction is done, we can now proceed to the coding of our chat application. As I have said earlier, Client and Server is essential in our chat app. So, here's the Server:

## Server Code ##
```
import socket
import threading

HOST = socket.gethostbyname(socket.gethostname())
PORT = 5055

server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server.bind((HOST, PORT))
server.listen()
```
As you can see on the first part of our code I have imported socket and threading modules. Since we are creating an application where multiple tasks or function calls might happen, We have to import threading. We all know that threading is used to run multiple threads (tasks, function calls) at the same time, and our chat app can hold a vast amount of clients at the same time. So to run it smoothly we should use threading.

The **HOST** variable is assigned the IP address associated with the local machine, obtained using *socket.gethostbyname(socket.gethostname())*. This code is used to obtain your IPv4 without cheking your ip configuration in command prompt. You can also write your IPv4 like this *'127.0.0.1'* in a much easier way. The **PORT** variable is set to *5055*, which is I randomly selected. So the port variable depends on your liking as long as it is available.

I also created a new socket object which I named as *server*, and is created  using *socket.socket(socket.AF_INET, socket.SOCK_STREAM)*. So the *socket.AF_INET* specifies the address family which is the IPv4. While, *socket.SOCK_STREAM* specifies the socket type which is TCP. This is the type of socket we will use to have a reliable and organized conversation between clients.

The **bind** method is utilized on the server socket to link it with a particular host (IP address) and port. In this instance, it attaches the socket to the host acquired through *socket.gethostbyname(socket.gethostname())* (representing the local machine's IP address) and the designated port number, which is 5055. This action is essential to enable the server to be receptive to incoming connections on the explicitly specified IP address and port.

The **listen** method is invoked on the server socket, making it a listening socket. This prepares the server to accept incoming client connections. Without this step, the server wouldn't be ready to receive connection requests.

After we finished setting up TCP server we can now start creating functions to manage a basic chat server. To do that here's the code:
```
clients = []
nicknames = []

def broadcast(message):
    for client in clients:
        client.send(message)

def handle(client):
    while True:
        try:
            message = client.recv(1024).decode('utf-8')
            if message == "{quit}":
                break

            nickname = nicknames[clients.index(client)]
        
            broadcast(f"{nickname}: {message}".encode('utf-8'))

        except:
            index = clients.index(client)
            clients.remove(client)
            client.close()
            nickname = nicknames[index]
            broadcast(f'{nickname} has left the chat.'.encode('utf-8'))
            nicknames.remove(nickname)
            break
```
Since a chat app should be able to handle huge amount of clients, we should use *list* to store each client connected to the server, as well as their respective nicknames. Below the clients and nicknames list you can see the *broadcast* function which iterate through all connected clients in the clients list and sends a given message to each of them. The message is sent using the *send* method of the client's socket.

Of course our chat application should be able to continuously receive messages from the client that's why I used *while True* to run an infinite loop. Inside the loop we should use *client.recv(1024).decode('utf-8')* to attempt to receive a message from the client. If the message received from the client is "*{quit}*", it will break out of the loop, indicating that the client wants to leave the chat room.

Now, to obtain the client's nickname I used *nicknames[clients.index(client)]* which is based on the client's index in the client's list.

We cannot exclude how the message should be broadcast across the clients, that's why we have to call the *broadcast* function to send the message to all clients in this kind of format *"{nickname}: {message}"*

The last code which is *except* handles the situation where the client might be disconnected unexpectedly by removing the client and its nickname from the respective lists and broadcasting the message that the client has left the chat. So then the loop is exited with *break*

We now have come to the last set of codes before proceeding to the client's code. this code establishes a continuous loop to accept incoming client connections
```
while True:
    client, address = server.accept()
    print(f'Connection from {address} has been established!')

    nickname = client.recv(2024).decode('utf-8')
    nicknames.append(nickname)
    clients.append(client)

    print(f'Nickname is {nickname}')
    broadcast(f'Welcome to the chat, {nickname}!'.encode('utf-8'))


    thread = threading.Thread(target=handle, args=(client,))
    thread.start()
```
Again you can see that I used while loop so the server will continuously wait for and handle incoming client connections. ***client, address = server.accept()***, in this code you can see the *accept()* method so that the server will for a new client to connect. A method returns new socket *'client'* representing the connection and the client's address *'address'* when a client connects.

To receive nickname from the newly connected client we have to use *client.recv(1024).decode('utf-8')* to receive and decode the message. The received nickname is added to the nicknames list, and the client's socket is added to the clients list. These lists are presumably used to keep track of connected clients and their nicknames.

Of course, the client needs a welcome message so that the other clients will be notify that a new client have joined the chat, and to do that we should use the broadcast function in order for the other clients to see the newcomer.

To handle communication with the newly connected client concurrently with other clients, a new thread is created using *threading.Thread*. It is given the handle function as the target, which manages communication with individual clients. The thread is started using *thread.start()*, allowing the server to handle multiple client connections simultaneously.

## Client Code ##
Now that we're done with the server's code, we can now proceed to the next sets of codes from the client's side.

This side is where you'll be able to know how I used Tkinter for my GUI of this application.

For the first set of codes:
```
import socket
import tkinter as tk
import threading
from tkinter import simpledialog

# Client configuration
HOST = socket.gethostbyname(socket.gethostname())
PORT = 5055

# Socket setup
client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
client.connect((HOST, PORT))
```
In this code you can see that it's just the same from the server code, but this time I have imported *tkinter* for the GUI of the application. 

You can also see that I used *client.connect((HOST, PORT))* instead of *.bind()*.  This code establishes a connection to the server using the specified host (IP address) and port. This assumes that a server is running on the specified host and port.

```
# Function to send messages
def send_message(event=None):
    message = my_msg.get()
    my_msg.set("")  # Clears input field
    client.send(message.encode('utf-8'))

# Function to close the application
def on_closing(event=None):
    my_msg.set("{quit}")
    send_message()

# Function to get the nickname from the user
def get_nickname():
    global nickname
    nickname = simpledialog.askstring("Name", "Enter your Name:")
    client.send(nickname.encode('utf-8'))
    # Show the main window after getting the nickname
    root.deiconify()
```
The first def function is to give the client's ability to send message on the input field by pressing the *Enter* key or just by simply clicking the send button. So It retrieves the message entered by the user from the input field *my_msg.get()*, clears the input field *my_msg.set("")*, and sends the message to the server using the client.send method. The message is encoded in UTF-8 before being sent.

When you're done talking or want to leave the chat room *on_closing* function is called when the client closes the application. It sets the message to "{quit}" using my_msg.set("{quit}") and then calls the send_message function to send this quit message to the server. This mechanism allows the server to recognize that the client is quitting.

Since you need a nickname to use the chat app I used *get_nickname* function where it ask the user to enter their nickname using *simpledialog.askstring*. The entered nickname is then sent to the server using client.send after encoding it in UTF-8. 

After getting the nickname of the user, the window will disappear and the main window which is the root will be visible using *root.deiconify()*.

After this def functions we need to set up the GUI of our chat app
```
# Tkinter setup
root = tk.Tk()
root.title("Chat Application")

# Hide the main window initially
root.withdraw()

# Call the function to get the nickname
get_nickname()

# Label to display the nickname
nickname_label = tk.Label(root, text=f"iChat: {nickname}", bg="grey", fg="white")
nickname_label.pack(anchor=tk.N, padx=5, pady=5)

# Chat display area with border
chat_frame = tk.Frame(root, bd=2, relief=tk.SUNKEN)
chat_frame.pack(padx=10, pady=10)

chat_area = tk.Text(chat_frame, height=20, width=50, state=tk.DISABLED)
chat_area.pack()

# Frame to hold entry field and button
entry_frame = tk.Frame(root)
entry_frame.pack(pady=5)

# Message input field
my_msg = tk.StringVar()
entry_field = tk.Entry(entry_frame, textvariable=my_msg, width=45)
entry_field.bind("<Return>", send_message)
entry_field.pack(side=tk.LEFT)

# Send button with grey background and white text
send_button = tk.Button(entry_frame, text="Send", command=send_message, bg="grey", fg="white")
send_button.pack(side=tk.LEFT, padx=5)

# Close the client when the window is closed
root.protocol("WM_DELETE_WINDOW", on_closing)

# Receive and display messages
def receive():
    while True:
        try:
            message = client.recv(1024).decode('utf-8')
            if message == "{quit}":
                break

            # Display the formatted message with sender's nickname
            chat_area.configure(state=tk.NORMAL)  # Enable editing
            chat_area.insert(tk.END, f"{message}\n")
            chat_area.configure(state=tk.DISABLED)  # Disable editing
            chat_area.yview(tk.END)

        except OSError:
            break

# Start receiving messages in a new thread
receive_thread = threading.Thread(target=receive)
receive_thread.start()

# Run the Tkinter main loop
root.mainloop()
```
This set of codes allows users to enter their nickname, send messages, and receive messages in real-time. It also ensures that the application can be closed gracefully. The GUI components are updated based on user interactions and incoming messages from the server.

But, as you can see in the above code def receive is different from the others since its function continuously listens for incoming messages from the server in a separate thread. When a message is received, it is displayed in the chat area, and the display is updated to show the latest message. If the received message is "{quit}", indicating that the server has closed the connection, the loop is exited.

*receive_thread* is created to run the *receive* function concurrently since it is continuously listens for incoming messages that the server will broadcast across all the clients connected to the server.

I didn't explain the tkinter any further since the explanation is in the code already, and it is easy to understand than CSS.

## We have come to an end ##

***To sum it up***, creating a real-time chat app using Python sockets and Tkinter has given us a good understanding of how networks and user interfaces work together. We used sockets to make sure messages flow smoothly between users and a server, and Tkinter helped make the chat app easy to use and nice to look at.

Building this chat app not only helped us get better at programming but also showed us how to manage multiple things happening at the same time using threading. From setting up how messages travel to designing how it all looks, each step taught us something important.

As we finish this journey, remember that we've just started. There are lots of ways to make chat apps even better – adding cool features, making it more secure, or trying out different tools. What we've learned here is a strong base for more adventures in making real-time communication apps.

I hope this chat app adventure sparked your interest and gave you the tools to explore more about how networks and programs talk to each other. Enjoy your coding journey!
