# 1. TCP的socket
Server的代码：
```
import java.io.IOException;
import java.io.InputStream;
import java.net.ServerSocket;
import java.net.Socket;

public class ServerThread extends Thread {
    @Override
    public void run() {
        try {
            // 创建服务器Socket，监听指定端口
            ServerSocket serverSocket = new ServerSocket(12345);

            while (true) {
                // 等待客户端连接
                Socket socket = serverSocket.accept();
                handleClient(socket);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    private void handleClient(Socket socket) throws IOException {
        // 获取客户端输入流
        InputStream inputStream = socket.getInputStream();

        // 读取客户端发送的数据
        byte[] buffer = new byte[1024];
        int length;
        while ((length = inputStream.read(buffer)) != -1) {
            String message = new String(buffer, 0, length);
            System.out.println("Received message: " + message);

            // 在此可以对接收到的数据进行处理

            // 回复客户端
            socket.getOutputStream().write("Hello, client!".getBytes());
        }

        // 关闭连接
        socket.close();
    }
}
```

客户端代码：
```
import java.io.IOException;
import java.net.Socket;

public class ClientThread extends Thread {
    @Override
    public void run() {
        try {
            // 连接到服务器
            Socket socket = new Socket("SERVER_IP_ADDRESS", 12345);

            // 发送数据到服务器
            socket.getOutputStream().write("Hello, server!".getBytes());

            // 接收服务器回复的数据
            byte[] buffer = new byte[1024];
            int length = socket.getInputStream().read(buffer);
            String message = new String(buffer, 0, length);
            System.out.println("Received message from server: " + message);

            // 关闭连接
            socket.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

```

总结：TCP的Socket是以流的方式进行通信。


# 2. UDP的Socket

Server代码：
```
import java.io.IOException;
import java.net.DatagramPacket;
import java.net.DatagramSocket;

public class UDPServerThread extends Thread {
    @Override
    public void run() {
        try {
            // 创建服务器 DatagramSocket，监听指定端口
            DatagramSocket serverSocket = new DatagramSocket(12345);

            while (true) {
                // 接收客户端发送的数据
                byte[] buffer = new byte[1024];
                DatagramPacket receivePacket = new DatagramPacket(buffer, buffer.length);
                serverSocket.receive(receivePacket);

                String message = new String(receivePacket.getData(), 0, receivePacket.getLength());
                System.out.println("Received message from client: " + message);

                // 在此可以对接收到的数据进行处理

                // 回复客户端
                String replyMessage = "Hello, client!";
                byte[] replyBuffer = replyMessage.getBytes();
                DatagramPacket replyPacket = new DatagramPacket(replyBuffer, replyBuffer.length, receivePacket.getAddress(), receivePacket.getPort());
                serverSocket.send(replyPacket);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

客户端代码：
```
import java.io.IOException;
import java.net.DatagramPacket;
import java.net.DatagramSocket;
import java.net.InetAddress;

public class UDPClientThread extends Thread {
    @Override
    public void run() {
        try {
            // 创建客户端 DatagramSocket
            DatagramSocket clientSocket = new DatagramSocket();

            // 发送数据到服务器
            String message = "Hello, server!";
            byte[] buffer = message.getBytes();
            InetAddress serverAddress = InetAddress.getByName("SERVER_IP_ADDRESS");
            DatagramPacket sendPacket = new DatagramPacket(buffer, buffer.length, serverAddress, 12345);
            clientSocket.send(sendPacket);

            // 接收服务器的响应数据
            byte[] replyBuffer = new byte[1024];
            DatagramPacket replyPacket = new DatagramPacket(replyBuffer, replyBuffer.length);
            clientSocket.receive(replyPacket);

            String replyMessage = new String(replyPacket.getData(), 0, replyPacket.getLength());
            System.out.println("Received reply from server: " + replyMessage);

            // 关闭连接
            clientSocket.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

总结：UDP的socket是基于数据块，以Datagram形式进行通信。且发送数据直接调用send方法，接受数据使用receive方法。
