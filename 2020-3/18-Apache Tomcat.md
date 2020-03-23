### Apache Tomcat

- Apache Tomcat은 아파치 소프트웨어 재단(Apache Software Foundation)에서 개발한 서블릿 컨테이너가 있는 Web Application Server이다.
- 톰캣은 웹 서버와 연동하여 실행할 수 있는 자바 환경을 제공하며 자바 서버 페이지(JSP)와 자바 서블릿이 실행할 수 있는 환경을 제공하고 있다.



#### Tomcat Directory 구조

- bin - tomcat 실행에 필요한 서버동장 제어 스크립트 및 실행파일 (binary) 
- common -웹어플리케이션에서 공통적으로 사용하는 클래스 파일
- conf - 서버 전체 설정파일 폴더 (server.xml 등 )
- logs - 예외 발생 사항 등의 로그 저장
- server - 서버에서 사용하는 클래스 라이브러리 
- temp - 임시 저장용 폴더 
- webapps - 웹 어플리케이션 루트 폴더 
- work - jsp 파일을 서블릿 형태로 변환한 java 파일과 class파일 저장
