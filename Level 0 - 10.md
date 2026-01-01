**Level 0 - 1**

SSH đến bandit.labs.overthewire.org port 2220 với username bandit0 và mật khẩu bandit0

	ssh bandit0@bandit.labs.overthewire.org -p 2220

![[Pasted image 20251228203246.png]]

	ZjLjTmM6FvvyRnrb2rfNWOZOTa6ip5If

Đọc file Readme lấy được password cho level 1. Dùng exit hoặc logout để thoát và ssh vào với username bandit1

**Level 1 - 2**

Level này yêu cầu đọc file có tên `-` , mà khi dùng lệnh trực tiếp ví dụ `cat -` thì shell sẽ lấy dữ liệu từ sdtin và không đọc file. Bài này mình dùng absolute path dạng`./-` hay `/home/bandit1/-` sẽ đọc được pass

	263JGJPfgU6LtdEvgfWU1XP5yac29mFx

![[Pasted image 20251228211014.png]]

**Level 2 - 3**

Level này cần đọc file tên  `--spaces in this filename--` . Có thể dùng absolute path như level trước hoặc dùng dấu `--` ở trước để shell hiểu là hết phần arguments.  Phần này tên file có khoảng trắng nên sẽ dùng dấu `' '` ngoài tên file hoặc dùng dấu `\`.

	MNk8KNH3Usiio41PRUEoDFPqfxLPlSmx

![[Pasted image 20251228213607.png]]

**Level 3 - 4**

 Phần này file ẩn ở folder inhere, cd sang folder `inhere`, sau đó dùng ls -a để list tất cả các file bao gồm cả file ẩn `...Hiding-From-You`

	2WmrDFRmJIq3IPxneAaMGhap0pFhF3NJ

![[Pasted image 20251228214056.png]]

Level 4 - 5

Level này file cũng nằm ở folder `inhere`, nhưng ở đây có nhiều file và file chứa password là file có chứa kí tự có thể đọc được(only human-readable), mình có thể đọc tất cả file bằng dấu `* (wildcard)`, và dùng lệnh `strings` để chỉ lấy kí tự có thể đọc được.

	4oQYVPkxZOOEOO5pTW81FB8j8lxXGUQw

![[Pasted image 20251228215308.png]]

**Level 5 - 6**

Phần level này cần tìm một file trong thư mục `inhere` có đủ 3 tính chất:
	- có thể đọc được
	- size 1033 bytes
	- không thực thi được
Mình dùng lệnh find với các args tương tự:
	-readable (có thể đọc), 
	-size 1033c (file nặng 1033 bytes), 
	!-executable (dùng ! trước executable để lọc file không thực thi được )

`find . -readable -size 1033c ! -executable`

	HWasnPhtq9AVKe0dmk45nxy20cvUa6EG
	
![[Pasted image 20251228222851.png]]
![[Pasted image 20251228222942.png]]

**Level 6 - 7**

Level này yêu cầu tìm một file trong toàn bộ server mà có các tính chất:
	- owned by user bandit7
	- owned by group bandit6
	- 33 bytes in size
mình sử dụng các args tương tự:
	 -user bandit7
	 -group bandit 6
	 -size 33c

`find / -user bandit7 -group bandit6 -size 33c`
Muốn cho kết quả gọn và sạch hơn (Vì find sẽ đi qua lần lượt từng folder để tìm nên ta sẽ thấy nhiều thông báo lỗi do những folder đó ta không có quyền truy cập), ta có thể redirect các stderr đến `/dev/null`
`find / -user bandit7 -group bandit6 -size 33c 2> /dev/null`

	morbNTDkSW6jIlUc0ymOdMaLnOlFVAaj

![[Pasted image 20251228224423.png]]

**Level 7 - 8**

Level này pass ở cạnh từ millionth trong file data.txt. Mình đơn giản dùng lệnh `grep` để tìm từ đó.

`grep millionth data.txt`

	 dfwvzFQi4mU0wfNbFOe9RoWskMLg7eEc
 
![[Pasted image 20251228224954.png]]
 
**Level 8 - 9**

Level này pass cũng nằm ở file data.txt và là dòng chỉ xuất hiện 1 lần.  Có lệnh `uniq` sẽ lọc các dòng giống nhau và chỉ hiển thị mỗi dòng một lần, dùng thêm arg `-u` sẽ chỉ hiển thị mỗi dòng giống nhau đúng 1 lần( trước khi dùng uniq thì lưu ý phải dùng lệnh sort bởi vì uniq chỉ lọc được các dòng đã sắp xếp). Vậy mình dùng theo thứ tự cat - sort - uniq

`cat data.txt | sort | uniq -u`

	4CKMh1JI91bUIZZPXDqGanal4xvAg0JM

![[Pasted image 20251228230326.png]]

**Level  9 - 10**

Pass ở level này nằm ở trong file data.txt là một trong những đoạn có thể đọc và phần trước nó là một vài dấu `=` . Mình dùng `strings` để lấy chuỗi có thể đọc và `grep` để lấy phần có dấu =

	FGUW5ilLVJrxX9kMYMmlN4MgbpfMiqey

![[Pasted image 20251228231827.png]]

