### Apache와 Tomcat 연동 방법

Apache와 Tomcat을 연동하는 방법은 세 가지가 있다.



1. tomcat connector(mod_jk) - 가장 많이 쓰는 방법
2. mod_proxy
3. mod_proxy_ajp



#### mod_jk

- 장점
  - mod_jk를 많이 사용하므로 관련 자료가 많다.
  - JkMount 옵션을 이용하면 URL이나 컨텐츠별로 유연한 설정이 가능하다.(이미지는 Web Server, 서블릿은 Tomcat)
- 단점
  - 별도의 모듈을 설치해야 한다.
  - 설정이 어렵다.
  - 톰캣 전용이다.



#### mod_proxy, mod_proxy_ajp

- 장점
  - 별도 묘듈 설치가 필요 없고(Apache 기본 모듈) 설정이 간편하다.
  - 특정 WAS에 의존적이지 않으므로 모든 WAS에 적용 가능하다.
- 단점
  - URL 별 유연한 설정이 어렵다.(ProxyPassMatch 사용 필요)



#### mod_jk 사용

1. mod_jk download 및 컴파일

   - ```shell
     wget http://mirror.apache-kr.org/tomcat/tomcat-connectors/jk/tomcat-connectors-1.2.48-src.tar.gz
     tar zxvf tomcat-connectors-1.2.48-src.tar.gz
     cd tomcat-connectors-1.2.48-src/native
     ./configure --with-apxs=/usr/sbin/apxs
     make 
     make install
     ```

   - make install 후 /etc/httpd/modules/mod_jk.so 에 복사가 된다. 

2. Apache httpd 설정

   - httpd/conf/httpd.conf 파일에서 LoadMoudle를 검색하여 수정

     - ```text
       LoadModule jk_module modules/mod_jk.so
       ```

   - httpd/conf.d/mod_jk.conf 설정

     - ```text
       <IfModule mod_jk.c>
         # Where to find workers.properties
         JkWorkersFile conf/workers_jk.properties
          
         # Where to put jk shared memory
         JkShmFile run/mod_jk.shm
          
         # Where to put jk logs
         JkLogFile logs/mod_jk.log
          
         # Set the jk log level [debug/error/info]
         JkLogLevel info
          
         # Select the timestamp log format
         JkLogStampFormat "[%a %b %d %H:%M:%S %Y] "
        
         ## url pattern 에 따른 connector mapping
         ##JkMountFile conf/uriworkermap.properties
       </IfModule>
       ```

   - httpd/conf/workers_jk.properties  설정

     - ```text
       worker.list=tomcat1
       # port는 tomcat의 web.xml의 <Connector protocol="AJP/1.3" 옵션의 port 부분 확인하여 설정
       worker.tomcat1.port=8009 
       worker.tomcat1.host=localhost
       worker.tomcat1.type=ajp13
       worker.tomcat1.lbfactor=1
       ```

   - httpd/conf/uriworkermap.properties 에서 url mapping 설정

     - ```text
       /service1/*.do=tomcat1
       /service1/*.jsp=tomcat1
       /service2/*=tomcat1
        
       # png와 jpg 는 apache 가 처리
       !/service2/*.png=tomcat1
       !/service2/*.jpg=tomcat1
       ```

       - 제외 규칙을 사용하면 URI 규칙에서 제외를 정의하여 요청을 tomcat으로 전달할 수 있다. 제외 규칙이 일치하면 요청이 전달되지 않는다. 일반적으로 웹 서버에서 정적 컨텐츠를 제공하는 데 사용된다. 규칙에 '!'접미사가 붙으면 제외 규칙이다. (http://tomcat.apache.org/connectors-doc/reference/uriworkermap.html의 Exclusions and rule disabling 부분)
       - workers_jk.properties에서 여러대의 WAS를 설정하고 url별로 다르게 mapping 시킬 수 있다.

   > [로드벨런싱](https://taetaetae.github.io/2019/08/04/apache-load-balancing/)



#### mod_proxy 사용

reverse proxy 로 동작하는 모듈이다. 보안상 문제가 있을 수 있으므로 reverse proxy 에 대해서 숙지한 후에 설정하는 것을 권장한다. 



[포워드 프록시(forward proxy) 리버스 프록시(reverse proxy) 의 차이](https://www.lesstif.com/pages/viewpage.action?pageId=21430345)



1. mod_proxy.so 와 mod_proxy_http.so를 LoadModule 로 로딩

2. ApachePath/conf/httpd.conf 설정

   - ```text
     <VirtualHost *:80>
         ServerName servername.com
         ErrorLog logs/servername.com-error_log
         CustomLog logs/servername.com-access_log common
      
         # Put this in the main section of your configuration (or desired virtual host, if using Apache virtual hosts)
         ProxyRequests Off
         ProxyPreserveHost On
       
         <Proxy *>
             Order deny,allow
             Allow from all
         </Proxy>
          
         ## mywebapp 설정
         ProxyPass /mywebapp http://mywebappurl:port/mywebapp
         ProxyPassReverse /mywebapp http://mywebappurl:port/mywebapp
         <Location /mywebapp>
             Order allow,deny
             Allow from all
         </Location>
     </VirtualHost>
     ```

     - ProxyRequests On 일 경우 Forward Proxy 로 동작하며 Off 일 경우 Reverse Proxy 이다.

     - ProxyPreserveHost On 일 경우 HTTP 요청 헤더중 Host: 부분을 유지한다.

     - 특정 url은 reverse proxy로 동작하지 않아야 할 경우 ProxyPassMatch로 처리 가능

       - ```text
          # /mywebapp/bar 하위의 URL 은 접근 금지
          ProxyPassMatch ^/mywebapp/bar[^.]+ !
          ProxyPass /mywebapp http://mywebappurl:port/mywebapp
          ProxyPassReverse /mywebapp http://mywebappurl:port/mywebapp
          <Location /mywebapp>
              Order allow,deny
              Allow from all
          </Location>
         ```

     - ssl일 경우 proxy 기능이 꺼져있다. ssl일때도 mod_proxy 사용하려면 호스트 설정에 다음 내용을 추가해야 한다.

       - ```text
         <VirtualHost _default_:443>
             SSLProxyEngine on
         </VirtualHost>
         ```



#### mod_proxy_ajp 사용

AJP13 protocol을 사용해서 reverse proxy 로 동작하는 방식이다. http reverse proxy 와 비슷하지만 protocol 을 ajp 로 적어주면 된다.

```text
ProxyPass /app ajp://mywebappurl.com:port/mywebapp
```

