**Level 10 - 11**
Phần level này chứa pass đã mã hoá base64 trong file data.txt. Để giải mã dùng lệnh `base64` với args `-d` để decode
```
bandit10@bandit:~$ base64 -d data.txt
The password is dtR173fZKb0RRsDFSGsg2RWnpNVj3qRr
```

**Level 11 - 12**
Pass ở level này đã được dịch chuyển 13 kí tự, đó cách mã hoá [ROT13](https://vi.wikipedia.org/wiki/ROT13). Để đưa lại về dạng ban đầu ta có thể dùng lệnh `tr` 
```
bandit11@bandit:~$ cat data.txt | tr [a-z][A-Z] [n-za-m][N-ZA-M]
The password is 7x16WNeHIi5YkIhWsfFIqoognUTyj9Q4
```

**Level 12 - 13**
Ở level này thì file đã bị chuyển sang hex và nén nhiều lần. Phần đây mình chuyển file về nhị phân bằng `xxd`, sau đó dùng các lệnh như `file` để biết file đang được nén ở dạng nào và đổi tên file với extension phù hợp, rồi dùng các tool giải nén tương ứng như `gunzip` ,`bzip2`, `tar` 
```
bandit12@bandit:~$ mktemp -d
/tmp/tmp.KxPQpaFg4b
bandit12@bandit:~$ xxd -r data.txt > /tmp/tmp.KxPQpaFg4b/data
bandit12@bandit:~$ cd  /tmp/tmp.KxPQpaFg4b/
bandit12@bandit:/tmp/tmp.KxPQpaFg4b$ file data
data: gzip compressed data, was "data2.bin", last modified: Tue Oct 14 09:26:00 2025, max compression, from Unix, original size modulo 2^32 572
bandit12@bandit:/tmp/tmp.KxPQpaFg4b$ file data
data: gzip compressed data, was "data2.bin", last modified: Tue Oct 14 09:26:00 2025, max compression, from Unix, original size modulo 2^32 572
bandit12@bandit:/tmp/tmp.KxPQpaFg4b$ mv data data.gz
bandit12@bandit:/tmp/tmp.KxPQpaFg4b$ gunzip data.gz
bandit12@bandit:/tmp/tmp.KxPQpaFg4b$ file data
data: bzip2 compressed data, block size = 900k
bandit12@bandit:/tmp/tmp.KxPQpaFg4b$ mv data data.bz2
bandit12@bandit:/tmp/tmp.KxPQpaFg4b$ bzip2 -d data.bz2
bandit12@bandit:/tmp/tmp.KxPQpaFg4b$ file data
data: gzip compressed data, was "data4.bin", last modified: Tue Oct 14 09:26:00 2025, max compression, from Unix, original size modulo 2^32 20480
bandit12@bandit:/tmp/tmp.KxPQpaFg4b$ mv data data.gz
bandit12@bandit:/tmp/tmp.KxPQpaFg4b$ gunzip data.gz
bandit12@bandit:/tmp/tmp.KxPQpaFg4b$ file data
data: POSIX tar archive (GNU)
bandit12@bandit:/tmp/tmp.KxPQpaFg4b$ mv data data.tar
bandit12@bandit:/tmp/tmp.KxPQpaFg4b$ tar -xf data.tar
bandit12@bandit:/tmp/tmp.KxPQpaFg4b$ ls
data5.bin  data.tar
bandit12@bandit:/tmp/tmp.KxPQpaFg4b$ file data5.bin
data5.bin: POSIX tar archive (GNU)
bandit12@bandit:/tmp/tmp.KxPQpaFg4b$ mv data5.bin data.tar
bandit12@bandit:/tmp/tmp.KxPQpaFg4b$ tar -xf data.tar
bandit12@bandit:/tmp/tmp.KxPQpaFg4b$ ls
data6.bin  data.tar
bandit12@bandit:/tmp/tmp.KxPQpaFg4b$ mv data6.bin data.tar
bandit12@bandit:/tmp/tmp.KxPQpaFg4b$ file data.tar
data.tar: bzip2 compressed data, block size = 900k
bandit12@bandit:/tmp/tmp.KxPQpaFg4b$ mv data.tar data.bzip2
bandit12@bandit:/tmp/tmp.KxPQpaFg4b$ bzip2 -d data.bz2
bandit12@bandit:/tmp/tmp.KxPQpaFg4b$ file data
data: POSIX tar archive (GNU)
bandit12@bandit:/tmp/tmp.KxPQpaFg4b$ mv data data.tar
bandit12@bandit:/tmp/tmp.KxPQpaFg4b$ tar -xf data.tar
bandit12@bandit:/tmp/tmp.KxPQpaFg4b$ ls
data8.bin  data.tar
bandit12@bandit:/tmp/tmp.KxPQpaFg4b$ file data8.bin
data8.bin: gzip compressed data, was "data9.bin", last modified: Tue Oct 14 09:26:00 2025, max compression, from Unix, original size modulo 2^32 49
bandit12@bandit:/tmp/tmp.KxPQpaFg4b$ mv data8.bin data.gz
bandit12@bandit:/tmp/tmp.KxPQpaFg4b$ gunzip data.gz
bandit12@bandit:/tmp/tmp.KxPQpaFg4b$ file data
data: ASCII text
bandit12@bandit:/tmp/tmp.KxPQpaFg4b$ cat data
The password is FO5dwFsc0cbaIiH0h8J2eUks2vdTDwAn
```

Sau khi làm xong thủ công mình có thử viết bash script để giải nén cho nhanh:)
```
#!/bin/bash
while true; do
        file=$(ls data*)
        ext=$(file $file)
        if echo $ext | grep -q "gzip" ; then
                mv $file "$file.gz"
                gunzip "$file.gz"
        elif echo $ext | grep -q "bzip2"; then
                mv $file "$file.bz2"
                bzip2 -d "$file.bz2"
        elif echo $ext | grep -q "tar"; then
                mv $file "$file.tar"
                tar -xf "$file.tar"
                rm "$file.tar"
        else
                echo "PROCESS COMPLETED:"
                cat $file
                break
        fi
done
```

**Level 13 - 14**
Level này không cho pass nhưng cho ssh key để đăng nhập vào level sau, mình dùng `scp` để download file key về (phần này cũng có thể mở, copy nội dung key và tạo file mới luôn:)). Lưu ý là ssh key phải là quyền 700, dùng` chmod` để đổi quyền. Sau đó`ssh` vào bandit14 với args -i cho sshkey
```
nice㉿catonnthemoon)-[~]
└─$ sudo scp -P 2220 \ bandit13@bandit.labs.overthewire.org:/home/bandit13/sshkey.private .

nice㉿catonnthemoon)-[~]
└─$ sudo chmod 700 ssh.private

nice㉿catonnthemoon)-[~]
└─$ sudo ssh bandit14@bandit.labs.overthewire.org -p 2220 -i sshkey.private
```

**Level 14 - 15**
Trong level này cần lấy pass của chính level này rồi gửi lên port 30000 ở localhost. Mình lấy pass ở chỗ mà bandit vẫn thường lưu cho từng level là /etc/bandit_pass/bandit14. Sau đó dùng `nc` để kết nối đến 127.0.0.1(localhost) ở port 30000 và nhập mật khẩu vừa tìm được
```
bandit14@bandit:~$ cat /etc/bandit_pass/bandit14
MU4VWeTyJk8ROof1qqmcBPaLh7lDCPvS

bandit14@bandit:~$ nc 127.0.0.1 30000
MU4VWeTyJk8ROof1qqmcBPaLh7lDCPvS
Correct!
8xCjnmgoKbGLhHFAZlGE5Tmu4M2tKJQo
```

**Level 15 - 16**
Ở level này mình dùng `openssl` `s_client` để tạo kết nối ssl đến 127.0.0.1 ở port 30001 và nhập pass của level này
```
bandit15@bandit:~$ openssl s_client -connect 127.0.0.1:30001
CONNECTED(00000003)
...
8xCjnmgoKbGLhHFAZlGE5Tmu4M2tKJQo
Correct!
kSkvUpMQ7lBYyCM4GBPvCvT1BfWRy0Dx
```

**Level 16 - 17**
Level này phải tìm port trong localhost có service lấy mật khẩu qua ssl trong khoảng 31000 - 32000. mình thử dùng `nmap` để scan ports. 
```
nmap -p 31000-32000 -sV 127.0.0.1 (hoặc) nmap -p 3100-32000 -sV --script ssl-enum-ciphers 127.0.0.1
```

```
bandit16@bandit:~$ nmap -sV -p 31000-32000 127.0.0.1
...
PORT      STATE SERVICE     VERSION
31046/tcp open  echo
31518/tcp open  ssl/echo
31691/tcp open  echo
31790/tcp open  ssl/unknown
31960/tcp open  echo

1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port31790-TCP:V=7.94SVN%T=SSL%I=7%D=12/29%Time=69521D50%P=x86_64-pc-lin
SF:ux-gnu%r(GenericLines,32,"Wrong!\x20Please\x20enter\x20the\x20correct...
```
Sau khi đã scan xong thì mình thấy có 5 port mở trong đó có 2 port chạy service ssl, nhưng ở dưới có fingerprint dạng "Wrong password..." ở port 31790 nên mình có thể chắc đây là port cần tìm. 
Ở đây khi mình dùng lệnh `openssl s_client 127.0.0.1:31790` như lần trước sẽ liên tục bị hiện thông báo debug `KEYUPDATE` ra stdout nên không thể thấy pass trả về của service. Để tránh bị thông báo debug, theo mình research thì dùng arg -quiet để bịt miệng thông báo đó lại và nhập pass như level trước.

```
bandit16@bandit:~$ openssl s_client -quiet -connect 127.0.0.1:31790
...
kSkvUpMQ7lBYyCM4GBPvCvT1BfWRy0Dx
Correct!
-----BEGIN RSA PRIVATE KEY-----
MIIEogIBAAKCAQEAvmOkuifmMg6HL2YPIOjon6iWfbp7c3jx34YkYWqUH57SUdyJ
...
vBgsyi/sN3RqRBcGU40fOoZyfAMT8s1m/uYv52O6IgeuZ/ujbjY=
-----END RSA PRIVATE KEY-----
```
 Mình exit sau đó lưu lại vào một file rồi đổi quyền thành 700 cho file này.
(Ngoài ra còn có vài tool cũng rất hay)

`ncat --ssl`
```
bandit16@bandit:~$ ncat --ssl 127.0.0.1 31790
kSkvUpMQ7lBYyCM4GBPvCvT1BfWRy0Dx
Correct!
-----BEGIN RSA PRIVATE KEY-----
MIIEogIBAAKCAQEAvmOkuifmMg6HL2YPIOjon6iWfbp7c3jx34YkYWqUH57SUdyJ
...
```

`socat - OPENSSL:host:port,verify 0`
```
bandit16@bandit:~$ socat - OPENSSL:127.0.0.1:31790,verify=0
2025/12/29 16:32:36 socat[21] W refusing to set empty SNI host name
kSkvUpMQ7lBYyCM4GBPvCvT1BfWRy0Dx
Correct!
-----BEGIN RSA PRIVATE KEY-----
MIIEogIBAAKCAQEAvmOkuifmMg6HL2YPIOjon6iWfbp7c3jx34YkYWqUH57SUdyJ
...
```

**Level 17 - 18**
Level này có 2 file là password.new và password.old. Mật khẩu nằm ở file password.new, ở dòng khác với file password.old. MÌnh dùng lệnh `diff` để so sánh file và lấy dòng khác nhau

```
bandit17@bandit:~$ ls
passwords.new  passwords.old
bandit17@bandit:~$ diff passwords.new passwords.old
42c42
< x2gLTTjFwMOhQ8oWNbMN362QKxfRqGlO (pass cho bandit 18)
---
> BMIOFKM7CRSLI97voLp3TD80NAq5exxk
```

**Level 18 - 19**
`.bashrc` là một file của bash shell, nó sẽ tự dộng chạy khi mở phiên interactive shell(vd. mở terminal mới, khi login ssh,..). Để đăng nhập mình dùng `bash --norc` để server chạy bash shell mà không đọc file `.bashrc`
```
┌──(nice㉿catonnthemoon)-[~]
└─$ sudo ssh bandit18@bandit.labs.overthewire.org -p 2220 -t "bash --norc"
$ cat readme
cGWpMaKXVwDUNgPAVJbWYuGHVn9zl3j8
```

Hoặc mình có một số cách khác:
```
┌──(nice㉿catonnthemoon)-[~]
└─$ sudo ssh bandit18@bandit.labs.overthewire.org -p 2220 -t "/bin/sh"

┌──(nice㉿catonnthemoon)-[~]
└─$ sudo ssh bandit18@bandit.labs.overthewire.org -p 2220 "cat readme"

┌──(nice㉿catonnthemoon)-[~]
└─$ scp -P 2220 bandit18@bandit.labs.overthewire.org:/readme .
```

**Level 19 - 20**
Trong bài có nói có chương trình để chạy lệnh với quyền user khác (tương tự sudo) - ở đây là chạy lệnh với quyền bandit20. Mình chạy chương trình này để lấy pass của bandit20
```
bandit19@bandit:~$ ls
bandit20-do
bandit19@bandit:~$ ./bandit20-do
Run a command as another user.
  Example: ./bandit20-do whoami
bandit19@bandit:~$ ./bandit20-do whoami
bandit20
bandit19@bandit:~$ ./bandit20-do cat /etc/bandit_pass/bandit20
0qXahG8ZjOVMN9Ghs7iOWsCfZyXOUbYO
```