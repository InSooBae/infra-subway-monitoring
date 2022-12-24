<p align="center">
    <img width="200px;" src="https://raw.githubusercontent.com/woowacourse/atdd-subway-admin-frontend/master/images/main_logo.png"/>
</p>
<p align="center">
  <img alt="npm" src="https://img.shields.io/badge/npm-%3E%3D%205.5.0-blue">
  <img alt="node" src="https://img.shields.io/badge/node-%3E%3D%209.3.0-blue">
  <a href="https://edu.nextstep.camp/c/R89PYi5H" alt="nextstep atdd">
    <img alt="Website" src="https://img.shields.io/website?url=https%3A%2F%2Fedu.nextstep.camp%2Fc%2FR89PYi5H">
  </a>
  <img alt="GitHub" src="https://img.shields.io/github/license/next-step/atdd-subway-service">
</p>

<br>

# 인프라공방 샘플 서비스 - 지하철 노선도

<br>

## 🚀 Getting Started

### Install
#### npm 설치
```
cd frontend
npm install
```
> `frontend` 디렉토리에서 수행해야 합니다.

### Usage
#### webpack server 구동
```
npm run dev
```
#### application 구동
```
./gradlew clean build
```
<br>


### 1단계 - 웹 성능 테스트
1. 웹 성능예산은 어느정도가 적당하다고 생각하시나요

- 3초의 법칙을 지키며, 경쟁 사이트의 속도와 비슷한 성능을 내보자는 목표로 잡았습니다.
- 경쟁 사이트 중 중간값으로 기준을 잡았습니다.


### 모바일
---

|            | FCP          | TTI          | Speed Index  | TBT      | LCP          | CLS   | 성능 종합 |
|------------|--------------|--------------|--------------|----------|--------------|-------|-------|
| 서울교통공사     | 6.2 s        | 8.1 s        | 7.3 s | 780 ms | 6.2 s        | 0     | 7    |
| 네이버지도      | **2.2 s**         | 4.1 s  | 4.7 s  | 130 ms    | 3.8 s   | 0.762     | 67    |
| 카카오맵       | 1.7 s  | **5.8 s**        | **3.8 s** | 90 ms    | 1.9 s        | 0 | 94    |
| infra-subway | 2.7 s       | 2.8 s       | 2.7 s | 30 ms     | 2.8 s       | 0.004     | 67    |


### 데스크탑
---

|            | FCP          | TTI          | Speed Index  | TBT      | LCP          | CLS   | 성능 종합 |
|------------|--------------|--------------|--------------|----------|--------------|-------|-------|
| 서울교통공사     | 1.5 s        | **2.0 s**        | **2.1 s**        | 80 ms | 2.0 s        | 0.001     | 81    |
| 네이버지도      | 0.3 s         | 3.8 s  | 2.8 s  | 1,060 ms    | 4.8 s   | 0.019     | 38    |
| 카카오맵       | **0.5 s**  | 0.5 s        | 1.2 s        | 0 ms    | 0.5 s        | 0 | 99    |
| infra-subway | 16.9 s       | 17.8 s       | 16.9 s       | 810 ms     | 17.6 s       | 0.041     | 26    |




2. 웹 성능예산을 바탕으로 현재 지하철 노선도 서비스의 서버 목표 응답시간 가설을 세워보세요.

여타 다른 경쟁사들의 리소스(html,css,js 등)의 크기는 1mb를 안넘는데, infra-subway같은 경우 1mb를 넘는 파일(특히 vendors.js)로 인해 로드 시간이 더 늘어지는 것을 확인했다.

이로 인해 캐시된 내용이 아닌 첫 방문일 경우 상당히 느려질 수 있었다.

모바일 말고 데스크탑을 기준으로 삼을 것이고, 데스크탑에서의 다른 경쟁사들의 속도가 ㅐ부분 3초 언저리였기에 4초를 넘지 않는 것을 목표로 정한다.

---

### 2단계 - 부하 테스트 
1. 부하테스트 전제조건은 어느정도로 설정하셨나요

**대상 시스템 범위 설정**

- infra-subway의 테스팅할 시스템 범위는 **nginx - was - db**와 같다.
- 가장 많이 접속할 페이지는 **메인 페이지**와 **노선을 조회하는 페이지**이다.
- nginx - was - db 까지 거치는 API 중 가장 많이 이용될 API는 **노선 조회**일 것으로 보인다.
- 즉 **접속 빈도가 높은 페이지** 위주로 진행이 될 것 같다.


**목푯값 설정**
 
- [통합 데이터 지도](https://www.bigdata-map.kr/datastory/traffic/seoul) 

Throughput : 1일 평균 rps ~ 1일 최대 rps
  1. DAU: 승하차 인원 하루 평균 약 4,470,000 명
  2. 피크시간대 집중율 : 2 (1,000,000[최대 트래픽]/ 500,000[평소 트래픽])
  3. 1명당 1일 평균 요청 수 : 2

- 1일 총 접속 수 = 4,470,000(DAU[1일 사용자 수]) * 2 = 8,940,000
- 1일 평균 rps = 8,940,000(1일 총 접속 수) / 86,400(초/일) = 103.472 -(소수점 1의 자리 반올림)> 103 rps
- 1일 최대 rps = 103 * 2 = 206 rps

Latency : 일반적으로 50~100ms이하

VUser 구하기
- 시나리오는 메인 페이지에서 노선 조회 페이지 접속 후 지향하는 노선을 찾는 것
  - 3번의 요청(시나리오 상 메인 페이지 + 노선 조회 페이지 + 노선 검색)
  - 총 latency 목표값 300ms(100 + 100 + 100)
- T(VU iteration) = (R * http_req_duration) + a(예상 지연 시간)
(3회 요청 * 0.3 s) + 1 s = 1.9 s
평균 VUser = 103 * 1.9 / 3 = 65.2
최대 VUser = 206 * 1.9 / 3 = 130.4

- 


2. Smoke, Load, Stress 테스트 스크립트와 결과를 공유해주세요

**smoke**

![](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/025651d3-a857-4bc8-a24e-a0ef10942b7d/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20221224%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20221224T020424Z&X-Amz-Expires=86400&X-Amz-Signature=ac16ce5fcb9d960c5ccd3658393d25adf2e5dafaed159dc4b750f6ad44e2460c&X-Amz-SignedHeaders=host&response-content-disposition=filename%3D%22Untitled.png%22&x-id=GetObject)

![](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/c9dc93a5-1e6c-4648-8a1a-dd83dbd3688c/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20221224%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20221224T020455Z&X-Amz-Expires=86400&X-Amz-Signature=6c93df54cd6268a3c0978a6d75db578db74098251941326d97a11bdc32d21119&X-Amz-SignedHeaders=host&response-content-disposition=filename%3D%22Untitled.png%22&x-id=GetObject)

```javascript
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
//    stages: [
//      {duration: '1m', target: 1}
//    ]
    vus: 1,
    duration: '1m',
    thresholds: {
        http_req_duration: ['p(99)<3000'], // 99% of requests must complete below 3s
    },
};

const BASE_URL = 'https://insoobae.n-e.kr/';
const STATION_MIN_ID = 1;
const STATION_MAX_ID = 727;

function makeRandomStationId() {
    return Math.floor(Math.random() * (STATION_MAX_ID - STATION_MIN_ID) + STATION_MIN_ID);
}

export default function()  {
    const params = {
        headers: {
            'Content-Type': 'application/json',
        },
    };

    check(http.get(`${BASE_URL}`, params), {
        '메인 페이지 응답 성공': (resp) => resp.status === 200,
    });

    check(http.get(`${BASE_URL}/path`, params), {
        '노선 검색 페이지 응답 성공': (resp) => resp.status === 200,
    });

    const findPathUrl = `${BASE_URL}/path?source=${makeRandomStationId()}&target=${makeRandomStationId()}`;
    check(http.get(findPathUrl, params), {
        '노선 검색 API 호출 성공': (resp) => resp.status === 200,
    });

    sleep(1);
};
```


**load**

![](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/55d76780-c51b-4304-9a0e-20df87300a29/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20221224%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20221224T020611Z&X-Amz-Expires=86400&X-Amz-Signature=c8a491d23f1f3c38e78e40583c156936b744298131e40cba4bda3d4019bf1c93&X-Amz-SignedHeaders=host&response-content-disposition=filename%3D%22Untitled.png%22&x-id=GetObject)

```javascript
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
    stages: [
        {duration: '10s', target: 130},
        {duration: '30s', target: 130},
        {duration: '10s', target: 0},
    ],
//    vus: 1,
//    duration: '1m',
    thresholds: {
        http_req_duration: ['p(99)<3000'], // 99% of requests must complete below 3s
    },
};

const BASE_URL = 'https://insoobae.n-e.kr/';
const STATION_MIN_ID = 1;
const STATION_MAX_ID = 727;

function makeRandomStationId() {
    return Math.floor(Math.random() * (STATION_MAX_ID - STATION_MIN_ID) + STATION_MIN_ID);
}

export default function()  {
    const params = {
        headers: {
            'Content-Type': 'application/json',
        },
    };

    check(http.get(`${BASE_URL}`, params), {
        '메인 페이지 응답 성공': (resp) => resp.status === 200,
    });

    check(http.get(`${BASE_URL}/path`, params), {
        '노선 검색 페이지 응답 성공': (resp) => resp.status === 200,
    });

    const findPathUrl = `${BASE_URL}/path?source=${makeRandomStationId()}&target=${makeRandomStationId()}`;
    check(http.get(findPathUrl, params), {
        '노선 검색 API 호출 성공': (resp) => resp.status === 200,
    });

    sleep(1);
};
```

**stress**

![](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/e0e216c2-e221-4a6d-b143-e173d57b5329/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20221224%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20221224T020625Z&X-Amz-Expires=86400&X-Amz-Signature=aa5d4cecaabeda148ff2dabca19748e1d4e13a20023714ef44f2ddc371832678&X-Amz-SignedHeaders=host&response-content-disposition=filename%3D%22Untitled.png%22&x-id=GetObject)

```javascript
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
    stages: [
        {duration: '5s', target: 20},
        {duration: '5s', target: 40},
        {duration: '5s', target: 80},
        {duration: '5s', target: 130},
        {duration: '5s', target: 195},
        {duration: '5s', target: 0},
    ],
//    vus: 1,
//    duration: '1m',
    thresholds: {
        http_req_duration: ['p(99)<3000'], // 99% of requests must complete below 3s
    },
};

const BASE_URL = 'https://insoobae.n-e.kr/';
const STATION_MIN_ID = 1;
const STATION_MAX_ID = 727;

function makeRandomStationId() {
    return Math.floor(Math.random() * (STATION_MAX_ID - STATION_MIN_ID) + STATION_MIN_ID);
}

export default function()  {
    const params = {
        headers: {
            'Content-Type': 'application/json',
        },
    };

    check(http.get(`${BASE_URL}`, params), {
        '메인 페이지 응답 성공': (resp) => resp.status === 200,
    });

    check(http.get(`${BASE_URL}/path`, params), {
        '노선 검색 페이지 응답 성공': (resp) => resp.status === 200,
    });

    const findPathUrl = `${BASE_URL}/path?source=${makeRandomStationId()}&target=${makeRandomStationId()}`;
    check(http.get(findPathUrl, params), {
        '노선 검색 API 호출 성공': (resp) => resp.status === 200,
    });

    sleep(1);
};
```

---

### 3단계 - 로깅, 모니터링
1. 각 서버내 로깅 경로를 알려주세요

2. Cloudwatch 대시보드 URL을 알려주세요
