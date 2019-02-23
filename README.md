# Apache & NGINX Client IP 확인 & Redirection to HTTPS

## Load Balancer 설계

* 프로토콜 - Terminated HTTPS / Port - 443

  

## 접속하는 Client IP 확인

  

###  Apache .conf 파일 오픈 후 내용 추가
```
$vi /etc/httpd/httpd.conf
```
  
```
RemoteIPHeader X-Forwarded-For
```
  

###  LogFormat으로 시작하는 행 변경

```
LogFormat "%a %l %u %t \"%r\" %>s %O \"%{Referer}i\" \"%{User-Agent}i\"" combined
```
  

### Apache 재시작 및 Access Log 확인
```
$systemctl restart httpd
```
  

### access log 확인
```
$tail -f /var/log/httpd/access_log
```
  

## SSL 인증서 발급 및 Load Balancer 설정

  

### openssl 다운로드 후 openssl.exe 관리자권한으로 실행

### Private Key(비밀번호 포함) 생성
```
$genrsa -des3 -out PRIVATE_KEY_NAME.pem 2048
```
  

### Private Key(비밀번호 미포함) 생성
```
$genrsa -out PRIVATE_KEY_NAME.key 2048
```
  

### 공개키 추출
```
$rsa -in private.key -pubout -out public.key
```
  

### CSR(인증요청서) 생성 및 추가 사항 입력
```
$req -new -key PRIVATE_KEY_NAME.key -out private.csr
```
  

### rootCA.key 생성
```
$genrsa -aes256 -out KEY_NAME.key 2048
```

### 사설 CSR 생성
```
$req -x509 -new -nodes -key KEY_NAME.key -days 3650 -out rootCA.pem
```
  

###  CRT 생성
```
$x509 -req -in private.csr -CA rootCA.pem -CAkey KEY_NAME.key -CAcreateserial -out private.crt -days 3650
```
  

  

## Apache HTTPS Redirection

  

### Apache .conf 설정파일 변경
```
$vi /etc/httpd/conf/httpd.conf
```
  

```
<Directory "/var/www/html">
	Options Indexes FollowSymLinks
	AllowOverride All
	Require all granted
</Directory>
  ```

### Apache 재시작 후 테스트

## NGINX https redirection

  

### nginx .conf 설정파일 변경
```
$vi /etc/nginx/nginx.conf
```
  
```
server {
	listen 80; server_name _;
	if ($http_x_forwarded_proto = 'http'){
		return 301 https://$host$request_uri;
	}
}
```
  

### nginx 재시작 후 테스트