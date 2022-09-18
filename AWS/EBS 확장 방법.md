# Overview

EC2 초기 구성 당시 프리티어 스펙으로 구축을 많이 하기 때문에 서버가 상용화가 되고, 소스가 점점 커지고 부수적인 기술들이 EC2 내에 추가가 되면서 문제가 종종 발생을 하게 된다. 따라서 서버 운용을 위해 내부 스펙을 변경해줘야 한다. EC2 서비스는 인스턴스 증설과 수정을 탄력적으로 제공한다.

먼저, EC2의 스펙에 대해서 알아보자.

### AMI

EC2는 AMI(Amazon Machine  Image)라고 하여 가장 기초적인 OS 셋팅이 된 이미지를 제공한다. 대부분의 개발이 리눅스/우분투의 기능으로도 충분히 구동이 가능하고 효율적이다. 초기 개발을 할 때는 AWS가 제공하는 AMI를 선택하고 이외의 기술들(nginx, git 등)을 구축하고나서 커스터마이징한 AMI를 생성하고 사용할 수 있다.

### 인스턴스 유형

시스템을 구성하는 스펙의 유형은 다양하다. 하지만 그 스펙들을 개별적으로 하나하나 선택하는 것은 비효율적이고 어렵고 각 스펙이 어울리지 못하다. 따라서 AWS는 CPU, 메모리, 스토리지 등을 조합을 맞춰사용을 한다.

![image](https://user-images.githubusercontent.com/66561524/190880139-0a902632-ca0f-4add-8ba9-e673861056ae.png)

[https://aws.amazon.com/ko/ec2/instance-types/?_fsi=CdDtKHOx](https://aws.amazon.com/ko/ec2/instance-types/?_fsi=CdDtKHOx)

### 스토리지

EC2 인스턴스는 직접 연결된 블록 디바이스 스토리지 형태인 인스토어 스토리지를 가지고 있다. EBS(Elastic Block Store)를 추가로 설정하여 환경을 구성한다. 인스토어 스토리지 볼륨은 인스턴스가 실행 된 이후에 변경할 수 없으며 EBS를 추가해줘야 한다.

추가적으로 AWS는 인스토어 스토리지를 임시 저장소의 용도로만 사용하기를 권장하고 있다.

[https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/Storage.html?icmpid=docs_ec2_console](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/Storage.html?icmpid=docs_ec2_console)

따라서 EC2의 CPU/메모리 등을 변경하고 싶은 경우에는 [인스턴스 유형 변경](https://aws.amazon.com/ko/premiumsupport/knowledge-center/resize-instance/)을 해주면 된다.

![image](https://user-images.githubusercontent.com/66561524/190880145-84125470-2b1f-488e-b145-2485de619374.png)

![image](https://user-images.githubusercontent.com/66561524/190880148-58ebee87-3751-4aac-b682-244a8d18350e.png)

우리는 EBS 전용 인스턴스 유형을 선택했기 때문에, 루트 스토리지도 EBS로 생성이 되어있다. 따라서 해당 인스턴스 안에서 `df -h` 로 확인할 수 있는 `/dev/xvda1 7.7G 7.5G 215M 98% /` 는 EBS 내부의 용량이라고 생각할 수 있다.

![image](https://user-images.githubusercontent.com/66561524/190880154-fa499b5b-82be-467c-9959-6d956ca661c7.png)

EBS 지원 인스턴스는 중지한 후 다시 시작해도 연결된 볼륨에 저장된 데이터는 영향을 받지 않는다. 따라서 인스턴스를 중지시키고 볼륨 관련 작업을 수행할 수 있는 것이다. 인스턴스의 관한 작업을 수행하거나 루트 볼륨을 다른 인스턴스에 연결할 수도 있다.

# EBS

EBS(Elastic Block Storage)는 EC2랑은 뗄레야 뗼 수가 없다.

- 저장 공간이 생성되어지며 EC2 인스턴스에 부착된다.
- 디스크 볼륨 위에 File System이 생성된다.
- EBS는 특정 Availability Zone에 생성된다.

**Availability Zone(AZ)**

![image](https://user-images.githubusercontent.com/66561524/190880245-4c4b84fe-e564-4aef-81a9-32741886fdb3.png)
1개의 Region안에 여러개의 AZ가 존재한다. 유사시 한 쪽 서버가 망가졌을 때 AZ 백업을 통해 한쪽 서버가 가능하게 해주는 Disaster Recovery를 해준다.

## EBS 볼륨 타입

<<SSD군>>

1) General Purpose SSD (GP2) : 최대 10K IOPS를 지원하며 1GB당 3IOPS 속도가 나옴

2) Provisioned IOPS SSD(I01) : 극도의 I/O률을 요구하는 (매우 큰 DB 관리) 환경에서 주로 사용된다. 10K 이상의 IOPS를 지원한다.

<<Magnetic/HDD군>>

3) Througthput Optimized HDD(ST1) : 빅데이터ㅓ Datawarehouse, Log 프로세싱시 주로 사용(boot volume으로 사용 가능 x)

4) CDD HDD (SC1) : 파일 서버와 같이 드문 volume 접근시 주로 사용, 역시 boot volume으로 사용 불가능하나 비용은 매우 저렴함

5) Magnetic (Standard) : 디스크 1GB당 가장 싼 비용을 자랑함. Boot volume으로 유일하게 가능함.


## 인스턴스 장애시 새로운 인스턴스에 EBS를 연결하는 방법
    
Amazon EBS 지원 인스턴스에서 장애가 발생할 경우 다음 방법 중 하나로 세션을 복원할 수 있습니다.

- 중지 후 다시 시작합니다(먼저 이 방법 시도).
- 모든 관련 볼륨의 스냅샷을 자동으로 생성하고 새 AMI를 생성합니다. 자세한 내용은 [Amazon EBS-backed Linux AMI 생성](https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/creating-an-ami-ebs.html) 섹션을 참조하세요.
- 다음 단계에 따라 볼륨에 새 인스턴스를 연결합니다.

    1. 루트 볼륨의 스냅샷을 생성합니다.
        2. 스냅샷을 사용하여 새 AMI를 등록합니다.
        3. 새 AMI에서 새 인스턴스를 시작합니다.
        4. 나머지 Amazon EBS 볼륨을 이전 인스턴스에서 분리합니다.
        5. Amazon EBS 볼륨을 새 인스턴스에 다시 연결합니다.
        
    
자세한 내용은 [Amazon EBS 볼륨](https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/ebs-volumes.html) 섹션을 참조하세요.
    
## 인스턴스의 루트 디바이스 유형을 확인하는 방법

**콘솔을 사용해 인스턴스의 루트 디바이스 유형을 확인하는 방법**

1. [https://console.aws.amazon.com/ec2/](https://console.aws.amazon.com/ec2/)에서 Amazon EC2 콘솔을 엽니다.
2. 탐색 창에서 **인스턴스**를 선택하고 인스턴스를 선택합니다.
3. **스토리지** 탭의 **루트 디바이스 세부 정보**에서 **루트 디바이스 유형**의 값을 다음과 같이 확인합니다.
    - 값이 `EBS`이면 Amazon EBS 지원 인스턴스입니다.
    - 값이 `INSTANCE-STORE`이면 인스턴스 스토어 지원 인스턴스입니다.

[https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/RootDeviceStorage.html](https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/RootDeviceStorage.html)

**볼륨 유형**

> Amazon EBS는 EC2 인스턴스의 수명 주기와는 별도로 유지되는 블록 수준의 스토리지 볼륨이므로 나중에 인스턴스를 중지했다가 다시 시작할 수 있습니다. 임시 인스턴스 스토어 볼륨은 호스트 컴퓨터에 물리적으로 연결됩니다. 인스턴스 스토어의 데이터는 인스턴스의 수명 주기 동안만 유지됩니다.

**스냅샷**

> 스냅샷은 S3에 저장된 EC2 볼륨의 백업입니다. 스냅샷의 ID를 입력하여 스냅샷에 저장된 데이터를 사용하여 새 볼륨을 만들 수 있습니다. 또는 [스냅샷] 필드에 텍스트를 입력하여 퍼블릭 스냅샷을 검색할 수 있습니다. 설명은 대/소문자를 구분합니다.

- Amazon EBS, Amazon EC2 instace store 추가 설명
    
    **Amazon EBS**
    
    Amazon EBS provides durable, block-level storage volumes that you can attach to a running instance. You can use Amazon EBS as a primary storage device for data that requires frequent and granular updates. For example, Amazon EBS is the recommended storage option when you run a database on an instance.
    
    An EBS volume behaves like a raw, unformatted, external block device that you can attach to a single instance. The volume persists independently from the running life of an instance. After an EBS volume is attached to an instance, you can use it like any other physical hard drive. As illustrated in the previous figure, multiple volumes can be attached to an instance. You can also detach an EBS volume from one instance and attach it to another instance. You can dynamically change the configuration of a volume attached to an instance. EBS volumes can also be created as encrypted volumes using the Amazon EBS encryption feature. For more information, see [Amazon EBS encryption](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSEncryption.html).
    
    To keep a backup copy of your data, you can create a *snapshot* of an EBS volume, which is stored in Amazon S3. You can create an EBS volume from a snapshot, and attach it to another instance. For more information, see [Amazon Elastic Block Store](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AmazonEBS.html).
    
    **Amazon EC2 instance store**
    
    Many instances can access storage from disks that are physically attached to the host computer. This disk storage is referred to as *instance store*. Instance store provides temporary block-level storage for instances. The data on an instance store volume persists only during the life of the associated instance; if you stop, hibernate, or terminate an instance, any data on instance store volumes is lost. For more information, see [Amazon EC2 instance store](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/InstanceStorage.html).
    

# EBS 볼륨 수정

진행 순서

1. [스냅샷 생성](https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/ebs-creating-snapshot.html)
2. [EBS 볼륨에 대한 수정 요청](https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/requesting-ebs-volume-modifications.html)
3. [EBS 볼륨 수정 진행 상황 모니터링](https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/monitoring-volume-modifications.html)
4. [볼륨 크기 조정 후 Linux 파일 시스템 확장](https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/recognize-expanded-volume-linux.html)

## 고려사항

- EBS 볼륨의 스냅샷을 생성할 때는 인스턴스를 중지한 후 스냅샷을 생성해야 한다.
- 최대 절전 모드를 비활성화해줘야 한다.
- 5개이상의 스냅샷을 동시에 진행하면 안된다.

## 스냅샷 생성

1. EC2 콘솔 
2. Elastic Block Store의 [스냅샷](https://ap-northeast-2.console.aws.amazon.com/ec2/v2/home?region=ap-northeast-2#Snapshots:sort=snapshotId)을 선택
3. 스냅샷 생성
4. 리소스 유형 선택에서 볼륨 선택

### 볼륨 수정 요청

1. EC2 콘솔
2. Elastic Blocks Store의 [볼륨](https://ap-northeast-2.console.aws.amazon.com/ec2/v2/home?region=ap-northeast-2#Volumes:sort=desc:size)을 선택
3. 해당 인스턴스의 볼륨 ID를 선택하고 작업 → 볼륨 수정
4. 볼륨 유형, 크기, IOPS, 처리량
5. 수정 완료

### 볼륨 수정 진행 상황 모니터링

1. EC2 콘솔
2. Elastic Blocks Store의 [볼륨](https://ap-northeast-2.console.aws.amazon.com/ec2/v2/home?region=ap-northeast-2#Volumes:sort=desc:size)을 선택
3. 볼륨 id 선택 → 세부정보 창 → 설명 → 상태

### 볼륨 크기 조정 후 Linux 파일 시스템 확장

1. 인스턴스 시작(탄력적 ip 설정 안했으면 새로운 ip 생성)
2. 인스턴스 접속
3. `df -h`

완료
