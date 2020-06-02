## Item9: try-finally보다는 try-with-resources를 사용하라

자바 라이브러리에는 close 메서드를 직접 호출에 닫아줘야 하는 자원이 있다.
InputStream. OutputStream, java.sql.Connection 등이 있다.



### try-finally를 이용한 close() 호출

~~~
	static String firstLineOfFile(String path) throws IOException {
        BufferedReader br = new BufferedReader(new FileReader(path));
        try {
            return br.readLine();
        } finally {
            br.close();
        }
    }
~~~

위와 같은 방법은 매우 전통적인 방법이다. 하지만 자원이 둘 이상이면 try-finally 방식은 지저분해진다.

~~~
   static void copy(String src, String dst) throws IOException {
        InputStream in = new FileInputStream(src);
        try {
            OutputStream out = new FileOutputStream(dst);
            try {
                byte[] buf = new byte[BUFFER_SIZE];
                int n;
                while ((n = in.read(buf)) >= 0)
                    out.write(buf, 0, n);
            } finally {
                out.close();
            }
        } finally {
            in.close();
        }
    }
~~~

또한 readLine과 close 호출 양쪽에서 예외가 발생하면, close에서 발생한 예외는 숨겨지고 readLine에서 발생한 예외가 기록될 수 있다.(예외를 덮을수 있다.)



### 해결 방법: try-with-resources

~~~
    static void copy(String src, String dst) throws IOException {
        try (InputStream in = new FileInputStream(src);
             OutputStream out = new FileOutputStream(dst)) {
            byte[] buf = new byte[BUFFER_SIZE];
            int n;
            while ((n = in.read(buf)) >= 0)
                out.write(buf, 0, n);
        }
    }
~~~

try-with-resource는 try(...)에서 선언된 객체들에 대해 try가 종료될 때 자동으로 자원을 해제해 준다.
다만 try에서 선언된 객체가 AutoCloseable을 구현해야 try구문이 종료될 때 객체의 close() 메서드를 호출해준다.

try-with-resources를 사용하게 되면 코드는 더 짧고 분명해지며, 만들어지는 예외 정보도 훨씬 유용하다.
한마디로 정확하고 쉽게 자원을 회수할 수 있다.

꼭 회수해야 하는 자원을 다룰 때 try-with-resource를 사용하자.
