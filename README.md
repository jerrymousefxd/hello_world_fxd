package nioserver;

import java.net.InetSocketAddress;
import java.net.ServerSocket;
import java.nio.channels.SelectionKey;
import java.nio.channels.Selector;
import java.nio.channels.ServerSocketChannel;
import java.nio.channels.SocketChannel;
import java.util.Iterator;
import java.util.LinkedList;
import java.util.List;
import java.util.Set;


/**
 * <p>Title: 主控服务线程,采用NIO实现的长连接服务器</p>
 * @author sxl
 * @version 1.0
 */

public class Server implements Runnable {
    private static Selector selector;
    private ServerSocketChannel sschannel;
    private InetSocketAddress address;
    protected Notifier notifier;
    private int port;

    /**
     * 创建主控服务线程
     * @param port 服务端口
     * @throws java.lang.Exception
     */
    public static int MAX_THREADS = 4;
    public Server(int port) throws Exception {
        this.port = port;

        // 获取事件触发器
        notifier = Notifier.getNotifier();

        // 创建读写线程池
        for (int i = 0; i < MAX_THREADS; i++) {
            Thread r = new Reader();
            Thread w = new Writer();
            Thread sys = new Syslog();
            r.start();
            w.start();
            sys.start();
        }

        // 创建无阻塞网络套接
        selector = Selector.open();
        sschannel = ServerSocketChannel.open();
        sschannel.configureBlocking(false);
        address = new InetSocketAddress(port);
        ServerSocket ss = sschannel.socket();
        ss.bind(address);
        sschannel.register(selector, SelectionKey.OP_ACCEPT);
    }

    public void run() {
        System.out.println("Server started ...");
        System.out.println("Server listening on port: " + port);
        // 监听
        while (true) {
            try {
                int num = 0;
                num = selector.select();
                if (num > 0) {
                    Set selectedKeys = selector.selectedKeys();
                    Iterator it = selectedKeys.iterator();
                    while (it.hasNext()) {
                        SelectionKey key = (SelectionKey) it.next();
                        it.remove();
                        // 处理IO事件
                        if (key.isAcceptable()) {
                           // Accept the new connection
                           ServerSocketChannel ssc = (ServerSocketChannel) key.channel();
                           notifier.fireOnAccept();
                           SocketChannel sc = ssc.accept();
                           sc.configureBlocking(false);
                           // 触发接受连接事件
                           Request request = new Request(sc);
                           notifier.fireOnAccepted(request);
                           // 注册读操作,以进行下一步的读操作
                           sc.register(selector,  SelectionKey.OP_READ, request);
                       }
                       else if (key.isReadable()) {
                           Reader.processRequest(key);  // 提交读服务线程读取客户端数据
                       }
                    }
                }
            }
            catch (Exception e) {
                continue;
            }
        }
    }
}
