## Real-Time Chat Application using Python 

Hello people!

The goal of this blog is to share with you the real-time chat application that I made using Python, a socket, threading, and a Tkinter. This application will be divided into two components: the client and the server.

As previously stated, the code for client.py is provided below: 


---

```tsql
import socket
import threading
import tkinter as tk 
from tkinter import scrolledtext
from tkinter import messagebox

HOST = '127.0.0.5'
PORT = 4321 

TOP = '#2c2c2c'
BG = '#3a3a3a'
OCEAN_BLUE = '#7856ff'
WHITE = "white"
FONT = ("monospace", 15)
BUTTON_FONT = ("monospace", 13)
SMALL_FONT = ("monospace", 10)

client = socket.socket(socket.AF_INET, socket.SOCK_STREAM) #create socket object 

def add_message(message):
    message_box.config(state=tk.NORMAL)
    message_box.insert(tk.END, message + '\n')
    message_box.config(state=tk.DISABLED)

def connect():
    try: 
        client.connect((HOST,PORT))
        print("Successfully Connected to the server")
        add_message("[SERVER] Successfully connected to the serve")

    except:
        messagebox.showerror("Unable to connect to server", f"Unable to connect to server {HOST} {PORT}")

    username = username_textbox.get()
    if username != '':
        client.sendall(username.encode())
    else:
        messagebox.showerror("Invalid username", "Username cannot be empty")

    threading.Thread(target=listen_for_messages_from_server, args=(client, )).start()

    username_textbox.config(state=tk.DISABLED)
    username_button.config(state=tk.DISABLED)

def send_message():
    message = message_textbox.get()
    if message != '':
        client.sendall(message.encode())
        message_textbox.delete(0, len(message))
    else:
        messagebox.showerror("Empty message", "Message cannot be empty")

root = tk.Tk()
root.geometry("600x600")
root.title("Real-Time Chat")
root.resizable(False, False)

root.grid_rowconfigure(0, weight=1)
root.grid_rowconfigure(1, weight=4)
root.grid_rowconfigure(2, weight=1)

top_frame = tk.Frame(root, width=500, height=100, bg=TOP)
top_frame.grid(row=0, column=0, sticky=tk.NSEW)

middle_frame = tk.Frame(root, width=500, height=400, bg=BG)
middle_frame.grid(row=1, column=0, sticky=tk.NSEW)

bottom_frame = tk.Frame(root, width=500, height=100, bg=TOP)
bottom_frame.grid(row=2, column=0, sticky=tk.NSEW)

username_label = tk.Label(top_frame, text="Enter username:", font=FONT, bg=TOP, fg=WHITE)
username_label.pack(side=tk.LEFT, padx=10)

username_textbox = tk.Entry(top_frame, font=FONT, bg=BG, fg=WHITE, width=29)
username_textbox.pack(side=tk.LEFT)

username_button = tk.Button(top_frame, text=" Join ", font=BUTTON_FONT, bg=OCEAN_BLUE, fg=WHITE, width=6, command=connect)
username_button.pack(side=tk.LEFT, padx=21)

message_textbox = tk.Entry(bottom_frame, font=FONT, bg=BG, fg=WHITE, width=43)
message_textbox.pack(side=tk.LEFT, padx=15)

message_button = tk.Button(bottom_frame, text=" Send ", font=BUTTON_FONT, bg=OCEAN_BLUE, fg=WHITE, command=send_message)
message_button.pack(side=tk.LEFT, padx=10)

message_box = scrolledtext.ScrolledText(middle_frame, font=SMALL_FONT, bg=BG, fg=WHITE, width=70, height=26.5)
message_box.config(state=tk.DISABLED)
message_box.pack(side=tk.TOP)

def listen_for_messages_from_server(client):

    while 1:

        message = client.recv(2048).decode('utf-8')
        if message != '':
            username = message.split("~")[0]
            content = message.split('~')[1]

            add_message(f"[{username}] {content}")
            
        else:
            messagebox.showerror("Error", "Message recevied from client is empty")

# main function
def main():

    root.mainloop()
    
if __name__ == '__main__':
    main()

```
Explanation: 

Using Python's socket module and the Tkinter package for the graphical user interface (GUI), this code creates a basic real-time chat client. To join the chat, users must enter their username when the chat client connects to the server that the HOST and PORT variables specify. Users can submit messages by putting them in the input field and pressing the "Send" button. The chat messages are shown in a scrollable text box.
1. Import Statements:
   - socket: Provides a low-level interface for network communication.
   - threading: Used for creating and managing threads to handle concurrent execution.
   - tkinter: Python's standard GUI (Graphical User Interface) package.
   - scrolledtext: A widget in Tkinter for displaying text with a vertical scrollbar.
     
2. Constants:
   - HOST and PORT: Define the server's host address and port number.
   - Various color constants and font styles for the GUI.
       
3. Socket Initialization:
   - client: Creates a socket object for communication with the server.
     
4. GUI Initialization:
   - The GUI is created using tkinter. Three frames (top_frame, middle_frame, and bottom_frame) are used to organize the layout.
   - Entry fields, labels, and buttons are placed in the frames to create a user-friendly interface.
     
5. Functions:
   - add_message(message): Appends a message to the chat box.
   - connect(): Handles the connection to the server, sends the username, and starts a thread to listen for incoming messages.
   - send_message(): Sends a message to the server.
   - listen_for_messages_from_server(client): Listens for messages from the server in a separate thread.
     
6. Main Loop:
   - root.mainloop(): Enters the main event loop for the GUI.

7. Execution:
   - if __name__ == '__main__': main(): Calls the main() function when the script is executed.

The code for server.py is provided below:

```
import socket # will use for main communication part 
import threading 

HOST = '127.0.0.5'
PORT = 4321
LISTENER_LIMIT = 7
active_clients = []

def listen_for_messages(client, username):

    while 1:

        message = client.recv(2048).decode('utf-8')
        if message != '':
            
            final_msg = username + '~' + message
            send_messages_to_all(final_msg)

        else:
            print(f"The message send from client {username} is empty")

def send_message_to_client(client,message):

    client.sendall(message.encode())

def send_messages_to_all(message):

    for user in active_clients:

        send_message_to_client(user[1], message)

def client_handler(client):

    while 1: 

        username = client.recv(2048).decode('utf-8')
        if username !='':
            active_clients.append((username, client))
            prompt_message = "SERVER~" + f"{username} added to the chat"
            send_messages_to_all(prompt_message)
            break
        else:
            print("Client username is empty")
    
    threading.Thread(target=listen_for_messages, args=(client, username, )).start()


def main():

    server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

    try:
        server.bind((HOST, PORT))
        print(f"Running the server on {HOST} {PORT}")
    
    except: 
        print(f"Unable to bind to host {HOST} and port {PORT}")

    server.listen(LISTENER_LIMIT)

    while 1:

        client, address = server.accept()
        print(f"Successfully connected to client {address[0]} {address[1]}")

        threading.Thread(target=client_handler, args=(client, )).start()


if __name__ == '__main__':
    main()

```
Explanation: 

Using Python's socket module and threading, this code creates a basic server for a multi-client chat program. The server manages new client registrations, keeps an eye out for incoming connections from clients, and employs distinct threads to broadcast messages to all clients and listen for messages from each connected client.
1. Import Statements:
   - socket: Provides a low-level interface for network communication.
   - threading: Used for creating and managing threads to handle concurrent execution.
     
2. Constants:
   - HOST and PORT: Define the server's host address and port number.
   - LISTENER_LIMIT: Maximum number of clients the server can listen to simultaneously.
   - active_clients: A list to keep track of connected clients, where each element is a tuple containing the client's username and the client socket.
     
3. Functions:
   - listen_for_messages(client, username): Listens for messages from a specific client and broadcasts them to all clients.
   - send_message_to_client(client, message): Sends a message to a specific client.
   - send_messages_to_all(message): Broadcasts a message to all connected clients.
   - client_handler(client): Handles the addition of a new client, sends a prompt message to all clients, and starts a thread to listen for
     messages from the new client.
    
4. Main Function (main()):
   - Creates a server socket and binds it to the specified host and port.
   - Listens for incoming connections, and when a client connects, it starts a new thread to handle that client using the client_handler function.
     
5. Execution (if __name__ == '__main__': main()):
   - Calls the main() function when the script is executed.
  
TO RUN THE CODE: 

Open the CMD to run then type in these code: 

![image](https://github.com/paudalida/paudalida.github.io/assets/151016630/63e9e31f-8471-41b8-b18b-600eaababfba)


![image](https://github.com/paudalida/paudalida.github.io/assets/151016630/95aabf81-0faa-43ae-aef3-7cf630e3c9d2)

After you have typed in these commands. The final output will be like this: 

![image](https://github.com/paudalida/paudalida.github.io/assets/151016630/7d38d810-8385-4552-881a-d74bfce3103e)

   
