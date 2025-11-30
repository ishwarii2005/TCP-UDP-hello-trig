# TCP-UDP-hello-trig

Short answer:

1. TCP: Hello + File Transfer (Java)

We’ll make **two programs**:

* `TcpServer.java` – waits for client, receives “Hello”, sends “Hello”, then receives a file and saves it.
* `TcpClient.java` – connects to server, sends “Hello”, receives “Hello”, then sends a file to server.

### 🖥 `TcpServer.java`

    import java.io.*;
    import java.net.*;

    public class TcpServer {

    public static void main(String[] args) {
        final int PORT = 5000;

        try (ServerSocket serverSocket = new ServerSocket(PORT)) {
            System.out.println("TCP Server started on port " + PORT + ", waiting for client...");

            Socket socket = serverSocket.accept();
            System.out.println("Client connected: " + socket.getInetAddress());

            DataInputStream dis = new DataInputStream(socket.getInputStream());
            DataOutputStream dos = new DataOutputStream(socket.getOutputStream());

            // 1. Exchange "Hello" messages
            String clientMsg = dis.readUTF();
            System.out.println("Client says: " + clientMsg);

            String hello = "Hello from TCP server";
            dos.writeUTF(hello);
            dos.flush();
            System.out.println("Sent to client: " + hello);

            // 2. File transfer (receive file from client)
            String fileName = dis.readUTF();   // file name
            long fileSize = dis.readLong();    // file size in bytes

            System.out.println("Receiving file: " + fileName + " (" + fileSize + " bytes)");

            FileOutputStream fos = new FileOutputStream("received_" + fileName);
            byte[] buffer = new byte[4096];
            long remaining = fileSize;

            while (remaining > 0) {
                int bytesRead = dis.read(buffer, 0, (int) Math.min(buffer.length, remaining));
                if (bytesRead == -1) break;
                fos.write(buffer, 0, bytesRead);
                remaining -= bytesRead;
            }

            fos.close();
            System.out.println("File saved as: received_" + fileName);

            // Close all
            dis.close();
            dos.close();
            socket.close();
            System.out.println("TCP Server finished.");

        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    }




### 💻 `TcpClient.java`

```java
import java.io.*;
import java.net.*;
import java.util.Scanner;

public class TcpClient {

    public static void main(String[] args) {
        final String SERVER_IP = "127.0.0.1";
        final int PORT = 5000;

        try (Socket socket = new Socket(SERVER_IP, PORT)) {
            System.out.println("Connected to TCP server.");

            DataInputStream dis = new DataInputStream(socket.getInputStream());
            DataOutputStream dos = new DataOutputStream(socket.getOutputStream());
            Scanner sc = new Scanner(System.in);

            // 1. Exchange "Hello" messages
            String hello = "Hello from TCP client";
            dos.writeUTF(hello);
            dos.flush();
            System.out.println("Sent to server: " + hello);

            String serverMsg = dis.readUTF();
            System.out.println("Server says: " + serverMsg);

            // 2. File transfer (send file to server)
            System.out.print("Enter file path to send (e.g., C:\\temp\\test.txt): ");
            String filePath = sc.nextLine();

            File file = new File(filePath);
            if (!file.exists()) {
                System.out.println("File does not exist. Exiting.");
                return;
            }

            String fileName = file.getName();
            long fileSize = file.length();

            // Send metadata
            dos.writeUTF(fileName);
            dos.writeLong(fileSize);
            dos.flush();

            System.out.println("Sending file: " + fileName + " (" + fileSize + " bytes)");

            FileInputStream fis = new FileInputStream(file);
            byte[] buffer = new byte[4096];
            int bytesRead;

            while ((bytesRead = fis.read(buffer)) != -1) {
                dos.write(buffer, 0, bytesRead);
            }
            dos.flush();
            fis.close();

            System.out.println("File sent successfully.");

            // Close all
            dis.close();
            dos.close();
            System.out.println("TCP Client finished.");

        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

---

## 2. UDP: Hello + Trigonometric Calculator (Java)

We’ll make:

* `UdpServer.java` – waits for UDP packets; replies “Hello” for HELLO message, and does trig operations like `sin 30`, `cos 60`, `tan 45`.
* `UdpClient.java` – sends a HELLO message, then sends trig queries.

### 🌐 `UdpServer.java`

```java
import java.net.*;
import java.io.*;

public class UdpServer {

    public static void main(String[] args) {
        final int PORT = 6000;
        byte[] buffer = new byte[1024];

        try (DatagramSocket socket = new DatagramSocket(PORT)) {
            System.out.println("UDP Server started on port " + PORT);

            while (true) {
                DatagramPacket packet = new DatagramPacket(buffer, buffer.length);
                socket.receive(packet);

                String received = new String(packet.getData(), 0, packet.getLength()).trim();
                System.out.println("Received: " + received);

                String response;

                if (received.equalsIgnoreCase("HELLO")) {
                    response = "Hello from UDP server";
                } else {
                    // Expect input as: "sin 30" or "cos 60" or "tan 45"
                    String[] parts = received.split("\\s+");
                    if (parts.length == 2) {
                        String op = parts[0].toLowerCase();
                        double angleDeg;
                        try {
                            angleDeg = Double.parseDouble(parts[1]);
                            double angleRad = Math.toRadians(angleDeg);
                            double result;

                            switch (op) {
                                case "sin": result = Math.sin(angleRad); break;
                                case "cos": result = Math.cos(angleRad); break;
                                case "tan": result = Math.tan(angleRad); break;
                                default: result = Double.NaN;
                            }

                            if (Double.isNaN(result)) {
                                response = "Invalid operation. Use sin/cos/tan.";
                            } else {
                                response = op + "(" + angleDeg + ") = " + result;
                            }
                        } catch (NumberFormatException e) {
                            response = "Invalid angle. Send like: sin 30";
                        }
                    } else {
                        response = "Invalid format. Use: sin 30";
                    }
                }

                byte[] sendData = response.getBytes();
                DatagramPacket sendPacket = new DatagramPacket(
                        sendData,
                        sendData.length,
                        packet.getAddress(),
                        packet.getPort()
                );
                socket.send(sendPacket);
            }

        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

---

### 📡 `UdpClient.java`

```java
import java.net.*;
import java.io.*;
import java.util.Scanner;

public class UdpClient {

    public static void main(String[] args) {
        final String SERVER_IP = "127.0.0.1";
        final int PORT = 6000;

        try (DatagramSocket socket = new DatagramSocket();
             Scanner sc = new Scanner(System.in)) {

            // 1. Send HELLO
            String hello = "HELLO";
            byte[] sendData = hello.getBytes();
            DatagramPacket sendPacket = new DatagramPacket(
                    sendData,
                    sendData.length,
                    InetAddress.getByName(SERVER_IP),
                    PORT
            );
            socket.send(sendPacket);

            byte[] buffer = new byte[1024];
            DatagramPacket recvPacket = new DatagramPacket(buffer, buffer.length);
            socket.receive(recvPacket);

            String serverReply = new String(recvPacket.getData(), 0, recvPacket.getLength());
            System.out.println("Server: " + serverReply);

            // 2. Trigonometric calculator
            System.out.println("Enter trigonometric queries like: sin 30, cos 60, tan 45");
            System.out.println("Type 'exit' to quit.");

            while (true) {
                System.out.print("You: ");
                String msg = sc.nextLine();
                if (msg.equalsIgnoreCase("exit")) break;

                sendData = msg.getBytes();
                sendPacket = new DatagramPacket(
                        sendData,
                        sendData.length,
                        InetAddress.getByName(SERVER_IP),
                        PORT
                );
                socket.send(sendPacket);

                recvPacket = new DatagramPacket(buffer, buffer.length);
                socket.receive(recvPacket);

                String resp = new String(recvPacket.getData(), 0, recvPacket.getLength());
                System.out.println("Server: " + resp);
            }

            System.out.println("UDP Client finished.");

        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

---

## 3. How to Run All This in VS Code (Step-by-step)

### ✅ Pre-requisite: Java installed

In VS Code **Terminal**:

```bash
java -version
javac -version
```

If versions appear → good.
If not → you need JDK installed (you can tell me if it's missing).

---

### ✅ Step 1: Create files

In one folder (say `C:\Users\Ishwari\Desktop\network_lab`), create:

* `TcpServer.java`
* `TcpClient.java`
* `UdpServer.java`
* `UdpClient.java`

Paste the code I gave into each matching file.

Make sure **no `package` line** at top.

---

### ✅ Step 2: Compile

In VS Code terminal, `cd` into that folder:

```bash
cd C:\Users\Ishwari\Desktop\network_lab
```

Then compile all:

```bash
javac TcpServer.java TcpClient.java UdpServer.java UdpClient.java
```

If no errors → ✅ you’re ready.

---

### ✅ Step 3: Run TCP Hello + File Transfer

1. First run **server**:

```bash
java TcpServer
```

2. In a **second terminal** (or new VS Code terminal tab), run **client**:

```bash
java TcpClient
```

3. When TCP client asks:

```text
Enter file path to send:
```

Give something like:

```text
C:\Users\Ishwari\Desktop\test.txt
```

After success:

* Server saves it as: `received_test.txt` in the same folder.

---

### ✅ Step 4: Run UDP Hello + Trig Calculator

1. Run **UDP server** first:

```bash
java UdpServer
```

2. In another terminal, run **UDP client**:

```bash
java UdpClient
```

Then type things like:

```text
sin 30
cos 60
tan 45
exit
```
