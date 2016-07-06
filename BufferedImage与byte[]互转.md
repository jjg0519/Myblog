##BufferedImage与byte[]互转

[TOC]

###需要用到的类

Java.awt.image.BufferedImage;
javax.imageio.ImageIO;
java.io.*;

###为什么要将BufferedImage转为byte数组

在传输中，图片是不能直接传的，因此需要把图片变为字节数组，然后传输比较方便；只需要一般输出流的write方法即可；
而字节数组变成BufferedImage能够还原图像；

###如何取得BufferedImage

BufferedImage image = ImageIO.read(new File("1.gif"));

###BufferedImage  ---->byte[]

ImageIO.write(BufferedImage image,String format,OutputStream out);方法可以很好的解决问题；
参数image表示获得的BufferedImage；
参数format表示图片的格式，比如“gif”等；
参数out表示输出流，如果要转成Byte数组，则输出流为ByteArrayOutputStream即可；
执行完后，只需要toByteArray()就能得到byte[];

###byte[] ------>BufferedImage

ByteArrayInputStream in = new ByteArrayInputStream(byte[]b);    //将b作为输入流；
BufferedImage image = ImageIO.read(InputStream in);     //将in作为输入流，读取图片存入image中，而这里in可以为ByteArrayInputStream();

###显示BufferedImage

public void paint(Graphics g){
super.paint(g);
g.drawImage(image,x,y,width,height,null);    //image为BufferedImage类型
}
如果要自动调用paint方法，则需要调用repaint()方法；

### 实例

要求：编写一个网络程序，通过Socket将图片从服务器端传到客户端，并存入文件系统；
Server端：

```java
package org.exam3;  
  
import java.awt.image.BufferedImage;  
import java.io.ByteArrayOutputStream;  
import java.io.DataOutputStream;  
import java.io.File;  
import java.net.ServerSocket;  
import java.net.Socket;  
  
import javax.imageio.ImageIO;  
  
public class T6Server {  
  
    public static void main(String[] args) throws Exception {  
        ServerSocket server = new ServerSocket(8889);  
        System.out.println("服务器开启连接...端口为8889");  
        Socket s = server.accept();  
        while(true){  
            System.out.println("一客户端连接服务器，服务器传输图片...");  
            DataOutputStream dout = new DataOutputStream(s.getOutputStream());  
            BufferedImage image = ImageIO.read(new File("1.gif"));  //读取1.gif并传输  
            ByteArrayOutputStream out = new ByteArrayOutputStream();  
            boolean flag = ImageIO.write(image, "gif", out);  
            byte[] b = out.toByteArray();  
            dout.write(b);  
            s.close();  
            System.out.println("图片传送完毕,服务器开启连接...端口为8889");  
            s = server.accept();  
              
        }  
    }  
}  
```

Client端：

```java
package org.exam3;  
  
import java.awt.BorderLayout;  
import java.awt.Graphics;  
import java.awt.event.ActionEvent;  
import java.awt.event.ActionListener;  
import java.awt.image.BufferedImage;  
import java.io.ByteArrayInputStream;  
import java.io.ByteArrayOutputStream;  
import java.io.DataInputStream;  
import java.io.File;  
import java.io.PrintWriter;  
import java.net.Socket;  
  
import javax.imageio.ImageIO;  
import javax.swing.JButton;  
import javax.swing.JFrame;  
import javax.swing.JPanel;  
  
public class T6Client extends JFrame {  
    JButton button;  
    JPanel panel;  
    int count = 0;  
    BufferedImage image ;  
    public T6Client() {  
        setSize(300, 400);  
        button = new JButton("获取图像");  
        add(button,BorderLayout.NORTH);  
        button.addActionListener(new ActionListener() {  
            public void actionPerformed(ActionEvent event) {  
                try {  
                    Socket s = new Socket("localhost",8889);  
                    PrintWriter out = new PrintWriter(s.getOutputStream());  
                    out.print("a");  
                    DataInputStream in = new DataInputStream(s.getInputStream());  
                    byte[]b = new byte[1024];  
                    ByteArrayOutputStream bout = new ByteArrayOutputStream();  
                    int length = 0;  
                    while((length=in.read(b))!=-1){  
                        bout.write(b, 0, length);  
                    }  
                    ByteArrayInputStream bin = new ByteArrayInputStream(bout.toByteArray());  
                    image = ImageIO.read(bin);  
                    repaint();  
                    ImageIO.write(image, "gif", new File("output-"+count+".gif"));  
                    count++;  
                    s.close();  
                } catch (Exception e) {  
                }  
            }  
        });  
        panel = new JPanel();  
        add(panel);  
    }  
    @Override  
    public void paint(Graphics g){  
        super.paint(g);  
        g.drawImage(image, 20, 20, 300, 150, null);//image为BufferedImage类型  
    }  
    public static void main(String[] args) throws Exception {  
        T6Client frame = new T6Client();  
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);  
        frame.setVisible(true);  
    }  
}  
```