# ESP32 Webserver

This is a simple webserver project using ESP32 and MicroPython. It allows you to control an LED connected to pin 2 of the ESP32 board through a web browser.

## Requirements

- An ESP32 board
- An LED and a resistor
- A USB cable
- A WiFi network
- MicroPython firmware installed on the ESP32 board
- A code editor (such as Thonny or uPyCraft)

## Wiring

Connect the LED and the resistor in series between pin 2 and GND of the ESP32 board. The longer leg of the LED should be connected to pin 2, and the shorter leg to the resistor.

## Code

The code consists of two parts: importing the libraries and setting up the WiFi connection, and creating and running the webserver.

### Importing libraries and setting up WiFi

The following libraries are imported:

- `machine` for accessing the hardware pins
- `usocket` for creating sockets
- `time` for delaying operations
- `network` for connecting to WiFi

A timeout variable is set to limit the time for connecting to WiFi. A WiFi object is created using the `network.WLAN` class, and its mode is set to station (STA). The WiFi is restarted by turning it off and on, and then it is connected to a network using its SSID and password. A loop is used to check if the WiFi is connected, and if not, it waits for one second and increments the timeout variable. If the WiFi is connected, it prints the network configuration, such as the IP address.

### Creating and running the webserver

A HTML document is defined as a multi-line string, which contains the title, heading, and buttons for controlling the LED. The buttons have names and values that will be used to identify them in the code.

An output pin object is created using the `machine.Pin` class, and its initial value is set to 0 (off).

A socket object is created using the `usocket.socket` class, and its family and type are set to IPv4 and TCP respectively. The socket is bound to an empty host (which means any IP address can connect) and port 80 (which is the default port for HTTP). The socket is set to listen for up to 5 clients at a time.

A while loop is used to accept incoming connections from clients, and store their socket objects and addresses. The request from the client is received as bytes and converted to a string. The request is searched for substrings that indicate which button was pressed by the client. If the button value is ON, the LED pin value is set to 1 (on). If the button value is OFF, the LED pin value is set to 0 (off).

The HTML document is sent as a response to every client, and then the connection socket is closed.

## Running

To run the code, you need to upload it to your ESP32 board using your code editor. Make sure you have selected the correct port and baud rate for your board. You can also use a serial monitor to see the output of your code.

Once the code is uploaded, you can open a web browser on any device connected to the same WiFi network as your ESP32 board. Enter the IP address of your board in the address bar, followed by a colon and port 80. For example, if your board's IP address is 192.168.1.10, you would enter http://192.168.1.10:80 in your browser.

You should see a web page with a title, heading, and two buttons for controlling the LED. You can click on either button to turn the LED on or off.

## Code block

Here is the python code in a code block:

```python
import machine
import usocket as socket
import time
import network


timeout = 0 # WiFi Connection Timeout variable 

wifi = network.WLAN(network.STA_IF)

# Restarting WiFi
wifi.active(False)
time.sleep(0.5)
wifi.active(True)

wifi.connect('WIFI_786','WIFI_786786')

if not wifi.isconnected():
    print('connecting..')
    while (not wifi.isconnected() and timeout < 5):
        print(5 - timeout)
        timeout = timeout + 1
        time.sleep(1)
        
if(wifi.isconnected()):
    print('Connected...')
    print('network config:', wifi.ifconfig())
    
# HTML Document

html='''<!DOCTYPE html>
<html>
<center><h2>ESP32 Webserver </h2></center>
<form>
<center>
<h3> LED </h3>
<button name="LED" value='ON' type='submit'>  ON </button>
<button name="LED" value='OFF' type='submit'> OFF </button>
</center>
'''

# Output Pin Declaration 
LED = machine.Pin(2,machine.Pin.OUT)
LED.value(0)

# Initialising Socket
s = socket.socket(socket.AF_INET,socket.SOCK_STREAM) # AF_INET - Internet Socket, SOCK_STREAM - TCP protocol

Host = '' # Empty means, it will allow all IP address to connect
Port = 80 # HTTP port
s.bind((Host,Port)) # Host,Port

s.listen(5) # It will handle maximum 5 clients at a time

# main loop
while True:
  connection_socket,address=s.accept() # Storing Conn_socket & address of new client connected
  print("Got a connection from ", address)
  request=connection_socket.recv(1024) # Storing Response coming from client
  print("Content ", request) # Printing Response 
  request=str(request) # Coverting Bytes to String
  # Comparing & Finding Postion of word in String 
  LED_ON =request.find('/?LED=ON')
  LED_OFF =request.find('/?LED=OFF')
  
  if(LED_ON==6):
    LED.value(1)
    
  if(LED_OFF==6):
    LED.value(0)
    
  # Sending HTML document in response everytime to all connected clients  
  response=html 
  connection_socket.send(response)
  
  #Closing the socket
  connection_socket.close()
```
