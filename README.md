Android Programing
----------------------------------------------------
### 2017.10.16 19일차

#### 예제
____________________________________________________

- [HttpConnection 예제](https://github.com/Hooooong/DAY25_JSONData)

#### 공부정리
____________________________________________________

##### __HTTPConnection__

- HTTP 란?

  > HTTP(__H__ yper __T__ ext __T__ ransfer __P__ rotocol)는 WWW 상에서 정보를 주고받을 수 있는 프로토콜이다. 주로 HTML 문서를 주고받는 데에 쓰인다. TCP와 UDP를 사용하며, 80번 포트를 사용한다. HTTP는 클라이언트와 서버 사이에 이루어지는 요청/응답(request/response) 프로토콜이다.<br>예를 들면, 클라이언트인 웹 브라우저가 HTTP를 통하여 서버로부터 웹페이지나 그림 정보를 요청하면, 서버는 이 요청에 응답하여 필요한 정보를 해당 사용자에게 전달하게 된다. 이 정보가 모니터와 같은 출력 장치를 통해 사용자에게 나타나는 것이다.
HTTP를 통해 전달되는 자료는 http:로 시작하는 URL(인터넷 주소)로 조회할 수 있다.

  - HTTP 은 웹 상에서 데이터를  주고받을 수 있는 프로토콜이다.

  - Android 에서는 HTTP 통신을 위해 `HttpURLConnection` 을 제공하지만 Thread 를 직접 다뤄야해서 복잡하다. Android는 좀 더 편리한 `AsyncTask` 를 제공한다.

  - 외부 Library 로는 `retrofit`, `volley`, `okHttp`, `Rx` 등이 있다.

- HTTPConnection 예제

  - 권한 설정이 필요하다. 다행이도 Runtime Permission 이 아니기 때문에 `<uses-permission android:name="android.permission.INTERNET"/>` 을 설정만 해주면 된다.

  - Android 는 내장 API 로 `HttpURLConnection` Class 를 제공한다.

  - __`HttpURLConnection` 을 사용할 경우에는 `Main Thread` 가 아닌, `Sub Thread` 에서 사용해야 한다.__

  - `HttpURLConnection` 사용방법

      1. URL 객체 선언 ( 웹 주소를 가지고 생성 )

      2. URL 객체에서 서버 연결을 해준다 -> `HttpURLConnection` 을 생성 ( Stream )

      3. Connection 방식을 선언 ( Default : GET )

      4. 연결되어 있는 Stream 을 통해서 Data 를 가져온다.

      5. 연결 Stream 을 닫는다.

      ```java
      class Network extends Thread{

          Handler handler;
          StringBuilder result = new StringBuilder();

          public Network(Handler handler) {
              this.handler = handler;
          }

          @Override
          public void run() {
              try {
                  // Network 처리
                  // 1. URL 객체 선언 ( 웹 주소를 가지고 생성 )
                  URL url = new URL("http://fastcampus.co.kr");
                  // 2. URL 객체에서 서버 연결을 해준다
                  HttpURLConnection urlConnection = (HttpURLConnection) url.openConnection();
                  // 3. Connection 방식을 선언 ( Default : GET )
                  urlConnection.setRequestMethod("GET");

                  // 통신이 성공적인지 체크
                  if (urlConnection.getResponseCode() == HttpURLConnection.HTTP_OK) {
                      // 4. 연결되어 있는 Stream 을 통해서 Data 를 가져온다.
                      // 여기서부터는 File 에서 Data 를 가져오는 방식과 동일
                      InputStreamReader isr = new InputStreamReader(urlConnection.getInputStream());
                      BufferedReader br = new BufferedReader(isr);

                      String temp = "'";
                      while ((temp = br.readLine()) != null) {
                          result.append(temp).append("\n");
                      }

                      // 5. 연결 Stream 을 닫는다.
                      br.close();
                      isr.close();
                  } else {
                      Log.e("ServerError", urlConnection.getResponseCode() + " , "  + urlConnection.getResponseMessage());
                  }
                  urlConnection.disconnect();
              }catch (Exception e){
                  Log.e("Error", e.toString());
              }

              // UI 를 변경하는 경우에는 Sub THread 에서는 변경이 불가능하기 때문에 3가지 방법을 사용한다.

              // 1. Handler 에 결과값을 Message.obj 에 담아 보낸다.
              sendMessage(result);

              // 2. Handler 에 빈 Message 를 보내 변경된 결과값을 사용한다.
              sendEmptyMessage(result);

              // 3. runOnUiThread 를 사용하여 결과값을 사용한다.
              runOnUiThread(new Runnable() {
                  @Override
                  public void run() {
                      textView.setText(result.toString());
                  }
              });
          }
      }

      public void sendMessage(StringBuilder result){
          Message msg = new Message();
          msg.what = ACTION_SEND;
          msg.obj = result;
          handler.sendMessage(msg);
      }

      public void sendEmptyMessage(StringBuilder result){
          data = result.toString();
          handler.sendEmptyMessage(999);
      }

      Handler handler = new Handler(){
           @Override
           public void handleMessage(Message msg) {
               switch (msg.what){
                   case ACTION_SEND:
                       // Message 에 담긴 obj 를 통해 TextView 를 변경한다.
                       textView.setText(msg.obj.toString());
                       break;
                   case 999:
                       // 변경된 결과값을 통해 TextView 를 변경한다.
                       textView.setText(data);
                       break;
               }
           }
       };
      ```

- 참조 : [HTTP](https://ko.wikipedia.org/wiki/HTTP), [HttpURLConnection API 명세서](https://developer.android.com/reference/java/net/HttpURLConnection.html), [HTTP 상태 코드](https://ko.wikipedia.org/wiki/HTTP_%EC%83%81%ED%83%9C_%EC%BD%94%EB%93%9C)

##### __AsyncTask__

- AsyncTask 란?

  ![AsyncTask](https://github.com/Hooooong/DAY25_HTTPConnect/blob/master/image/AsyncTask.png)

  > AsyncTask 란 Android 에서 요구하는 Main Thread 와 Sub Thread의 분리 구조를 보다 쉽게 구현하도록 도와주는 추상클래스이다. AsyncTask 의 핵심은 하나의 Sub Thread 를 실행하고, 5가지 재정의 함수를 Main Thread 와 Sub Thread 를 분리하여 실행한다.

- AsyncTask 설명

  ![AsyncTask Generic](https://github.com/Hooooong/DAY25_HTTPConnect/blob/master/image/Asynctask%20Generic.png)

  - AsyncTask 는 3가지 Generic Type 을 반드시 지정해야 한다.

      1. doInBackground() 의 parameter 로 사용
      2. onProgressUpdate() 의 parameter 로 사용, 주로 진행상태의 Percent 값 (Integer) 로 사용된다.
      3. doInBackground() 의 return 값이면서 onPostExecute() 의 parameter

  - 5가지의 재정의 함수가 있지만, 반드시 필요한 메소드는 `doInBackground()` 이고 부가적으로 중요한 메소드는 `onPreExecute()` 와 `onPostExecute()` 가 있다.

  메소드 | 설명
  :----: | :----:
  onPreExecute() | doInBackground() 메소드가 실행되기 전에 실행하는 메소드
  doInBackground() | 백그라운드(sub Thread) 에서 코드를 실행하는 메소드 ( 이 메소드만 SubThread )
  onProgressUpdate() | doInBackground() 메소드를 처리하는 도중에 publishProgress 메소드를 실행한다. <br> 그러면 호출이 일어날 때마다 onProgressUpdate() 메소드가 실행된다.
  onCancelled() | doInBackground() 메소드가 취소되면 실행하는 메소드
  onPostExecute() | doInBackground() 메소드가 실행된 후에 실행하는 메소드 <br> doPostExecute() 는 doInBackground() 로부터 데이터를 받을 수 있다.

- AsyncTask 예제

  - 위에 설명한 `HttpURLConnection` 을 좀 더 쉽게 구현할 수 있다.

  ```java
  AsyncTask<String, Integer, String> asyncTask = new AsyncTask<String, Integer, String>(){
      // 1번째 인자값 : doInBackground() 의 parameter 로 사용
      // 2번째 인자값 : onProgressUpdate() 의 parameter 로 사용
      //                주로 진행상태의 Percent 값 (Integer) 로 사용된다.
      // 3번째 인자값 : doInBackground() 의 return 값이면서 onPostExecute() 의 parameter
      @Override
      protected void onPreExecute() {
          progressBar.setVisibility(View.VISIBLE);
      }
      @Override
      protected String doInBackground(String... s) {
          String param1 = s[0];
          return Remote.getData(param1);
      }
      @Override
      protected void onPostExecute(String result) {
          progressBar.setVisibility(View.GONE);
          // 전체 html 코드 중에서
          // <title> </title> 안에 있는 내용만 출력하세요.
          int startNumber = result.indexOf("<title>");
          int lastNumber = result.indexOf("</title>");
          String title = result.substring(startNumber + "<title>".length(), lastNumber);
          textView.setText(title);
      }
  };

  // 실행 메소드
  // doInBackground 에 들어가는 parameter 를 정하는 메소드
  asyncTask.execute(url);
  ```
