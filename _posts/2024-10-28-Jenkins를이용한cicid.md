---
layout: post
title: Jenkins와 CI/CD
categories: DevOps
---

젠킨스를 이용하여 간단한 CI/CD 파이프라인을 구현해보자.

Ubuntu, Jenkins, Docker, Github 를 이용하여 로컬에서 Github로 push하면 Jenkins가 해당 commit을 감지한다.

Jenkins에선 해당 내용을 빌드하고 DockerHub로 push 한다. 또한 Ubuntu 서버에서 해당 도커 이미지를 pull & run 하여 서버를 실행한다.

### 코드 작성

단순하게 SpringBoot의 RestController에 루트 url로 요청을 하면 "Hello Jenkins"라는 글자가 나오는 API를 구현한다.

포트 번호는 80 이다.

이 API로 빌드 & 배포가 된 결과를 확인해보자

```java

@RestController
public class HelloController {

    @GetMapping
    public String hello() {
        return "Hello Jenkins";
    }
}

```

![alt text](/public/img/241028/image-3.png)

도커로 배포할 것이기 때문에 도커파일도 작성해둔다.

```Dockerfile

FROM openjdk:17-alpine
ARG JAR_FILE=/build/libs/jenkins-sample.jar
COPY ${JAR_FILE} jenkins-sample.jar
ENTRYPOINT ["java","-jar", "/jenkins-sample.jar"]

```



### Jenkins 설치

Jenkins를 도커로 설치해보자.

```bash

docker run -d -p 8080:8080 --name jenkins jenkins/jenkins:lts

```

도커로 젠킨스 컨테이너를 실행하고 http://localhost:8080 주소로 들어가면 Jenkins 설치 창이 나온다.


![alt text](/public/img/241028/image-2.png)

여기서 초기 패스워드는 젠킨스 서버를 띄우는 로그에서 확인할 수 있다.

아니면 /var/lib/jenkins/secrets/initialAdminPassword 경로로 가서 확인할 수도 있다.

```bash

docker logs jenkins

```

![alt text](/public/img/241028/image-4.png)

이 초기 비밀번호를 젠킨스 설치창에 입력하자.

이후 install suggested plugins을 클릭하면 설치가 진행된다.

![alt text](/public/img/241028/image-5.png)


설치가 완료되면 계정, 암호 설정을 진행하면 대시보드가 나타난다.

![alt text](/public/img/241028/image-6.png)


### 깃허브 설정

깃허브에서 public repository를 하나 생성해서 만들어 놓은 스프링부트 프로젝트를 push 하자.

![alt text](/public/img/241028/image-7.png)

다음으론 젠킨스에서 깃허브에 접근할 수 있게 깃허브 토큰을 하나 발급하자.

Settings -> Developer settings -> Personal access tokens -> Tokens (classic) -> Generate new token (classic) 

위의 순서로 발급하면 된다. 권한은 repo를 체크하자.

![alt text](/public/img/241028/image-8.png)


### 젠킨스 토큰 설정

방금 발급한 토큰을 젠킨스 Credential에 등록하자.

Jenins 관리 -> Credentials -> Add credentials를 클릭하면 추가할 수 있다.

![alt text](/public/img/241028/image-9.png)

여기서 Username에 Github id를 입력하고 password에 토큰 값을 입력한다.

![alt text](/public/img/241028/image-10.png)


### 깃허브 웹훅 설정

이제 깃허브에 push 이벤트가 발생했을 때 젠킨스에서 감지할 수 있도록 webhook 설정을 해보자.

깃허브 레파지토리의 Settings의 Webhook 탭에서 Add Webhook을 클릭하자.

Payload URL에 Jenkins 서버의 주소를 입력해주자.

### SSH 설정

Jenkins에서 빌드한 결과물을 Ubuntu 서버에서 실행하기 위해 Jenkins 서버와 Ubuntu 서버를 ssh를 이용하여 연결해보자.

우선 Jenkins plugin에서 Publish Over SSH 설치하자.

설치가 완료되면 Jenkins 관리 -> System 에 들어가면 Publish Over SSH 탭이 생성된 것을 확인할 수 있다.

![alt text](/public/img/241028/image-12.png)

SSH 연결을 하려면 젠킨스 서버에서 생성한 ssh 키를 배포하고자 하는 Ubuntu 서버에 복사하는 작업을 해야 한다.

[SSH 접속하기](블록그포스팅적기)

SSH 키를 복사했다면 Jenkins에 돌아와 확인해보자

![alt text](/public/img/241028/image-13.png)

내용을 입력하고 Test Configuration 을 클릭했을 때 Success 가 나오면 성공이다.


### 젠킨스 파이프라인 생성

젠킨스 파이프라인을 작성해보자.

프리스타일 프로젝트, 파이프라인 프로젝트 등을 선택할 수 있는데, 파이프라인 프로젝트가 좀 더 세밀하게 제어할 수 있어서 선호하는 편이다.

![alt text](/public/img/241028/image-11.png)

Github project에 레파지토리의 url을 적고 Build Trigger 탭에서 GitHub hook trigger for GITScm polling을 체크한다.

Pipeline 탭에서는 아래와 같이 작성한다.

```jenkinsfile

pipeline {
    agent any

    stages {
        stage('git') {
            steps {
                git branch: 'master', credentialsId: 'test', url: 'https://github.com/Ghosttrio/jenkins-sample.git'
            }
        }
        
        stage('build') {
            steps {
                sh 'chmod +x gradlew'

                sh './gradlew clean build'

                sh '''
                    docker build -t ghosttrio/jenkins-sample .
                    
                    docker push ghosttrio/jenkins-sample:latest
                '''
            }
        }
        
        stage('deploy') {
            steps {
                sshPublisher(publishers: [sshPublisherDesc(configName: 'test-ubuntu', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: '''
                        docker pull ghosttrio/jenkins-sample:latest

                        docker ps -q --filter name=jenkins-sample | grep -q . && docker rm -f $(docker ps -aq --filter name=jenkins-sample)

                        docker run -d -p 80:80 ghosttrio/jenkins-sample:latest

                    ''', execTimeout: 100000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '', remoteDirectorySDF: false, removePrefix: '', sourceFiles: '**/*.jar')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
            }
        }
   	}
}

```

각 단계를 설명하자면, 

git stage에서는 github에 변경된 내용을 감지해서 Jenkins 서버로 가져온다.

build stage에서는 gradle 권한을 주고 빌드 후 dockerhub로 빌드한 이미지를 push 한다.

deploy stage에서는 Publish Over SSH 에서 설정한 configName과 execCommand에 이미 올라가 있는 docker 컨테이너를 삭제함과 동시에 새로운 이미지를 run 한다.


### 테스트

현재는 루트 url에 요청을 보내면 Hello Jenkins 라는 글자가 출력되는 형태다.

해당 메시지를 Hello CICD로 수정 후 PUSH를 하자.

직후 Jenkins에서 해당 push를 감지하면서 빌드 & 배포를 시작한다.

![alt text](/public/img/241028/image-14.png)

Jenkins 파이프라인 작업이 성공한 것을 확인한 후 서버에 접속해보면 Hello CICD로 바뀐 것을 확인할 수 있다.

![alt text](/public/img/241028/image-15.png)