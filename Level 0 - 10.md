**Level 0 - 1**

SSH đến bandit.labs.overthewire.org port 2220 với username bandit0 và mật khẩu bandit0

	ssh bandit0@bandit.labs.overthewire.org -p 2220
<img width="1051" height="896" alt="image" src="https://github.com/user-attachments/assets/3f94be27-fd23-4273-b082-0b7aefbc323e" />

	ZjLjTmM6FvvyRnrb2rfNWOZOTa6ip5If

Đọc file Readme lấy được password cho level 1. Dùng exit hoặc logout để thoát và ssh vào với username bandit1

**Level 1 - 2**

Level này yêu cầu đọc file có tên `-` , mà khi dùng lệnh trực tiếp ví dụ `cat -` thì shell sẽ lấy dữ liệu từ sdtin và không đọc file. Bài này mình dùng absolute path dạng`./-` hay `/home/bandit1/-` sẽ đọc được pass

	263JGJPfgU6LtdEvgfWU1XP5yac29mFx

<img width="849" height="316" alt="image" src="https://github.com/user-attachments/assets/f6b44921-4037-4da4-b5c2-37c28d7f5a6b" />


**Level 2 - 3**

Level này cần đọc file tên  `--spaces in this filename--` . Có thể dùng absolute path như level trước hoặc dùng dấu `--` ở trước để shell hiểu là hết phần arguments.  Phần này tên file có khoảng trắng nên sẽ dùng dấu `' '` ngoài tên file hoặc dùng dấu `\`.

	MNk8KNH3Usiio41PRUEoDFPqfxLPlSmx

<img width="1038" height="220" alt="image" src="https://github.com/user-attachments/assets/866617d9-d309-45a9-ae92-c9a9634aef1f" />


**Level 3 - 4**

 Phần này file ẩn ở folder inhere, cd sang folder `inhere`, sau đó dùng ls -a để list tất cả các file bao gồm cả file ẩn `...Hiding-From-You`

	2WmrDFRmJIq3IPxneAaMGhap0pFhF3NJ

<img width="1046" height="363" alt="image" src="https://github.com/user-attachments/assets/8e9ec1a6-c547-4bc2-ab2b-16abf8428e71" />


Level 4 - 5

Level này file cũng nằm ở folder `inhere`, nhưng ở đây có nhiều file và file chứa password là file có chứa kí tự có thể đọc được(only human-readable), mình có thể đọc tất cả file bằng dấu `* (wildcard)`, và dùng lệnh `strings` để chỉ lấy kí tự có thể đọc được.

	4oQYVPkxZOOEOO5pTW81FB8j8lxXGUQw

<img width="1042" height="272" alt="image" src="https://github.com/user-attachments/assets/e80817dc-5481-4ee6-9866-350591cd5fdb" />


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
	
<img width="1050" height="113" alt="image" src="https://github.com/user-attachments/assets/bf413cfd-d67a-445f-b3fb-d28789a612c2" />
<img width="1052" height="115" alt="image" src="https://github.com/user-attachments/assets/ce657d67-d8d3-4fdd-b8e4-c6785c696a29" />

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

<img width="1064" height="153" alt="image" src="https://github.com/user-attachments/assets/c7b47e2c-7b45-4d52-b8e9-43b9491e138d" />

**Level 7 - 8**

Level này pass ở cạnh từ millionth trong file data.txt. Mình đơn giản dùng lệnh `grep` để tìm từ đó.

`grep millionth data.txt`

	 dfwvzFQi4mU0wfNbFOe9RoWskMLg7eEc
 
<img width="1053" height="260" alt="image" src="https://github.com/user-attachments/assets/31cfe862-69ce-4839-b2d8-6a61cd234891" />
 
**Level 8 - 9**

Level này pass cũng nằm ở file data.txt và là dòng chỉ xuất hiện 1 lần.  Có lệnh `uniq` sẽ lọc các dòng giống nhau và chỉ hiển thị mỗi dòng một lần, dùng thêm arg `-u` sẽ chỉ hiển thị mỗi dòng giống nhau đúng 1 lần( trước khi dùng uniq thì lưu ý phải dùng lệnh sort bởi vì uniq chỉ lọc được các dòng đã sắp xếp). Vậy mình dùng theo thứ tự cat - sort - uniq

`cat data.txt | sort | uniq -u`

	4CKMh1JI91bUIZZPXDqGanal4xvAg0JM

<img width="1056" height="234" alt="image" src="https://github.com/user-attachments/assets/6cc3a573-ab0b-4556-95b8-c4addb4b7900" />

**Level  9 - 10**

Pass ở level này nằm ở trong file data.txt là một trong những đoạn có thể đọc và phần trước nó là một vài dấu `=` . Mình dùng `strings` để lấy chuỗi có thể đọc và `grep` để lấy phần có dấu =

	FGUW5ilLVJrxX9kMYMmlN4MgbpfMiqey

<img width="975" height="738" alt="image" src="https://github.com/user-attachments/assets/b1d7021d-0aee-4bd1-8a09-2d58df122503" />

