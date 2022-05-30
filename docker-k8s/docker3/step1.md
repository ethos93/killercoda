Dockerfile을 생성하고 docker 이미지를 생성하는 실습을 진행합니다.

Dockerfile 생성전에 간단한 Java Application을 작성해 봅니다.

## Java Application
`touch HelloDocker.java`{{execute}} 를 통해 HelloDocker.java 파일을 생성하고
Editor 탭에서 아래 내용으로 파일을 완성합니다.

vi가 익숙하시면 vi를 사용하셔도 됩니다.
`vi HelloDocker.java`{{execute}}

```java
public class HelloDocker {
	public static void main(String[] args) {
		System.out.println("Hello Docker!!!");
	}
}
```

java로 실행시 "Hello Docker!!!"를 출력하고 종료되는 아주 간단한 Application 입니다.
원하신다면 Java code를 직접 수정해 보셔도 좋습니다.

## Dockerfile 생성
이제 앞서 만든 Java Application이 구동되는 docker 이미지를 만듭니다.

docker 이미지는 일반적으로 Dockerfile를 통해 build하여 만듭니다.
`touch Dockerfile`{{execute}} 를 통해 Dockerfile 파일을 생성하고
Editor 탭에서 아래 내용으로 파일을 완성합니다.

역시 vi가 익숙하시면 vi를 사용하셔도 됩니다.
`vi Dockerfile`{{execute}}

```Dockerfile
FROM openjdk:8
COPY HelloDocker.java /hello/
WORKDIR /hello
RUN javac HelloDocker.java
CMD ["java","HelloDocker"]
```

Dockerfile의 각 라인을 설명하자면,
1. openjdk8이 포함된 이미지를 생성하며
2. /hello 경로에 HelloDocker.java 파일을 복사하고
3. /hello 경로로 이동한 뒤
4. HelloDocker.java 를 컴파일하고
5. docker container가 구동되면 java HelloDocker 를 실행

Dockerfile을 잘 생성하였다면, 이제 이미지를 생성합니다.

## docker 이미지 생성
hellodocker이미지를 v1 tag를 붙여서 생성합니다.

`docker build -t hellodocker:v1 .`{{execute}}

출력되는 로그를 보시면, Base Image Pull 한 뒤에 Dockerfile에 명시된 대로, 진행됩니다.


## docker 이미지 확인
hellodocker 이미지가 정상적으로 생성 되었는지 확인합니다.

`docker images`{{execute}}

생성된 이미지가 잘 실행되는지도 확인합니다.

`docker run hellodocker:v1`{{execute}}
