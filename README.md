# 바닐라 JS 프로젝트 성능 개선

- url: https://anveloper.dev/front_5th_chapter4-2_basic/index.html

# 성능 개선 보고서

## 개선 전 상태

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


### 개선 전 PageSpeed 측정 결과

- Mobile
  ![mobile b](https://github.com/user-attachments/assets/900b3600-4117-49ce-b5c6-ca1f04440817)

- Desktop
  ![desktop b](https://github.com/user-attachments/assets/d5d2b1dc-f54f-444a-a2a8-6aa44df0a7e5)



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
  - (추가) imagemin 만으로는 해상도 조절이 안되서, cwebp 로 라이브러리 변경, 컴포넌트 랜더링에 정확히 맞는 사이즈로 변경
    ```yml
          - name: Convert images to WebP
            run: |
              mkdir -p dist/images
              cwebp -resize 1905 886 -q 80 images/Hero_Desktop.jpg -o dist/images/Hero_Desktop.webp
              cwebp -resize 934 749 -q 80 images/Hero_Tablet.jpg -o dist/images/Hero_Tablet.webp
              cwebp -resize 618 618 -q 80 images/Hero_Mobile.jpg -o dist/images/Hero_Mobile.webp
              cwebp -resize 200 200 -q 80 images/vr1.jpg -o dist/images/vr1.webp
              cwebp -resize 200 200 -q 80 images/vr2.jpg -o dist/images/vr2.webp
              cwebp -resize 200 200 -q 80 images/vr3.jpg -o dist/images/vr3.webp
              cwebp -resize 36 26 -q 80 images/menu_icon.png -o dist/images/menu_icon.webp
    ```
    
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
  - gh-pages 브랜치에서만 최적화된 이미지 사용
  - *.min.css, *.min.js 파일로 소스 압축
  - main 브랜치는 원본 소스만 유지
  
    ![image](https://github.com/user-attachments/assets/6620cb46-a90e-4144-9862-85bcd49b9b89)

- 초기 랜더링 속도개선을 위한 비동기 로딩 추가
- 필수 랜더링 요소를 제외한 스크립트 defer 추가
  ```js
  <script defer>
      (function (w, d, s, l, i) {
  /* ... */
  </script>
  <script type="text/javascript" src="//www.freeprivacypolicy.com/public/cookie-consent/4.1.0/cookie-consent.js" charset="UTF-8" defer></script>
  <script type="text/javascript" charset="UTF-8">
    window.addEventListener("DOMContentLoaded", () => {
      cookieconsent.run({
  /* ... */
  </script>
  ```
  
- 폰트 파일을 로컬에서 직접 제공하도록 변경 
  ```css
  @font-face {
    font-family: "Heebo";
    src: url("fonts/Heebo-Light.woff2") format("woff");
    font-display: swap;
    font-weight: 300;
    font-style: light;
  }
  /* ... */
  ```
- Hero 이미지  랜더링 개선, webp 변환, picture source 방식으로 변경
  ```html
      <section class="hero">
        <!-- <img class="desktop" src="images/Hero_Desktop.jpg" /> -->
        <!-- <img class="mobile" src="images/Hero_Mobile.jpg" /> -->
        <!-- <img class="tablet" src="images/Hero_Tablet.jpg" /> -->
        <picture>
          <source srcset="images/Hero_Mobile.webp" type="image/webp" media="(max-width: 576px)" />
          <source srcset="images/Hero_Tablet.webp" type="image/webp" media="(max-width: 960px)" />
          <source srcset="images/Hero_Desktop.webp" type="image/webp" media="(min-width: 961px)" />
          <img class="desktop" src="images/Hero_Desktop.webp" alt="hero image" loading="eager" width="1905" height="886" fetchpriority="high" />
        </picture>
      <!-- ... -->
      </section>
  ```

## 개선 후 상태

### 🎯 Lighthouse 점수
| 카테고리 | 점수 | 상태 |
|----------|------|------|
| Performance | 76% | 🟠 |
| Accessibility | 94% | 🟢 |
| Best Practices | 61% | 🟠 |
| SEO | 100% | 🟢 |
| PWA | 0% | 🔴 |

### 📊 Core Web Vitals (2024)
| 메트릭 | 설명 | 측정값 | 상태 |
|--------|------|--------|------|
| LCP | Largest Contentful Paint | 2.49s | 🟢 |
| INP | Interaction to Next Paint | N/A | 🟢 |
| CLS | Cumulative Layout Shift | 0.561 | 🔴 |

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

> 📅 측정 시간: 2025. 6. 3. 오전 8:06:13

 
### 개선 후 PageSpeed 측정 결과

- Mobile
  ![mobile a](https://github.com/user-attachments/assets/88be69ae-cf32-4e02-b9ce-2f4493388a18)

- Desktop
  ![desktop a](https://github.com/user-attachments/assets/c398057c-1eaf-47a1-8108-e22914a50ad7)
 
## 개선 결과

### 
