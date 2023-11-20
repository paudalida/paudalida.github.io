## Real-Time Chat Application using Python 

Hello people!

The goal of this blog is to share with you the real-time chat application that I made using Python, a socket, threading, and a Tkinter. This application will be divided into two components: the client and the server.

As previously stated, the code for client.py is provided below. 


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

#### Some PowerShell Code

```powershell
Write-Host "This is a powershell Code block";

# There are many other languages you can use, but the style has to be loaded first

ForEach ($thing in $things) {
    Write-Output "It highlights it using the GitHub style"
}
```
