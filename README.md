1 vài bất lợi

Như chúng tôi đã nói thì SQLite hoạt động đơn luồng. Có nghĩa là SQlite chỉ thực hiện 1 hoạt động tại 1 thời điểm nhất định. Chúng tôi cũng đã nói SQLite giữ dữ liệu gốc trong 1 file. Có nghĩa là toàn bộ cơ sở dữ liệu sẽ được khóa trong quá trình ghi. Đây thực sự là điều không mong muốn với các truy cập cơ sở dữ liệu lớn.

Không yêu cầu các mức độ xác thực ứng dụng nào.

Thực hiện việc ghi đa luồng hầu như cùng lúc.

Now I will try to show small trick to make write operation in almost same time.
Bây giờ tôi sẽ cố gắng đưa ra 1 vài thủ thuật nhỏ để thực hiện việc ghi cùng lúc.

Lưu ý: SQLite không bao giờ cho phép bạn có thể ROW LEVEL LOCK. Vì vậy không cần phí thời gian tìm hiểu về nó .




Of course it can impact performance if you do write operation on big part of table. But second thread will not wait a lot of time for first operation end. 
Tất nhiên nó có thể ảnh hưởng đến hiệu suất nếu bạn thực hiện lệnh viết trên phần lớn dữ liệu trong bảng/ Nhưng luồng thứ 2 sẽ không đợi lâu cho đến khi luồng thứ nhất kết thúc.

here is main point STEP 1.7.

Because as you know B TREE index is super fast. And your WRITE operation will do according for each ROWID which is indexed by default.

for exaample:

delete from table where name='Fariz' // it will lock whole main database for 10 sec.

REPLACE WITH:

insert into tepm.Lock select rowid,'processid' from table where name = 'Fariz'; // it will lock only temporary attached database for 10 sec.

delete from table where ROWID in ( select rowid from tepm.Lock where name='fariz' and processid='XYZ' ) // delete will lock DB file for 0.001 second.

I have done small test and result is great for me. I was able to finish 2 same update operation in 11 second which is each transaction takes 10 sec. for table which include is 40 million row.

Hopefully it will help for SQLite users.

LikeMake multi thread write operation on SQLite almost same time / Almost row level lock performance.
