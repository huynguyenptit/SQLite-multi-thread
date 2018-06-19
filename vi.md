# Thực hiện thao tác ghi đa luồng trên SQLite trong cùng một thời điểm / Almost row level lock performance.

Các bạn thân mến,

Như bạn biết SQLite là cơ sở dữ liệu đơn luồng và mặc định được nhúng trên hệ điều hành Linux OS. Có rất nhiều các cuộc nghiên cứu về việc sử dụng SQLite để lưu trữ dữ liệu. Cũng có rất nhiều những nghiên cứu về việc truy cập CSDL SQLite đa luồng. Tôi sẽ chia sẻ một nghiên cứu nhỏ về việc làm sao để có được đa luồng trong CSDL SQLite.

Hãy cùng tìm hiểu về ưu nhược điểm của SQLite.

### Ưu điểm

* SQLite được viết hoàn toàn bằng C. Vì vậy nó có thể truy cập vào ổ đĩa hay bộ nhớ CSDL một cách nhanh nhất và xử lí dữ liệu. Hãy nghĩ như bạn đang sử dụng ổ SSD.
* SQLite hỗ trợ bộ nhớ trong. Bbộ nhớ trong SQLite nhanh gấp 2 lần. Nếu bạn hiểu vấn đề phân trang. Nó đủ nhanh để làm điều đó
* SQLite chạy đơn luồng. Vì vậy nguy cơ hỏng dữ liệu gần như là thấp nhất.
* CSDL SQLite trong 1 file độc lập. Vì thế chúng ta có thể di chuyển và truy cập nó hết sức đơn giản trên các nền tảng khác.
* SQLite là một hệ quản trị cho người dùng đầu cuối. 
* Lướt qua về nền tảng, SQLite có thể được sử dụng trên tất cả các nền tảng OS.
* Nó là mã nguồn mở !!!
* Vân vân ………..

### Nhược điểm.
* Như chúng tôi đã đề cập ở trên SQLite chạy đơn luồng. Vì vậy có nghĩa là SQLite chỉ có thể ghi duy nhất một hành động trong một thời điểm.
* Như chúng tôi đã đề cập SQLite giữ dữ liệu trong 1 file. Vì thế tất cả dữ liệu sẽ bị khóa trong khi đang ghi hoạt động. Đây là một bất lợi đối với các cơ sở dữ liệu lớn và sâu.
* Không có tầng yêu cầu xác thực.

####Tạo đa luồng ghi trong gần như một thời điểm.
Bây giờ tôi sẽ thử một mẹo nhỏ để ghi hành đông trong gần như 1 thời điểm.
#### Chú ý: SQLite không cho phép bạn hoàn thành ROW LEVER LOCK. Vì thế không cần phí thời gian để tìm kiếm nó.



Đương nhiên nó có thể tác động tới hiệu suất nếu bạn ghi hành động trên một phần của bảng lớn. Nhưng luồng thứ 2 sẽ không đợi quá lâu đến khi cái thứ nhất xong.



#### Đây là bước chính BƯỚC 1.7

Ban biết đấy, index B TREE là siêu nhanh. Và hành động GHI của bạn sẽ thực hiện theo ID từng dòng dòng đã được đánh index theo mặc định.

Ví dụ:

delete from table where name='Fariz' // Nó sẽ khóa DB chính trong 10s.

Thay thế nó bằng:

insert into tepm.Lock select rowid,'processid' from table where name = 'Fariz'; // Nó sẽ chỉ khóa cơ sở dữ liệu tạm thời trong 10s

delete from table where ROWID in ( select rowid from tepm.Lock where name='fariz' and processid='XYZ' ) // việc xóa sẽ khóa file DB trong 0.0001s.

Tôi vừa làm xong 1 thử nghiệm nhỏ và kết quả thật tuyệt vời với tôi. Tôi đã có thể hoàn thành 2 hành động update trong 11s mà đáng nhẽ mỗi cái sẽ mất 10s cho bảng cơ sở dữ liệu có 40tr bản ghi.

Hi vọng nó sẽ giúp ích cho những người dùng SQLite
