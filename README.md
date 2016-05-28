#Talker를 C9 및 AWS ElasticBeanstalk으로 배포 하기

Talker 프로그램은 간단하게 말을 가르치면 그대로 응답하는 '모바일 심심이 앱' 프로그램입니다. 본 프로그램은 [3기 멋쟁이 사자처럼 교육 예제](https://github.com/likelioncorp/talker)로 만들어진 것입니다. 본 과정을 통해 이 프로젝트를 C9 및 AWS ElasticBeanstalk에 배포해 보겠습니다.

## C9에 배포하기 
- Create a new workspace를 선택한 후, Clone from Git or Mercurial URL에서 https://github.com/channy/talker.git 를 입력한 후, Ruby 프로젝트를 생성합니다. 
- 먼저 ▶ Run Project를 하면, 기본적인 에러가 나오게 됩니다. 에러는 크게 세 가지로 해결가능합니다. 
 - 프로그램에사 사용하는 필요한 Gem을 설치하기 위해, 먼저 bash셀에서 `$ bundle install`을 실행 후, 필수 라이브러리를 설치합니다. (이 때 시간이 꽤 걸릴 수 있음)
 - 본 프로젝트는 SQLite Database를 사용합니다. `$ bundle exec rake db:migrate`를 실행하여 DB를 초기화 합니다. 
 - 이제, `config/secrets.yml`에 있는 몇 가지 환경 변수를 설정합니다. Ruby on Rails 탭의 가장 오른쪽에 ENV라는 부분을 클릭하여, 필요한 환경 변수를 추가 합니다. Name에 `SECRET_KEY_BASE` Value에는 아무 알파벳이나 넣으면 됩니다. (세션에 대한 암호화를 담당합니다.) 여기서 사용하지는 않지만, AWS Access Key나 Secrets 등은 절대로 코드에 삽입하지 않고, 이처럼 환경 변수로 지정합니다. (보안 키 값, 접속 암호, DB 암호 같은 것을 소스코드에 하드 코딩하는 것은 매우 나쁜 코딩 습관입니다.)
- 자 이제! 다시 한번 ▶ Run Project를 하고 https://my-project-id.c9users.io를 선택하시면 Talker를 실행할 수 있습니다.
- 좀 더 자세한 것인 기존 Rails 코드를 열심히 읽어보고, 공부해보시기 바랍니다.

## AWS Elastic Beanstalk에 배포하기 
AWS Elastic Beanstalk는 EC2 가상 머신을 직접 띄운 후, 리눅스 쉘로 들어가 Ruby 및 관련 프로그램를 설치하고, 앱을 배포하는 복잡한 과정 없이 자동으로 모든 것을 설정해 주는 간편한 자동 배포 서비스입니다. 여러분은 간단히 앱을 ZIP파일로 묶어 배포하기만 하면 됩니다. 기존의 C9으로 만들거나 Github에 올린 Talker 프로젝트를 AWS Elastic Beanstalk으로 배포를 해보겠습니다.

### EB로 Ruby on Rails 서버 자동 설치하기 
- AWS 관리 콘솔에 로그인 하여, 왼쪽 상단 Compute의 [Elastic Beanstalk](https://ap-northeast-2.console.aws.amazon.com/elasticbeanstalk/home?region=ap-northeast-2)을 선택합니다.
- 오른쪽 상단의 'Create New Application'을 선택합니다.
- Application Name과 Description을 간단히 적습니다.
- Create Web Server를 선택합니다.
- Predefined configuration: 에서 "Ruby"를 선택합니다. Environment type: 에서 꼭 "Single Instance"를 선택합니다. (Ruby on Rails 서버 1대만 사용합니다.)
- Application에서는 우선 Sample Application을 선택합니다. (나중에 Talker 코드를 Zip으로 압축해서 올릴 수 있습니다.)
- Environment name에 원하는 URL 앞 주소를 적습니다. 자신의 아이디를 넣어 Unique하게 만든 후, Check availability를 통해 사용 가능한지 알아봅니다. 
- Additional Resources는 그냥 NEXT를 누릅니다. (이 프로젝트는 관계형 DB(RDS)나 VPC를 사용하지 않습니다.)
- Configuration Details을 설정합니다.Instance type은 무료 제공 서버인 t2.micro를 선택합니다. (다른 인스턴스 타입을 선택하면 과금 될 수 있으니 유의!) 나머지는 기본 설정 그대로 두고 NEXT를 누릅니다.
- Environment Tags는 그냥 NEXT를 누릅니다.
- Permissions에서는 aws-elasticbeanstalk-ec2role 등을 Create Role을 눌러 생성합니다. (그냥 NEXT만 누르면 자동으로 만들어집니다.)
- 마지막으로 Launch를 누르면, 서비스 환경이 자동으로 설정됩니다. (약 2-5분의 시간이 걸립니다.)
- 설치가 완료되면, 조금전 설정했던 URL (xxx.region.elasticbeanstalk.com 형식)으로 접근해 보면 샘플 애플리케이션이 구동되는 것을 확인할 수 있습니다. 

### EB로 Talker 프로그램 배포하기 
- Talker 프로그램을 [ZIP 파일  다운로드](https://github.com/channy/talker/archive/master.zip)를 받습니다. 로컬에 있는 프로젝트를 ZIP으로 압축하거나, C9에 있는 프로젝트를 다운로드(tar.gz) 받아 ZIP으로 다시 압축 해도 됩니다. 
- 대시보드의 Upload and Deploy에서 생성한 프로젝트 ZIP 파일을 업로드 하면, 자동으로 Gem 파일 생성 및 배포를 진행합니다.
- 이제 C9에서 처럼, `config/secrets.yml`에 있는 환경 변수를 설정합니다. 왼쪽 메뉴의 Configuration을 선택하고, 박스 중 Software Configuration을 선택합니다. 가장 하단의 Environment Properties에서 Property Name에 `SECRET_KEY_BASE` Property Value에는 아무 알파벳이나 넣으면 됩니다. 
- 자! 이제 잠깐 기다리시면 Talker앱과  환경 변수가 업데이트 됩니다. 완료 후, 다시 서비스 URL에 접속하면 Talker를 만나실 수 있습니다. 

AWS Elastic Beanstalk은 한대의 서버 뿐만 아니라 Load Balacing 및 Auto Scaling 뿐만 아니라 RDS(데이터베이스 서비스)를 제공할 뿐만 아니라, 앱 배포 역시 손쉽게 제공할 수 있습니다. 

이제 AWS EB를 통해 멋쟁이 사자처럼 학생팀들이 더 빠르고 유연하게 대체할 수 있습니다. 
