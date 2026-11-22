# Netcatを試してみた

### Netcatとは
TCP/IPネットワークを通じてデータの送受信を行うコマンドラインツール。クライアン・サーバーどちらとしても動作することができる。

### TCP

- サーバー側
	- -l: Listen mode, for inbound connects
	- -v: Verbose
```bash
$ nc -lv 8080
```
- クライアント側
```bash
$ nc -v 127.0.0.1 8080
```

双方向通信が確立され、双方の端末に入力した文字列のやり取りができる。
TCPには接続終了の仕組みが存在するため、一方の端末で正常に接続を終了すると、もう一方の端末も終了を検知できる。今回のNetcatでは、双方のコマンドが終了する。

### UDP
- サーバー側
	- -u: UDP mode
```bash
$ nc -luv 8080
```
- クライアント側
```bash
$ nc -uv 127.0.0.1 8080
```
TCPと同じく双方向通信ができる。
UDPには接続の確立・終了の仕組みが存在しないため、一方の端末でコマンドを終了しても、もう一方の端末は受信待機を継続する。

### HTTP
- Pythonの標準機能で簡易的なWebサーバーを立ち上げる
```bash
$ python3 -m http.server 8080 --bind 127.0.0.1
```
- Curlでリクエストを送信する
```bash
$ curl 127.0.0.1:8080
```
- Netcatで正常なHTTPリクエストを送信する
```bash
$ printf 'GET / HTTP/1.1\r\nHost: localhost\r\n\r\n' | nc -v 127.0.0.1 8080
```
- 末尾の改行を含めずリクエストを送信する
	- ヘッダーの終了を示す `\r\n\r\n` が存在しないため、サーバー側は追加データの受信を待ち続け、レスポンスは返却されない。
```bash
$ printf 'GET / HTTP/1.1\r\nHost: localhost' | nc -v 127.0.0.1 8080
```
- 存在しないHTTPメソッドのリクエストを送信する
	- `501 Unsupported method` が返却される
```bash
printf 'HOGE / HTTP/1.1\r\nHost: localhost\r\n\r\n' | nc -v 127.0.0.1 8080
```
