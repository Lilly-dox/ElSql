# Bai ka tuoi tre
```
@app.get('/')
def home():
	if request.args.get('file'):
		filename = join("./static", request.args.get('file'))
		if isfile(normpath(filename)) and access(normpath(filename), R_OK) and (stat(normpath(filename)).st_size < 1024 * 1024 * 2):
			try:
				with open(normpath(filename), "rb") as file:
					if not regex.search(r'^(([ -~])+.)+([(^~\'!*<>:;,?"*|%)]+)|([^\x00-\x7F]+)(([ -~])+.)+$', filename, timeout=2) and "flag" not in filename:
						return file.read(1024 * 1024 * 2)
			except:
				pass
	return redirect("/?file=index.html")
```
Chức năng là GET / với parameter là `file` 

```
filename = join("./static", request.args.get('file'))
```
Nối chuỗi với thư mục ./static 
```
if isfile(normpath(filename)) and access(normpath(filename), R_OK) and (stat(normpath(filename)).st_size < 1024 * 1024 * 2):
			try:
				with open(normpath(filename), "rb") as file:
with open(normpath(filename), "rb") as file:
					if not regex.search(r'^(([ -~])+.)+([(^~\'!*<>:;,?"*|%)]+)|([^\x00-\x7F]+)(([ -~])+.)+$', filename, timeout=2) and "flag" not in filename:
						return file.read(1024 * 1024 * 2)
			except:
				pass
	return redirect("/?file=index.html")
```
 Kiểm tra xem file đó có tồn tại hay không. Nếu tồn tạo thì sẽ open file
 -> Kiểm tra tên file với regex ```r'^(([ -~])+.)+([(^~\'!*<>:;,?"*|%)]+)|([^\x00-\x7F]+```  ```timeout=2```:
timeout là tham số giới hạn thời gian cho phép việc tìm kiếm diễn ra. Nếu việc tìm kiếm mất nhiều thời gian hơn giá trị timeout (trong trường hợp này là 2 giây), nó sẽ dừng lại và gây ra một ngoại lệ re.error.

Trang web dính lỗi path travesal
![Screenshot 2024-05-21 134024](https://github.com/Lilly-dox/ElSql/assets/130746941/727e86d8-21e0-4da3-adae-85ac89c236d3)

![Screenshot 2024-05-21 142440](https://github.com/Lilly-dox/ElSql/assets/130746941/88ba8184-f015-4c69-b099-c8f1345c2e31)


Tuy nhiên khi tiếp tục xử lí challenge theo hướng bypass regex để khai thác lỗi hổng nhưng không phát hiện được thêm gì hữu ích

Hint hướng giải quyết : 
Tiến trình mở file ```with open(normpath(filename), "rb") as file: ```
Trong hệ điều hành Linux, mỗi tiến trình khi được khởi tạo sẽ được hệ thống cấp ba luồng I/O chuẩn, được biết đến là: stdin (standard input), stdout (standard output), và stderr (standard error). Mỗi luồng này được liên kết với một file descriptor (FD) cụ thể:

- stdin thường được liên kết với FD 0, nơi tiến trình đọc đầu vào.
- stdout thường được liên kết với FD 1, nơi tiến trình ghi đầu ra.
- stderr thường được liên kết với FD 2, nơi tiến trình ghi thông báo lỗi.

Khi tiến trình mở một file khác, hệ thống sẽ cấp cho nó một file descriptor mới, bắt đầu từ số nguyên nhỏ nhất chưa được sử dụng (thường là 3 trở lên vì 0, 1, và 2 đã được sử dụng cho stdin, stdout, và stderr).

![Screenshot 2024-05-21 232538](https://github.com/Lilly-dox/ElSql/assets/130746941/5878be33-328c-4ac6-aad5-0c3b0d009f68)
Các symlink này trỏ đến các file thực tế hoặc tài nguyên hệ thống khác mà tiến trình đang truy cập:
- Mỗi symlink trong /proc/{pid}/fd/ có tên là số của file descriptor.
- Click vào symlink sẽ đưa bạn đến tài nguyên mà FD đang thao tác.

Ví dụ, nếu một tiến trình có một file descriptor số 4 mở một file có đường dẫn /home/app/flag.txt, thì trong thư mục /proc/{pid}/fd/, sẽ có một symlink 4 trỏ tới /home/app/flag.txt

Thì tương tự khi send request trỏ đến flag.txt, tức là tiến trình đang truy cập đến flag.txt . Thì chúng ta sẽ có 1 symlink tương ứng dẫn đến tài nguyên flag.txt đang thao tác

Vấn đề ở đây là chúng ta phải tìm đúng thư mục chứa symlink dẫn đến flag.txt

Chúng ta sẽ cố gắng điền tên file dài nhất có thể để có thể đạt gần tới timeout 2s để có thể đồng thời race condition để đọc được file /flag.txt được trỏ đến ở trong fd/ được tạo ra trong tiến trình này.

![Screenshot 2024-05-21 232930](https://github.com/Lilly-dox/ElSql/assets/130746941/1f39cbae-3bfb-44ab-b149-9d5b270af454)

Sử dụng Burp Intruder để brute force với {id} và {num}  /proc/{id}/fd/{num} từ 1 đến 10 .
![Screenshot 2024-05-21 232903](https://github.com/Lilly-dox/ElSql/assets/130746941/2feb29a3-22b6-4f37-9f5a-276332a5cddb)

Và đã thành công dò được folder chứa symlink dẫn đến flag.txt .
![Screenshot 2024-05-21 232821](https://github.com/Lilly-dox/ElSql/assets/130746941/72678da0-382d-474a-94a9-b4a3bbfc916e)



