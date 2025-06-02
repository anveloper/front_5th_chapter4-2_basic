# 바닐라 JS 프로젝트 성능 개선

- url: https://anveloper.dev/front_5th_chapter4-2_basic/index.html


# 성능 개선 보고서

## 초기 상태

### 🎯 Lighthouse 점수
| 카테고리 | 점수 | 상태 |
|----------|------|------|
| Performance | 72% | 🟠 |
| Accessibility | 82% | 🟠 |
| Best Practices | 75% | 🟠 |
| SEO | 82% | 🟠 |
| PWA | 0% | 🔴 |

### 📊 Core Web Vitals (2024)
| 메트릭 | 설명 | 측정값 | 상태 |
|--------|------|--------|------|
| LCP | Largest Contentful Paint | 14.78s | 🔴 |
| INP | Interaction to Next Paint | N/A | 🟢 |
| CLS | Cumulative Layout Shift | 0.011 | 🟢 |

### 📝 Core Web Vitals 기준값
- **LCP (Largest Contentful Paint)**: 가장 큰 콘텐츠가 화면에 그려지는 시점 
  - 🟢 Good: < 2.5s
  - 🟠 Needs Improvement: < 4.0s
  - 🔴 Poor: ≥ 4.0s

- **INP (Interaction to Next Paint)**: 사용자 상호작용에 대한 전반적인 응답성
  - 🟢 Good: < 200ms
  - 🟠 Needs Improvement: < 500ms
  - 🔴 Poor: ≥ 500ms

- **CLS (Cumulative Layout Shift)**: 페이지 로드 중 예기치 않은 레이아웃 변경의 정도
  - 🟢 Good: < 0.1
  - 🟠 Needs Improvement: < 0.25
  - 🔴 Poor: ≥ 0.25

> 📅 측정 시간: 2025. 6. 2. 오후 8:04:35


### https://pagespeed.web.dev/ 측정 결과

- Mobile
  ![초기 측정 값](https://github.com/user-attachments/assets/900b3600-4117-49ce-b5c6-ca1f04440817)

- Desktop
  ![Frame 102](https://github.com/user-attachments/assets/d5d2b1dc-f54f-444a-a2a8-6aa44df0a7e5)



### 예측되는 문제 사항

- 이미지 최적화가 가장 큰 문제로 보임
- 가장 큰 이미지가 1.1mb 데스크탑 Hero 이미지로 원본 그대로를 노출중에 있음
- 이미지이 기본 크기가 설정되지 않아 레이아웃이 밀리는 현상이 있음
- js 스크립트 중, 불필요한 로직이 있고, 필수가 아님에도 초기에 동기적으로 접근하여 랜더링에 영향을 줌
- css 파일 중 외부에서 사용중인 폰트 파일이 전체 레이아웃의 변화에 영향을 줌

## 조치 사항

- 배포시에 이미지 파일을 `*.webp`로 변환
- 배포시에 `*.js`, `*.css` 파일은 `*.min.js`, `*.min.css` 파일로 압축
- 수동 압축도 가능하지만, github actions를 통한 자동화로 진행

  - imagemin-cli, imagemin-webp: 이미지 파일을 *.webp로 변환하는 라이브러리
    ```yml
          - name: Convert images to WebP
            run: |
              imagemin images/*.{png,jpg,jpeg} --plugin=webp --out-dir=dist/images
    ```
  - (추가) imagemin 만으로는 해상도 조절이 안되서, sharp 로 라이브러리 변경, 컴포넌트 랜더링에 정확히 맞는 사이즈로 변경

  - cleancss: *.min.css 파일로 압축하는 라이브러리
    ```yml
        - name: Minify CSS
          run: |
            cleancss -o dist/assets/styles.min.css dist/assets/styles.css
            rm dist/assets/styles.css
    ```

  - terser: *.min.js 파일로 압축하는 라이브러리
    ```yml
          - name: Minify JS
            run: |
              terser dist/assets/main.js -o dist/assets/main.min.js --compress --mangle
              terser dist/assets/products.js -o dist/assets/products.min.js --compress --mangle
              rm dist/assets/main.js dist/assets/products.js
    ```

- 초기 랜더링 속도개선을 위한 비동기 로딩 추가
- 필수 랜더링 요소를 제외한 스크립트 defer 추가
- 폰트 파일을 로컬에서 직접 제공하도록 변경 


 
