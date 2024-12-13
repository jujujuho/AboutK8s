Docker
=========

컨테이너란?
--------

코드와 종속성을 패키지화 하여 다른 PC에서도 안정적으로 실행하게함 --> Runtime Env

1. 기존의 하이퍼바이저식 가상화 방법이나, Vm보다 속도가 빠름
2. 독립적인 환경으로 다른 것에 영향을 받지 않음

Image란?
-------

코드나 라이브러리, 종속성, 도구 등 필요한 기타 파일을 가지는 파일 ( 템플릿 ) 
스냅샷이라고도 불리며, 수정이 불가함.

DockerFile ----build----> Image ----create---> Container 


![image](https://github.com/user-attachments/assets/f71e4222-b704-4249-aa04-6573e02a5487)
