# socket访问本地服务器

## 代码

```python

import socket
import threading
import os
import time


# Define the host and port number for the server
HOST = '127.0.0.1'
PORT = 8080

# Define the root directory for the server
ROOT_DIRECTORY = os.getcwd()
print ("当前工作目录",ROOT_DIRECTORY)
# Define the response status codes and messages
STATUS_CODES = {
    200: 'OK',
    404: 'Not Found'
}

# Define a function to handle client requests
def handle_client(client_socket):
    # Receive the HTTP request from the client
    request = client_socket.recv(1024).decode('utf-8')
    print(request)

    # Parse the HTTP request to determine the requested file
    request_parts = request.split()
    request_file = request_parts[1]
    print("请求内容",request_parts)
    request_file_list =list(request_file[1:])
    if request_file_list:
        print('需要的文件',request_file_list)
        request_file_used=''.join(request_file_list)
    if request_file_list:
        pass
    else:
        request_file_used = "index.html"
    # Check if the requested file exists
    file_path = ROOT_DIRECTORY+os.path.join("\Html",request_file_used)
    print("请求路径",file_path)
    if not os.path.exists(file_path):
        # If the file does not exist, send a "404 Not Found" error message
        response = 'HTTP/1.1 404 Not Found\r\n\r\n'
        response += '<h1>404 Not Found</h1>'
        client_socket.sendall(response.encode('utf-8'))
        client_socket.close()
        return

    # Read the requested file
    with open(file_path, 'rb') as f:
        file_content = f.read()
        print(file_content)
    # Create the HTTP response message
    response = 'HTTP/1.1 200 OK\r\n'
    response += 'Content-Type: text/html\r\n'
    response += 'Content-Length: {}\r\n'.format(len(file_content))
    response += '\r\n'
    response = response.encode('utf-8') + file_content

    # Send the HTTP response to the client
    client_socket.sendall(response)


    # Log the request to a file
    with open('log.txt', 'a') as f:
        f.write('{} {} {}\n'.format(client_socket.getpeername()[0], time.strftime('%Y-%m-%d %H:%M:%S'), request_parts[1], STATUS_CODES[200]))
    client_socket.close()#关闭socket的时机
# Create the server socket
server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server_socket.bind((HOST, PORT))
server_socket.listen()

print('Server running on {}:{}'.format(HOST, PORT))

# Start the main server loop
while True:
    time.sleep(1)
    # Accept a client connection
    print("..")
    client_socket, client_address = server_socket.accept()
    print('Client connected: {}'.format(client_address))

    # Create a new thread to handle the client request
    client_thread = threading.Thread(target = handle_client , args=(client_socket,))
    client_thread.start()
    # handle_client(client_socket)

```

