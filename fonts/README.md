# fonts

청첩장에서 직접 로드하는 웹폰트. 원본 TTF는 저장소에 두지 않고 **서브셋 woff2**만 둔다.

| 파일 | 용도 | 크기 |
|---|---|---|
| `08SeoulHangangL.woff2` | 한글 본문/제목 (weight 300) | 147 KB |
| `08SeoulHangangM.woff2` | 한글 본문/제목 (weight 500) | 146 KB |
| `white_angelica/WhiteAngelica.woff2` | 히어로 영문 필기체 | 17 KB |

원본 TTF 합계는 16.6 MB였다. 하객 대부분이 모바일 데이터로 여는 페이지라
서브셋 + woff2 변환으로 약 98% 줄였다.

## 포함 문자 범위

한글은 **KS X 1001 상용 2,350자**와 `index.html`에 실제 등장하는 문자(한자 父·母 포함),
ASCII, 자주 쓰는 문장부호를 담았다. 일상적인 한국어 문장은 모두 표현되지만,
이 범위를 벗어나는 희귀 글자를 본문에 넣으면 그 글자만 시스템 폰트로 표시된다.

White Angelica는 히어로 문구에만 쓰이므로 라틴 문자 영역만 남겼다.

## 재생성 방법

원본 TTF는 git 이력에 있다 (`0fcf99e` 이전 커밋).

```sh
python3 -m venv /tmp/fontenv
/tmp/fontenv/bin/pip install fonttools brotli

# 포함할 문자 목록 생성 (KS X 1001 = iso2022_kr로 인코딩 가능한 음절)
/tmp/fontenv/bin/python - <<'PY' > /tmp/chars.txt
src = open('index.html', encoding='utf-8').read()
ksx = set()
for cp in range(0xAC00, 0xD7A4):
    try:
        chr(cp).encode('iso2022_kr'); ksx.add(chr(cp))
    except Exception:
        pass
chars = set(src) | ksx | {chr(c) for c in range(0x20, 0x7F)} | set('·×÷…‘’“”—–♥♡€₩※°')
print(''.join(sorted(chars)), end='')
PY

/tmp/fontenv/bin/pyftsubset 08SeoulHangangL.ttf \
  --output-file=fonts/08SeoulHangangL.woff2 \
  --flavor=woff2 --text-file=/tmp/chars.txt --layout-features='*'
```

White Angelica는 라틴 영역만 지정한다.

```sh
/tmp/fontenv/bin/pyftsubset WhiteAngelica.ttf \
  --output-file=fonts/white_angelica/WhiteAngelica.woff2 \
  --flavor=woff2 --unicodes='U+0020-007E,U+00A0-00FF,U+2018-201D,U+2026' \
  --layout-features='*'
```

## 주의

- `@font-face`의 `url()`은 `index.html` 기준 **상대 경로**로 적는다.
  GitHub Pages가 저장소 하위 경로로 서비스하므로 `/fonts/...` 절대 경로는 배포본에서 404가 난다.
- 파일명 대소문자를 실제 파일과 정확히 맞춘다. macOS는 구분하지 않지만 배포 서버는 구분한다.
- woff2를 지원하지 않는 매우 오래된 브라우저에서는 `--sans` / `--myeongjo` 스택의
  시스템 폰트(Pretendard, Apple SD Gothic Neo 등)로 자연스럽게 대체된다.
