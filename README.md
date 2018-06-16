# SQlite
1 vài bất lợi

As we mentioned SQLite is single thread. So it means SQLite can do one write operation at the same time. 
As we mentioned SQLite keep data [base] in one file. So it means whole database locks during write operation. This very unwanted for huge and intensive access database. 
No application level authentication required.
Make Multi thread write almost same time.
Như chúng tôi đã nói thì SQLite hoạt động đơn luồng. Có nghĩa là SQlite chỉ thực hiện 1 hoạt động tại 1 thời điểm nhất định. Chúng tôi cũng đã nói SQLite giữ dữ liệu gốc trong 1 file. Có nghĩa là toàn bộ cơ sở dữ liệu sẽ được khóa trong quá trình ghi. Đây thực sự là điều không mong muốn với các truy cập cơ sở dữ liệu lớn và 

Now I will try to show small trick to make write operation in almost same time.

note: SQLite never allow you to achieve ROW LEVEL LOCK. So no need to waste time for search about it.



Of course it can impact performance if you do write operation on big part of table. But second thread will not wait a lot of time for first operation end. 

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
