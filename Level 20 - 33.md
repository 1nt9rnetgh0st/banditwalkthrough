**Level 20 - 21**
Level này có 1 chương trình của bandit21 đã setuid khi chạy để kết nối đến port ở arg và nhận dữ liệu, nếu đúng là pass của bandit20 thì sẽ trả về pass của bandit21.  Mình dùng một chương trình để nghe như `netcat` ở port bất kì (ngoài khoảng 1 - 1023 vì trong khoảng này chỉ cho root) và gửi pass của level này khi có chương trình kết nối đến(như suconnect)
```
bandit20@bandit:~$ ls
suconnect
bandit20@bandit:~$ (echo "0qXahG8ZjOVMN9Ghs7iOWsCfZyXOUbYO") | nc -l -p 9999 &
[1] 51
bandit20@bandit:~$ ./suconnect 9999
Read: 0qXahG8ZjOVMN9Ghs7iOWsCfZyXOUbYO
Password matches, sending next password
EeoULMCra2q0dSkYj561DX7s1CpBuOBt
```

**Level 21 - 22**
Ở level này có chương trình chứa pass được tự dộng chạy bằng `cron`. Mình mở `/etc/cron.d` để xem các cron thì thấy có file tên cronjob_bandit22 file này sẽ chạy mỗi khi reboot hoặc cách 1 phút. Khi đọc file script cronjob_bandit22.sh mà nó chạy mình thấy nó sẽ đọc mật khẩu và ghi vào một file tạm thời trong /tmp , vậy chỉ cần đọc file đó là lấy được pass.
```
bandit21@bandit:/etc/cron.d$ ls
behemoth4_cleanup  cronjob_bandit22  cronjob_bandit24  leviathan5_cleanup    
otw-tmp-dir        clean_tmp         cronjob_bandit23  e2scrub_all       manpage3_resetpw_job  sysstat
bandit21@bandit:/etc/cron.d$ cat cronjob_bandit22
@reboot bandit22 /usr/bin/cronjob_bandit22.sh &> /dev/null
* * * * * bandit22 /usr/bin/cronjob_bandit22.sh &> /dev/null
bandit21@bandit:/etc/cron.d$ cat /usr/bin/cronjob_bandit22.sh
#!/bin/bash
chmod 644 /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
cat /etc/bandit_pass/bandit22 > /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
bandit21@bandit:/etc/cron.d$ cat /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
tRae0UfB9v0UzbCdn9cY0gQnds9GF58Q
```

**Level 22 - 23**
Ở level này cũng tương tự. Mình thấy có script cronjob_bandit23.sh copy pass vào một file trong /tmp có tên dạng encrypt md5 của cụm từ "I am user $myname" và $myname ở đây là bandit23 
```
bandit22@bandit:~$ cd /etc/cron.d
bandit22@bandit:/etc/cron.d$ ls
behemoth4_cleanup  cronjob_bandit22  cronjob_bandit24  leviathan5_cleanup    
otw-tmp-dir        clean_tmp         cronjob_bandit23  e2scrub_all       manpage3_resetpw_job  sysstat
bandit22@bandit:/etc/cron.d$ cat cronjob_bandit23
@reboot bandit23 /usr/bin/cronjob_bandit23.sh  &> /dev/null
* * * * * bandit23 /usr/bin/cronjob_bandit23.sh  &> /dev/null
bandit22@bandit:/etc/cron.d$ cat /usr/bin/cronjob_bandit23.sh
#!/bin/bash

myname=$(whoami)
mytarget=$(echo I am user $myname | md5sum | cut -d ' ' -f 1)

echo "Copying passwordfile /etc/bandit_pass/$myname to /tmp/$mytarget"

cat /etc/bandit_pass/$myname > /tmp/$mytarget
bandit22@bandit:/etc/cron.d$ echo I am user bandit23 | md5sum | cut -d ' ' -f 1
8ca319486bfbbc3663ea0fbe81326349
bandit22@bandit:/etc/cron.d$ cat /tmp/8ca319486bfbbc3663ea0fbe81326349
0Zf11ioIjMVN551jX3CmStKLYqjk54Ga
```

**Level 23 - 24**
Cái này mình làm hơi mất thời gian tại viết nhầm script hay gì đấy nên chạy mãi không được:)). Đọc qua chương trình bandit24.sh thì đơn giản nếu nó thấy file owned bởi bandit23 thì nó sẽ thực thi trong tối đa 60 giây nó sẽ tự tắt, sau đó sẽ xoá file ấy. Ở đây mình tạo một script để lấy pass và lưu pass ở một file mới trong thư mục đó. Sau đó đợi cron chạy lệnh và lấy password

```
(run.sh)
cat /etc/bandit_pass/bandit24 > /var/spool/bandit24/foo/password
```

```
bandit23@bandit:~$ cd /var/spool/bandit24/foo
bandit23@bandit:/var/spool/bandit24/foo$ nano run.sh
bandit23@bandit:/var/spool/bandit24/foo$ chmod +x run.sh
bandit23@bandit:/var/spool/bandit24/foo$ cat run.sh (ở đây cat hiển thị không có
cat: run.sh: No such file or directory         file chứng tỏ nó đã được thực hiện)
bandit23@bandit:/var/spool/bandit24/foo$ cat password
gb8KRRCsshuZXI0tUuR6ypOFjiZbf3G8
```

**Level 24 - 25**
Level này cần gửi password của level 24 kèm mã pin 4 số cần bruteforce, để nhanh thì sẽ không đóng kết nối mà mở kết nối và gửi pass liên tục nhiều lần. Sau vài giây chatgpt, mình thấy có cách là gán file descriptor vào socket có sẵn của linux. Sau đó viết script để liên tục gửi và nhận khi thấy tìm được pass đúng
```
  (run.sh)
#!/bin/bash
list=$(seq -w 0 9999)
exec 3<>/dev/tcp/127.0.0.1/30002
read -u 3 res
echo $res
for d in $list; do
        echo "[+]TRYING:gb8KRRCsshuZXI0tUuR6ypOFjiZbf3G8 $d"
        echo "gb8KRRCsshuZXI0tUuR6ypOFjiZbf3G8 $d" >&3
        read -u 3 res
        echo $res
        if echo $res | grep -q -v "Wrong"; then
                echo "FOUND PASSWORD:"
                while read -t 1 -u 3 res; do
                        echo $res
                done
                break
        fi
done
```

```
Correct!
FOUND: Correct!
password of user bandit25 is iCi86ttT4KSNe1armKiwbQNmB3YJP3q4
```

**Level 25 - 27**


upsNCc7vzaRDx6oZC6GiR6ERwe1MowGB

**Level 27 - 28**
Dùng git clone với với port 2220 và lấy được repo. Pass nằm ở trong file Readme.
```

┌──(nice㉿catonnthemoon)-[~/repo]
└─$ git clone  ssh://bandit27-git@bandit.labs.overthewire.org:2220/home/bandit27-git/repo
Cloning into 'repo'...
...
┌──(nice㉿catonnthemoon)-[~]
└─$ cd repo

┌──(nice㉿catonnthemoon)-[~/repo]
└─$ cat README
The password to the next level is: Yz9IpL0sBcCeuG7m9uQFt8ZNpS4HZRcN
```

**Level 28 - 29**
Dùng git clone về như level trước. Mình xem log thì thấy có một commit 'add missing data'. show commit đấy để lấy pass
```

┌──(nice㉿catonnthemoon)-[~/repo]
└─$ git log
...
commit d0cf2ab7dd7ebc6075b59102a980155268f0fe8f
Author: Morla Porla <morla@overthewire.org>
Date:   Tue Oct 14 09:26:19 2025 +0000

    add missing data
...
┌──(nice㉿catonnthemoon)-[~/repo]
└─$ git show d0cf2ab7dd7ebc6075b59102a980155268f0fe8f
...
+- password: 4pT1t5DENaYuqnqvadYs1oE4QLCdjmJ7

```

**Level 29 - 30**
Sau khi tải repo về, dùng git rev-list --all để thấy tất cả các commits, check các commits ấy để lấy pass.
```
┌──(nice㉿catonnthemoon)-[~/repo/.git]
└─$ git grep password $(git rev-list --all)
...
eb435001e0eb0219ee071219d1f16f1cd3941a3b:README.md:- password: qp30ex3VLz5MDG1n91YowTv4Q8l7CDZL
...
```

**Level 30 - 31**
Level này nó nằm trong một refs khác. Đọc file packed-ref sẽ thấy được. show commit đó lên.
```
┌──(nice㉿catonnthemoon)-[~/repo/.git]
└─$ cat packed-refs
# pack-refs with: peeled fully-peeled sorted
62152a9969c62cb647406aa88c1b5376dcf58968 refs/remotes/origin/master
84368f3a7ee06ac993ed579e34b8bd144afad351 refs/tags/secret

┌──(nice㉿catonnthemoon)-[~/repo/.git]
└─$ git show 84368f3a7ee06ac993ed579e34b8bd144afad351
fb5S2xb7bRyFmAvQYQGEqsbhVyJqhnDy
```

**Level 31 - 32**
Level này cần push file key.txt với đúng nội dung lên server, nhưng ở đây có file .gitignore không cho phép file .txt lên. Để add lên mình dùng arg `-f` để force git

```
cat README.md
This time your task is to push a file to the remote repository.

Details:
    File name: key.txt
    Content: 'May I come in?'
    Branch: master


┌──(nice㉿catonnthemoon)-[~/repo]
└─$ echo "May I come in?" > key.txt

┌──(nice㉿catonnthemoon)-[~/repo]
└─$ git add -f key.txt

┌──(nice㉿catonnthemoon)-[~/repo]
└─$ git commit -m "add key file to server"
 ...
 1 file changed, 1 insertion(+)
 create mode 100644 key.txt

┌──(nice㉿catonnthemoon)-[~/repo]
└─$ git push
...
remote: ### Attempting to validate files... ####
remote:
remote: .oOo.oOo.oOo.oOo.oOo.oOo.oOo.oOo.oOo.oOo.
remote:
remote: Well done! Here is the password for the next level:
remote: 3O9RfhqyAlVBEZpVb6LYStshZoqoSx5K
remote:
remote: .oOo.oOo.oOo.oOo.oOo.oOo.oOo.oOo.oOo.oOo.
...
```

**Level 32 - 33**
Trong Level này thì tất cả các lệnh khi gõ vào shell sẽ bị chuyển sang uppercase, mình sẽ dùng biến $0 - biến này sẽ trả về arg[0] của process đang chạy,ví dụ chạy script`bash ./run.sh` thì sẽ trả về 'bash', tương tự khi gõ trong terminal sẽ trả về tên shell. Khi dùng $0 ở level này sẽ trả về `sh` - shell đang chạy proccess này. Sau đó dùng nhưng bình thường để đọc pass
```
WELCOME TO THE UPPERCASE SHELL
>> $0
$ ls
uppershell
$ cat /etc/bandit_pass/bandit33
tQdtbs5D5i2vJwkO8mEyYEyTL8izoeJ0
```

**Level 33 - 34**
Đến đây thì đã hoàn thành hết level bandit cho đến thời điểm hiện tại rồi hehe=)). Level 34 vẫn có thể login vào sẽ có file `Readme` thông báo chúc mừng đã hoàn thành game và kêu gọi mọi người có thể đóng góp ý tưởng cho level tiếp theo:))