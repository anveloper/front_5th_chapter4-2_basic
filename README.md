# 바닐라 JS 프로젝트 성능 개선

- url: https://anveloper.dev/front_5th_chapter4-2_basic/index.html

## 성능 개선 보고서

### 조치 사항

- 배포시에 이미지 파일을 `*.webp`로 변환
- 배포시에 `*.js`, `*.css` 파일은 `*.min.js`, `*.min.css` 파일로 압축
- 수동 압축도 가능하지만, github actions를 통한 자동화로 진행

  - imagemin-cli, imagemin-webp: 이미지 파일을 *.webp로 변환하는 라이브러리
    ```yml
          - name: Convert images to WebP
            run: |
              imagemin images/*.{png,jpg,jpeg} --plugin=webp --out-dir=dist/images
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

- 초기 랜더링 속도개선을 위한 비동기 로딩 추가
- 필수 랜더링 요소를 제외한 스크립트 defer 추가
- CSS preload+onload 형식으로 변경


 