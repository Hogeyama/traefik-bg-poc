
# Session-aware Blue/Green Deployment with Traefik (PoC)

## 実験

### 事前準備

* traefikを起動する
  ```
  $ docker-compose up -d
  ```

### 初回起動

* アプリケーションサーバーv1を作成する
  ```
  $ ./build-sample-app v1
  $ docker image ls | grep v1
  app   v1  d99ca8eae615   2 hours ago  384MB
  ```
  * 中身はtomcat
* デプロイする
  ```
  $ ./traefik-bg --app app --deploy v1
  Generating /home/hogeyama/repo/traefik-bluegreen/apps/app/compose.yml
  Starting the service
  [+] Running 1/1
   ✔ Container app-v1-1  Started
  Update default traefik route
  Done
  ```
* curlでアクセスしてみる
  ```
  $ curl -ifsSL http://localhost/app/
  HTTP/1.1 200 OK
  Accept-Ranges: bytes
  Content-Length: 43
  Content-Type: text/html
  Date: Fri, 08 Nov 2024 16:54:26 GMT
  Etag: W/"43-1731079037207"
  Last-Modified: Fri, 08 Nov 2024 15:17:17 GMT
  Set-Cookie: app-v1=91d05f7e81f17d99; Path=/; Max-Age=60

  <html><body><h1>Tag: v1</h1></body></html>
  ```

### 更新

* アプリケーションサーバーv1を作成する
  ```
  $ ./build-sample-app v2
  ```
* デプロイする
  ```
  $ ./traefik-bg --app app --deploy v2
  Updating /home/hogeyama/repo/traefik-bluegreen/apps/app/compose.yml
  Starting the service
  [+] Running 1/1
   ✔ Container app-v2-1  Started
  Update default traefik route
  Done
  ```
* curlでアクセスしてみる
  ```
  $ curl -ifsSL http://localhost/app/
  HTTP/1.1 200 OK
  Accept-Ranges: bytes
  Content-Length: 43
  Content-Type: text/html
  Date: Fri, 08 Nov 2024 16:55:28 GMT
  Etag: W/"43-1731079673886"
  Last-Modified: Fri, 08 Nov 2024 15:27:53 GMT
  Set-Cookie: app-v2=f00cc81e7495315b; Path=/; Max-Age=60

  <html><body><h1>Tag: v2</h1></body></html>
  ```
  * v2が返ってくる
* v1のsticky cookieを使ってアクセスしてみる
  ```
  $ curl -ifsSL http://localhost/app/ -b 'app-v1=91d05f7e81f17d99'
  HTTP/1.1 200 OK
  Accept-Ranges: bytes
  Content-Length: 43
  Content-Type: text/html
  Date: Fri, 08 Nov 2024 16:56:41 GMT
  Etag: W/"43-1731079037207"
  Last-Modified: Fri, 08 Nov 2024 15:17:17 GMT

  <html><body><h1>Tag: v1</h1></body></html>
  ```
  * v1が返ってくる

### 古いバージョンの削除

* 古いバージョンを削除する
  ```
  $ ./traefik-bg --app app --undeploy-old
  Old containers: v1
  v1 is running but has no sessions. Undeploying
  [+] Stopping 1/1
   ✔ Container app-v1-1  Stopped
  Removing undeployed services from the compose file
  Done
  ```
  * セッションが残っている場合は削除されないようにしている
    * このPoCではTomcatの/manager/text/list APIでセッション数を確認している


