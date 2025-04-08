---
layout: post
title: "[개발] FSD란"
categories:
  - 개발
  - 학습
tags:
  - frontend
comments: true
---
# 0. 서론

데이터 베이스에 대한 추가적인 개발을 하고 싶어져 조그마한 토이프로젝트를 시작하게 되었습니다.

이 과정에서 또 팀원 모으고 기획하고 하면 오래 걸릴것 같아 차라리 혼자 해보기로 하였습니다.

그래서 호기롭게 혼자 프론트 개발을 하게 되었는데....

기존 백엔드 개발에서 쓰던 폴더 구조, MVC 구조 처럼 레이어 별로 팩키지를 나누거나 DDD에서 어그리게이트 별로 팩키지를 나누면 되겠지 하고 나누었습니다.

후에 아는 프론트 개발자분이 보시더니 나중에 이거 한번 구조 나눠보세요 라고 하더라고요... 나름 나눈다고 했는데.

그래서 이참에 프론트엔드에서는 어떻게 프로젝트 구조를 가지고 있는지 알아보면 좋겠다 싶어서 학습하게 되었습니다.

# 1. FSD 란

FSD란 Feature-Sliced Design 의 약자로 기능 분할 설계를 의미합니다.

FSD의 목적은 기능 별로 나누어 코드를 관리하는 것을 목표로 합니다.

크게 **Layer**, **Slice**, **Segments**로 나누어서 관리합니다.

## 1.1. Layer

```tree
└── src
    ├── app/
    ├── proccesses/ (deorecated)
    ├── pages/
    ├── widgets/
    ├── features/
    ├── entities/
    └── shared/
```


레이어는 최상위 디렉토리입니다. 최대 7개로 제한되어 있으며, 일부는 선택사항이지만, 표준화되어 있는 구조입니다.

각 레이어는 책임과 영역이 존재합니다.

또한 상위, 하위의 개념이 있어 하위 레이어는 상위 레이어를 참조하지 못합니다. 이를 통해 순환 참조를 방지합니다.


| 순서  | 레이어       | 역할                                | 예시                       | 비고       |
| --- | --------- | --------------------------------- | ------------------------ | -------- |
| 1   | app       | 로직 초기화, 프로바이더, 전역 스타일, 타입 선언등     | App, Router, Providers   |          |
| 2   | processes | 여러 페이지에 걸친 프로세스 처리를 담당            |                          | optional |
| 3   | pages     | 어플리케이션 페이지                        | Home, About,             |          |
| 4   | widgets   | 페이지에 사용되는 UI 컴포넌트                 | Navbar, header, layouts  |          |
| 5   | features  | 비지니스 로직 기능 단위                     | Card, Review             | optional |
| 6   | entities  | 비지니스 주제를 나타냄                      | users, comments          | optional |
| 7   | shared    | 특정한 비지니스 로직에 속하지 않는 유틸, common 객체 | UI kit, Utils, axios ... |          |

여기서 중요한 점은 위계적인 구조를 따른다는 것입니다. 상위 레이어는 하위 레이어를 참조하되 하위는 참조하지 못하는 것입니다.

예를 들어 pages에서 Home은 하위 widgets의 navbar를 참조할 수 있지만 navbar는 상위 Home을 참조하지 못합니다.

## 1.2. Slices

각 Layer는 Slices라는 서브 디렉토리를 가집니다. 이는 이제 두번째 분해 레벨로 가치 별로 묶는 것을 목표로 합니다 (group code by its value)

```tree
└── src
    ├── app/
    │   ├── providers/
    │   ├── styles/
    │   └── index.tsx/
    ├── pages/
    │   ├── home/
    │   └── about/
    ├── widgets/
    │   ├── newsfeed/
    │   ├── header/
    │   └── footer/
    ├── features/
    │   ├── user/
    │   ├── auth/
    │   └── favorites/
    ├── entities/
    │   ├── user/
    │   └── sessions/
    └── shared/
```

## 1.3. Segments

각각의 Slice는 다시 Segment로 구성됩니다. Segment의 목적은 Slice 내의 코드를 분할하는데 있습니다.

# 2. FSD의 장단점

## 2.1. 장점

- 아키텍처의 유지 보수가 수월 (교체, 추가, 제거에 용이)
- 표준화 가능
- 높은 확장성
- 비지니스 중심의 방법론
- 개발 스택과 무관
- 모듈 간 제어 및 명시적 연결

## 2.2. 단점

- 다른 솔루션에 비해 높은 진입 장벽
- 팀 내의 인식 필요
