1. Giới thiệu
Trước khi nghĩ đến chủ đề này mình có search trên viblo với keyword websocket viblo xem đã có ai viết về mục này chưa và kết quả là đã có rất nhiều bài viết về mục này nhưng đa số là lý thuyết. Để hiện thực hóa những lý thuyết đã được đọc thì trong phạm vi bài viết này mình xin giới thiệu về cách xây dựng 1 ứng dụng chat cơ bản sử dụng thư viện Ratchet - WebSockets for PHP . Lý thuyết cơ bản về WebSocket các bạn có thể tham khảo tại đây

2. Bắt đầu thử một bài cơ bản với PHP WebSocket
Là một người thích cái đẹp và ưa vẽ vời mình luôn mong muốn những sản phẩm của bản thân cho dù đơn giản nhất thì trước hết vẫn phải đẹp mắt. :v Vì thế lại một lần nữa nhờ đến Google với keyword chat box template và sau 2 phút mình đã chọn ra đc cái mặt tiền cho cái ứng dụng đơn giản này, nó ở đây. Tải về unzip ra ta có thư mục chat-box. Tạm thời chưa cần động đến nó, tiếp theo ta sẽ implement theo cái hello-world của Ratchet để xem nó là gì đã.

Vào luôn thư mục chat-box gõ lệnh composer require cboden/ratchet để thêm thư viện vào project. Sau đó ta thêm 1 class (src/ChatWebSocket/Chat.php) với nội dung như sau:

-----------------------

    namespace ChatWebSocket;
    use Ratchet\MessageComponentInterface;
    use Ratchet\ConnectionInterface;

    class Chat implements MessageComponentInterface {
        protected $clients;

        public function __construct() {
            $this->clients = new \SplObjectStorage;
        }

        public function onOpen(ConnectionInterface $conn) {
            // Store the new connection to send messages to later
            $this->clients->attach($conn);

            echo "New connection! ({$conn->resourceId})\n";
        }

        public function onMessage(ConnectionInterface $from, $msg) {
            $numRecv = count($this->clients) - 1;
            echo sprintf('Connection %d sending message "%s" to %d other connections' . "\n"
                , $from->resourceId, $msg, $numRecv);

            foreach ($this->clients as $client) {
                if ($from !== $client) {
                    // The sender is not the receiver, send to each client connected
                    $client->send($msg);
                }
            }
        }

        public function onClose(ConnectionInterface $conn) {
            // The connection is closed, remove it, as we can no longer send it messages
            $this->clients->detach($conn);

            echo "Connection {$conn->resourceId} has disconnected\n";
        }

        public function onError(ConnectionInterface $conn, \Exception $e) {
            echo "An error has occurred: {$e->getMessage()}\n";

            $conn->close();
        }
    }
-----------------------


Nhìn qua nội dung class ta cũng có thể hiểu được ý nghĩa của từng method qua comment bên cạnh, tóm tắt lại thì class này implement các sự kiện của WebSocket đó là onOpen, onMessage, onClose, onError. Tiếp tục ta sẽ tạo một shell script để mở 1 kết nối socket, tạo file script chat_server.php trong thư mục bin/ với nội dung như sau:

-------------------------------

    use Ratchet\Server\IoServer;
    use Ratchet\Http\HttpServer;
    use Ratchet\WebSocket\WsServer;
    use ChatWebSocket\Chat;

    require dirname(__DIR__) . '/vendor/autoload.php';

    $server = IoServer::factory(
        new HttpServer(
            new WsServer(
                new Chat()
            )
        ),
        8080
    );
    $server->run();

-------------------------------


Để chạy đoạn script trên ta chạy lệnh

-------------------------------
    php bin/chat_server.php
-------------------------------

Chạy xong câu lệnh đó ta đã mở được 1 connection websocket có thể lắng nghe mọi request trên cổng 8080. Bây giờ ở phía client ta sẽ thử test bằng cách như sau, mở 2 trình duyệt (Ví dụ Chrome và Firefox) lên ấn F12 mở console ra và paste dòng lệnh này vào để khởi tạo connect socket đến server ta đã chạy script ở trên.

-------------------------------
    var conn = new WebSocket('ws://localhost:8080');
    conn.onopen = function(e) {
        console.log("Connection established!");
    };

    conn.onmessage = function(e) {
        console.log(e.data);
    };
-------------------------------

Ở Chrome ta gõ vào console dòng lệnh conn.send('Hello World!'); thì ở bên Firefox message Hello World! sẽ ngay lập tức đc ghi ra console. Bằng cách test như vậy thì ta cũng có thể hiểu cơ bản về cơ chế hoạt động của nó. Giờ thì đến đoạn vẽ hươu vẽ vượn thôi. Thêm một vài đường cơ bản vào file mặt tiền mà ta đã get về ở ngay đâu tiên. Và cuối cùng ta có kết quả như sau: sẽ thấy message chát được broadcast


3. Kết luận
Bằng ứng dụng demo đơn giản trên ta có thể hiểu rõ hơn về Websocket. Trong phần tiếp mình sẽ tiếp tục vẽ vời thêm với một vài ý tưởng trong đầu để nghịch thêm về ratchet. Cảm ơn các bạn đã theo dõi bài viết. To be continue ...

4. Tham khảo
    4.1 https://viblo.asia/p/xay-dung-ung-dung-chat-su-dung-php-websocket-PdbknLYqGyA