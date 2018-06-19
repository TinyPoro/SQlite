
Chào các bạn của tui,

Như các bạn đã biết thì SQLite là cơ sở dữ liệu đơn luồng mặc định được nhúng vào Linux OS. Có rất nhiều nghiên cứu về việc lưu trữ dữ liệu sử dụng SQLite. Cũng có nhiều nghiên cứu về cách truy cập cơ sở dữ liệu SQLite cho các quá trình ghi đa luồng. Tôi sẽ chia sẻ nghiên cứu nhỏ của tôi về cách thực hiện hoạt động ghi đa luồng trên SQLite DB.

Trước tiên hay tìm hiểu về các lợi thế và bất lợi của SQLite.

## Các lợi thế.
- SQLite is written pure C programming language. So it is fastest access to DISK or Memory database and process data. Think about if you use SSD disk. 
- SQLite được viết bằng ngôn ngữ lập trình C thuần. Vì thế nó có tốc độ truy cập 
- SQLite support in-memory. In memory SQLite almost 2 time fast. If you understand paging issue. It is enough fast.
- SQLite is single thread. So it lowest risk to corrupt data. 
- SQLite database in single file. So we can move database and access by another platform very easyly. 
- SQLite is 0 administration for end user. 
- Cross platform. SQLite can be used on all major OS platform. 
- OPENSOURCE OPENSOURCE OPENSOURCE !!!
- And so on ………..

## 1 vài bất lợi

- Như chúng tôi đã nói thì SQLite hoạt động đơn luồng. Có nghĩa là SQlite chỉ thực hiện 1 hoạt động tại 1 thời điểm nhất định.
- Chúng tôi cũng đã nói SQLite giữ dữ liệu gốc trong 1 file. Có nghĩa là toàn bộ cơ sở dữ liệu sẽ được khóa trong quá trình ghi. Đây thực sự là điều không mong muốn với các truy cập cơ sở dữ liệu lớn.
- Không yêu cầu các mức độ xác thực ứng dụng nào.

## Thực hiện việc ghi đa luồng hầu như cùng lúc.

Bây giờ tôi sẽ cố gắng đưa ra 1 vài thủ thuật nhỏ để thực hiện việc ghi cùng lúc.

## Lưu ý: SQLite không bao giờ cho phép bạn có thể ROW LEVEL LOCK. Vì vậy không cần phí thời gian tìm hiểu về nó .

Tất nhiên nó có thể ảnh hưởng đến hiệu năng nếu bạn thực hiện việc ghi trên phần lớn của bảng. Nhưng luồng thứ hai sẽ không đợi lâu khi luồng thứ nhất kết thúc.

## Đây là điểm chính của STEP 1.7.

Vì như bạn đã biết thì  chỉ mục B Tree cực kì nhanh . Và hoạt động viết của bạn sẽ dựa theo mỗi ROWID được index mặc định.

ví dụ:

delete from table where name='Fariz' //  nó sẽ khóa toàn bộ cơ sở dữ liệu chính trong 10s

Thay thế bằng:

insert into tepm.Lock select rowid,'processid' from table where name = 'Fariz'; // Nó chỉ khóa các sơ sở dữ liệu tạm thời liên quan 10s.

delete from table where ROWID in ( select rowid from tepm.Lock where name='fariz' and processid='XYZ' ) // lệnh xóa sẽ khóa file db trong 0.001 giây.

Tôi đã làm 1 bài kiểm tra nhỏ và kết quả thật tuyệt với tôi. Tôi có thể hoàn thành 2 hành động update trong 11s mà mỗi hành động mất 10 giây, cho bảng bao gồm 40 triệu dòng.

Hi vọng nó sẽ giúp ích cho những người sử dụng SQLite
