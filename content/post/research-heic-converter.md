---
title: "[회고] heic-convert 분석 - NodeJS 라이브러리"
date: 2024-01-25T18:36:44+09:00
draft: true

categories:
- 회고
tags:
- 이미지처리
- Node
keywords:
- HEIF
- Node에서 HEIC 이미지 가공하기

---
# 

# TL;DR

- 커뮤니티 만들때 HEIF파일을 JPEG로 바꾸어 다루면 편합니다.
- NodeJS에서 이를 구현한 heic-converter 라이브러리를 분석해 봅니다.

# 왜 와이

커뮤니티를 만들때면 이미지를 입력 받는 경우가 있습니다. 
이때 애플의 아이폰/아이패드의 경우, 고효율 압축 방식인 HEIF(High Efficiency Image File Format) 방식을 사용합니다. 
당연하게도, ISO 표준인 JPEG(Joint Photographic Expert Group) 등과는 다른 파일 형식을 가지고 있습니다.
libheif등의 라이브러리가 등장하는 대목입니다.

만약 node에서 sharp를 이용해 heif로 파일을 처리하고자 한다면 이 과정은 정말 쉽지 않습니다.
HEIF는 HEVC(High Efficiency Video Coding)을 이용해 compression을 하기 때문에 libde265와 x265 같은 라이브러리가 추가로 필요하기 때문입니다.
이전에 이 방식으로 heic 파일을 변환하고자 많은 시간을 녹였던 기억이 납니다.

이 글은 HEIF 파일을 JPEG로 바꾸어 다루는 아이디어 부터 출발해, 이 아이디어를 구현한 npm 라이브러리인 heic-converter를 분석했던 과정을 회고하는 글입니다.
도데체 HEIF파일을 어떻게 변환하는지 살펴보고자 분석을 하는 글이니, HEIF에 대한 자세한 설명 등은 [킹무위키](https://namu.wiki/w/HEIF)나 chatgpt등을 참조해 주세요.


# 시작은 소스코드에서부터

## heic-convert
| https://github.com/catdad-experiments/heic-convert/tree/master


폴더 구조를 먼저 살펴 봅니다. 아무래도 `lib.js` 가 핵심 로직을 담고 있는 것으로 보이는 군요. 

### lib.js
전문이 40줄 정도밖에 안되는 간단한 코드입니다. 여기서 주목할 함수는 convert라는 함수입니다.
convert를 따라가면, 주입받은 `decode`함수로 buffer를 해석하고, `convertImage`함수로 이미지를 변환합니다.
`convertImage`함수는 주입 받은 `encode`함수를 이용해 형식을 변환하는 것으로 보입니다.
```javascript
module.exports = (decode, encode) => {
  const convertImage = async ({ image, format, quality }) => {
    return await encode[format]({
      width: image.width,
      height: image.height,
      data: image.data,
      quality
    });
  };

  const convert = async ({ buffer, format, quality, all }) => {
    if (!encode[format]) {
      throw new Error(`output format needs to be one of [${Object.keys(encode)}]`);
    }

    if (!all) {
      const image = await decode({ buffer });
      return await convertImage({ image, format, quality });
    }

    const images = await decode.all({ buffer });

    return images.map(image => {
      return {
        convert: async () => await convertImage({
          image: await image.decode(),
          format,
          quality
        })
      };
    });
  };

  return {
    one: async ({ buffer, format, quality = 0.92 }) => await convert({ buffer, format, quality, all: false }),
    all: async ({ buffer, format, quality = 0.92 }) => await convert({ buffer, format, quality, all: true })
  };
};
```

### index.js
그럼 이제 주입 받는 `decode`함수와 `encode`함수를 찾아갑니다.
```javascript
const decode = require('heic-decode');
const formats = require('./formats-node.js');
const { one, all } = require('./lib.js')(decode, formats);

module.exports = one;
module.exports.all = all;
```

`decode`함수는 `heic-decode`라는 라이브러리인 것을 알 수 있습니다. `encode`함수는 `formats-node.js`에 있겠군요

### formats-node.js
지원하는 형식은 PNG형식과 JPEG형식만이군요! 다른 형식이 필요하면 이 부분을 수정하면 되겠어요.
```javascript
const jpegJs = require('jpeg-js');
const { PNG } = require('pngjs');

module.exports = {};

module.exports.JPEG = ({ data, width, height, quality }) => jpegJs.encode({ data, width, height }, Math.floor(quality * 100)).data;

module.exports.PNG = ({ data, width, height }) => {
  const png = new PNG({ width, height });
  png.data = data;

  return PNG.sync.write(png, {
    width: width,
    height: height,
    deflateLevel: 9,
    deflateStrategy: 3,
    filterType: -1,
    colorType: 6,
    inputHasAlpha: true
  });
};
```

### 정리
`convert`함수는 하나의 HEIF 포멧 버퍼를 변환하는 함수로, `heic-decode` 라이브러리를 이용해 HEIF 버퍼를 변환한다. 
그리고 변환한 버퍼를 JPEG나 PNG포멧으로 변경한다.

## heic-decode
그럼 다음으로는 HEIF 버퍼를 실제로 변환하는 `heic-decode`를 까보겠습니다. 
같은 사람이 만들어 그런지 구조는 비슷하군요! 
일단 `lib.js`부터 열어보면 될 것 같습니다.

### lib.js
이번에는 80줄이 되는 코드라 buffer를 decode하는 로직만 발췌하였습니다. 결국 libheif를 주입 받아 decode를 하게 되는 군요
```javascript
// ...
module.exports = libheif => {
  const decodeBuffer = async ({ buffer, all }) => {
    if (!isHeic(buffer)) {
      throw new TypeError('input buffer is not a HEIC image');
    }

    // wait for module to be initialized
    // currently it is synchronous but it might be async in the future
    await libheif.ready;

    const decoder = new libheif.HeifDecoder();
    const data = decoder.decode(buffer);

    // ...
  };

  return {
    one: async ({ buffer }) => await decodeBuffer({ buffer, all: false }),
    all: async ({ buffer }) => await decodeBuffer({ buffer, all: true })
  };
};
```

### index.js
그럼 이제 여기만 보면 어떻게 libheif를 주입하는지 알 수 있을 겁니다.
`libheif-js`의 wasm-bundle을 이용하는군요! 왜 wasm인지는 차치하고 일단 넘어가 봅시다.

```javascript
const libheif = require('libheif-js/wasm-bundle');

const { one, all } = require('./lib.js')(libheif);

module.exports = one;
module.exports.all = all;
```

### 정리
`libheif-js`를 주입받아 HEIF파일을 변환합니다.

## libheif-js
살펴볼 파일은 `wasm.js`, `wasm-bundle.js`이겠군요. 일단 이전과 구조가 다르니 `index.js`먼저 살펴보도록 합시다.

### index.js
> ???

js번들을 만드는 건가? 바로 `wasm-bundle.js`를 살펴봐야겠군요.
```javascript
module.exports = require('./libheif/libheif.js')();
```

### wasm-bundle.js
> ???

남은 wasm.js를 봐야겠습니다.
```javascript
module.exports = require('./libheif-wasm/libheif-bundle.js')();
```

### wasm.js
자, 여기까지 나온 것을 보고 생각해 보았습니다.
이제 찾아야 하는 것은 다음 폴더들이 어떻게 만들어지는지 알아보는 것입니다.
1. libheif-wasm

```javascript
// I technically don't have to do this, but I am keeping it around
// for demonstration purposes
const fs = require('fs');
const wasmBinary = fs.readFileSync('./libheif-wasm/libheif.wasm');

module.exports = require('./libheif-wasm/libheif.js')({ wasmBinary });
```

### scripts/install.js
일단 libheif를 설치하는 부분으로 보입니다. libheif를 다운받고, esbuild를 통해 `libheif-wasm/libheif-bundle.js`를 만들어 내는 군요
```javascript

// ...
const base = `https://github.com/catdad-experiments/libheif-emscripten/releases/download/${version}`;
const tarball = `${base}/libheif.tar.gz`;

// ...

(async () => {

    // ...

      for await (const entry of (await getStream(tarball)).pipe(gunzip()).pipe(tar.extract())) {
        const basedir = entry.header.name.split('/')[0];

        if (entry.header.type === 'file' && ['libheif', 'libheif-wasm'].includes(basedir)) {
          const outfile = path.resolve(root, entry.header.name);
          console.log(`  writing "${outfile}"`);
          await fs.outputFile(outfile, await autoReadStream(entry));
        } else {
          await autoReadStream(entry);
        }
      }

    //...
await esbuild.build({
    ...buildOptions,
    outfile: path.resolve(root, 'libheif-wasm/libheif-bundle.js'),
    format: 'iife',
    globalName: 'libheif',
    footer: {
      // hack to support a single bundle as a node cjs module
      // and a browser <script>, similar to the js version libheif
      js: `
libheif = libheif.default;
if (typeof exports === 'object' && typeof module === 'object') {
  module.exports = libheif;
}`
    }
  });

// ...
})().then(() => {
  console.log(`fetched libheif ${version}`);
}).catch(err => {
  console.error(`failed to fetch libheif ${version}\n`, err);
  process.exitCode = 1;
});
//...
```

### 정리
libheif를 직접 다운 받아 libheif를 번들링 합니다. libheif/libheif.js는 libheif를 살펴 봐야겠습니다.

# ref

- https://cloudinary.com/guides/image-formats/heif-format-meet-the-the-next-evolution-of-jpeg
- [킹무위키 HEIF](https://namu.wiki/w/HEIF)
- [heic-convert](https://github.com/catdad-experiments/heic-convert)
- [heic-decode](https://github.com/catdad-experiments/heic-decode)
- [libheif-js](https://github.com/catdad-experiments/libheif-js)
- [libheif](https://github.com/strukturag/libheif)

