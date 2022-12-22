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
| 서울교통공사     | 1.5 s        | 2.0 s        | 2.1 s        | 80 ms | 2.0 s        | 0.001     | 81    |
| 네이버지도      | 0.3 s         | 3.8 s  | 2.8 s  | 1,060 ms    | 4.8 s   | 0.019     | 38    |
| 카카오맵       | 0.5 s  | 0.5 s        | 1.2 s        | 0 ms    | 0.5 s        | 0 | 99    |
| infra-subway | 16.9 s       | 17.8 s       | 16.9 s       | 810 ms     | 17.6 s       | 0.041     | 26    |




2. 웹 성능예산을 바탕으로 현재 지하철 노선도 서비스의 서버 목표 응답시간 가설을 세워보세요.

여타 다른 경쟁사들의 리소스(html,css,js 등)의 크기는 1mb를 안넘는데, infra-subway같은 경우 1mb를 넘는 파일(특히 vendors.js)로 인해 로드 시간이 더 늘어지는 것을 확인했다.

이로 인해 캐시된 내용이 아닌 첫 방문일 경우 상당히 느려질 수 있었다.

모바일 말고 데스크탑을 기준으로 삼을 것이고, 데스크탑에서의 다른 경쟁사들의 속도가 ㅐ부분 3초 언저리였기에 4초를 넘지 않는 것을 목표로 정한다.

---

### 2단계 - 부하 테스트 
1. 부하테스트 전제조건은 어느정도로 설정하셨나요

2. Smoke, Load, Stress 테스트 스크립트와 결과를 공유해주세요

---

### 3단계 - 로깅, 모니터링
1. 각 서버내 로깅 경로를 알려주세요

2. Cloudwatch 대시보드 URL을 알려주세요
