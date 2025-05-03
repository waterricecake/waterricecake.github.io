---
layout: post
title: "[개발] SSL 재발급 script 작성"
categories:
  - 개발
tags:
  - Mafia
  - 사이드
comments: true
---
# 1. 상황

`Cerbot` 을 사용하면서 스케쥴링으로 자동재발급이 계속 실패하여 수동으로 재발급을 해주고 있었슴다.

뭐가 문제이니 보니 자동으로 재발급을 할때 `Nginx`에 의해서 `80` 포트가 막혀 문제가 발생한것이었습니다.

그래서 재발급할때 `Nginx`를 꺼주고 다시 켜주는 것을 반복하고 있었습니다.

`Nginx` 설정을 통해 안꺼주는 방법이 있지만 이는 다음에 해결하고 우선 자동화 스크립트과 스케쥴러를 작성해 보겠습니다.

이것을 자동화 해주는 자동화 스크립트를 짜볼려고 합니다.

# 2. 재발급 스크립트

우선 재발급시 실행되는 과정은 다음과 같습니다.

1. 인증서 남은 기간 확인
	1. 기간이 30일 이상일 경우 종료
2. Nginx 종료
3. 재발급이 가능한지 테스트 (이는 재발급 횟수 제한이 있기 때문입니다)
	1. 가능할 경우 재발급
4. Nginx 재시작

위와 같은 과정으로 재발급 스크립트를 작성해보겠습니다.

## 2.1. 환경
```shell
#!/bin/bash
set -e

restart_nginx(){
        echo "Restart nginx"
        cd ~/nginx
        docker compose up -d
}
```
1. `#!/bin/bash`
   이는 bash 쉘을 사용하겠다는 의미입니다.
   `#`은 주석을 의미하지만 `#!`는 shebang이라고 하며 다음에 오는 명령어를 실행합니다.
2. `set -e`
   이는 error 발생시 종료하라는 의미입니다.
3. `restart_naginx()`
   nginx는 실패하더라도 항상떠있어야하기 때문에 항상 재시작을 해줘야합니다 따라서 함수로 재시작을 선언해주고 nginx shut down 전 trap 명령어를 통해 다시 실행을 해줄 것입니다.

## 2.2.  로깅시 날짜 입력

```shell
echo
echo "LOGING $(date '+%Y-%m-%d %H:%M:%S')"
```

## 2.3.  인증서 만료 날짜 확인
```shell
echo
echo "LOGING $(date '+%Y-%m-%d %H:%M:%S')"

valid_date=$(sudo certbot certificates | grep 'VALID' | grep -oP '(?<=VALID: )\d+')

if [ "$valid_date" -gt 30 ]; then
        echo "SSL is valid. valid date : $valid_date"
        exit 0
fi
```

1. `echo "LOGING $(date '+%Y-%m-%d %H:%M:%S')"`
   스크립트 실행 날짜를 출력합니다.
2. `valid_date=$(sudo certbot certificates | grep 'VALID' | grep -oP '(?<=VALID: )\d+')`
   인증서를 확인하고, VALID를 찾아 얼마나 남았는지 확인합니다.
3. `if`
   해당 인증서가 30일 보다 클경우 종료시킵니다.

## 2.4.  nginx 종료
```shell
trap restart_nginx EXIT
echo "Shutdown nginx"
docker rm -f nginx
```
1. `trap restart_nginx EXIT`
   에러로 인해 종료되도 Nginx를 재시작 하게 합니다.

## 2.5. Renew

```shell
echo "Running dry run"
if sudo certbot renew --dry-run; then
        echo "Renew ssl"
        sudo certbot renew
        exit 0
fi
echo "Dry run failed check"
exit 1
```

1. `if sudo certbot renew --dry-run`
   `--dry-run`을 통해 테스트를 합니다. 인증서 발급에 제한이 있으므로 가능한지 우선 테스트를 합니다.
   가능할 경우 재발급을 합니다

## 2.6. 전체 스크립트

```shell
#!/bin/bash
set -e

restart_nginx(){
        echo "Restart nginx"
        cd ~/nginx
        docker compose up -d
}

echo
echo "LOGING $(date '+%Y-%m-%d %H:%M:%S')"

valid_date=$(sudo certbot certificates | grep 'VALID' | grep -oP '(?<=VALID: )\d+')

if [ "$valid_date" -gt 30 ]; then
        echo "SSL is valid. valid date : $valid_date"
#        exit 0
fi

trap restart_nginx EXIT
echo "Shutdown nginx"
docker rm -f nginx

echo "Running dry run"
if sudo certbot renew --dry-run; then
        echo "Renew ssl"
        sudo certbot renew
        exit 0
fi
echo "Dry run failed check"
exit 1
```
# 3. 스케쥴러
이제 위 스크립트 특정시간마다 실행시키면 됩니다.

# 3.1. 기존 스케쥴러 삭제

`Certbot`에서 기본으로 제공하는 스케쥴러가 Systemd로 이루어져 있습니다.

```shell
~$ systemctl list-timers | grep certbot
Sat 2025-05-03 07:49:00 UTC      5h 43min Fri 2025-05-02 13:20:20 UTC      12h ago snap.certbot.renew.timer       snap.certbot.renew.service
```

위는 `Systemd` 중 `certbot`과 관련된 타이머를 확인해본 것입니다.

이때 위 명령은 각

```
(다음 실행 예정) (다음 실행 예정까지 남은 시간) (마지막 실행 시간) (마지막 실행 후 경과 시간) (타이머 유닛) (실행 서비스)
```

를 의미합니다.

 ```shell
 $ sudo systemctl stop snap.certbot.renew.timer
 $ sudo systemctl stop snap.certbot.renew.service
```

## 3.1. Cron

`Cron` 은 UNIX 기반의 OS 시간 스케쥴러입니다.

```
분 시 일 월 명령어
```

로 단순하게 입력할 수 있습니다.

설정 방법이 간단하지만, 로깅을 수동으로 해야하며, cron 자체의 상태나 의존성 설정 (네트워크 상태 등)을 할 수 없는 것이 단점입니다.

단순 반복 행위를 입력할때 좋습니다.

러닝 커브가 짧아 빠르게 설정하기 좋습니다.

### 3.1.1.  실습

`Crontab` 명령어를 통해 cron을 설정할 수 있습니다

```shell
sudo crontab -e
```

이러면 `nano`로 해당 파일이 열리며, 맨 밑줄에

```shell
0 14 * * * /home/{사용자명}/{스크립트위치}/ssl_renew.sh >> /home/{사용자명}/{스크립트위치}/log/certbot-check.log 2>&1
```

를 추가하면 됩니다.

이는 매일 14시 0분마다 위 작성한 `ssl_renew.sh`를 실행시키겠다는 의미이며

`/log/certbot-check.log`에 로그를 기록하겠다는 의미입니다.

또한 `ssl_renew.sh`의 권한을 설정해줘야하는데

```shell
chmod +x ssl_renew.sh
```

를 해주면 됩니다.
## 3.2. Systemd

`Systemd`는 Linux 운영 체제용 시스템 및 서비스 관리자입니다.

`Cron` 에 비해 정교하게 프로세스를 관리할 수 있으며 타이머를 통해 스케쥴러를 설정할 수 있습니다.

```shell
~$ systemctl list-timers | grep certbot
Sat 2025-05-03 07:49:00 UTC      5h 43min Fri 2025-05-02 13:20:20 UTC      12h ago snap.certbot.renew.timer       snap.certbot.renew.service
```

이때 `timer`는 언제 실행할 지를 정의하며 `service`는 무엇을 실행할 지 결정합니다.

## 3.2.1. 실습

#### 3.2.1.1. timer

```ini
[Unit]
Description=SSL timer

[Timer]
OnCalendar=*-*-* 15:00:00
Persistent=true

[Install]
WantedBy=timers.target
```

1. `[Unit]`
   설정 및 의존성을 정의합니다
	1. `Description`
	   설명입니다.
2. `[Timer]`
   어떤 것을 수행할지 정의합니다.
	1. `OnCalendar`
	   실행 날짜입니다.
	2. `Persistent`
	   서버가 꺼지는 등에 의한 이슈로 타이머가 동작안 할때 다시 재시작시 시간에 상관없이 한번 실행하는가를 의미합니다
#### 3.2.1.2.  service

```ini
[Unit]
Description=SSL Renewal Script
After=network.target

[Service]
Type=oneshot
ExecStart=/home/ubuntu/ssl_renew.sh
```

1. `[Unit]`
   설정 및 의존성을 정의합니다
	1. `Description`
	   설명입니다.
	2. `After`
	   의존성을 의미하며, 네트워크가 준비된 후 실행한다는 의미입니다.
2. `[Service]`
   어떤 것을 수행할지 정의합니다.
	1. `Type`
	   실행 방식입니다. `oneshot`은 한번만 실행한다는 의미입니다.
	2. `ExecStart`
	   실행할 스크립트 경로 및 스크립트입니다.

#### 3.2.1.3. 설정
이제 작성한 스크립트를 `Systemd` 가 확인하도록 하면 됩니다

```shell
sudo nano /etc/systemd/system/ssl-renew.service
sudo nano /etc/systemd/system/ssl-renew.timer
```

로 작성한 후 

```shell
sudo systemctl daemon-reexec
sudo systemctl daemon-reload
sudo systemctl enable --now ssl-renew.timer
```

을 통해 입력한 스크립트를 활성화 시킵니다.

이후 다음 명령어를 통해 확인할 수 있습니다.

 ```shell
 $ systemctl list-timers | grep ssl-renew
-                                   - Sat 2025-05-03 06:02:42 UTC     22ms ago ssl-renew.timer                ssl-renew.service
```


# 4.   고찰

확실히 `Systemd` 가 더 자세히 설정할 수 있어 좋지만, 매일 오후 2시에 돌리는 간단한 스케쥴이기 때문에 `Cron`을 선택하게 되었습니다.