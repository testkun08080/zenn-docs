---
title: ポストエフェクト祭り。React/ポストプロセスを触ってカスタムシェーダーの作り方をざっくり理解してみよう
published_at: '2025-10-16 12:30'
private: false
tags:
  - react
  - threejs
  - shader
  - postprocessing
  - webgl
updated_at: '2025-10-15T06:44:44.101Z'
id: null
organization_url_name: null
slide: false
---

# はじめに

もう九月か。。。
独り立ちして約１年ぐらい経とうとしている中、ポートフォリオページがまだ`Under Maintenance`なので、
重い腰を上げて、ポートフォリオページを作成することにしました。
そこで、大好きなシェーダーをポートフォリオにもモリモリ使おうと思い立ったのがきっかけです。
（ゲームだとそうは行かないので）

この記事では以下モジュール

- **[@react-three/drei](https://github.com/pmndrs/drei)**
- **[@react-three/postprocessing](https://docs.pmnd.rs/react-postprocessing)**
- **[postprocessing](https://github.com/pmndrs/postprocessing)**

で用意されている一般的なポストエフェクトの実装と、カスタムシェーダーを使ったワールドノーマルを表示を行ってみます。
また、一般的なカスタムシェーダーを Claude に手伝っておもらったので、載せておきます。

少し目次は長いですけど、こんなのもあるのかぁ程度にサンプルとしてみて使っていただけたらと思います。

## レポ

https://github.com/testkun08080/react-postprocess-tester

## テスター用ページ

https://testkun.net/react-postprocess-tester/

## 開発環境

- macOS Sequoia 15.5
- VsCode
- Node.js 20+
- npm

---

<details><summary>使用技術スタック</summary>


### コアライブラリ

- **React 19.1.1** - UI フレームワーク
- **Vite 7.1.2** - ビルドツール
- **Three.js 0.179.1** - 3D レンダリングエンジン
- **@react-three/fiber 9.3.0** - React と Three.js のブリッジ
- **@react-three/drei 10.7.4** - React Three Fiber ヘルパー集
- **@react-three/postprocessing 3.0.4** - ポストプロセス React ラッパー
- **postprocessing 6.37.7** - ポストプロセスエフェクトライブラリ

### UI・ツール

- **TailwindCSS 4.1.12** - スタイリング
- **Leva 0.10.0** - リアルタイムパラメーター調整 UI
  </details>

---

## プロジェクト構造

```
react-postprocess-tester/
├── src/
│   ├── main.jsx                      # エントリーポイント
│   ├── App.jsx                       # ルーター設定
│   ├── components/
│   │   ├── Navigation.jsx            # ナビゲーション
│   │   ├── Scene.jsx                 # 3Dシーン設定
│   │   └── postprocessing/           # ポストプロセスエフェクト
│   │       ├── PostEffects.jsx       # エフェクトコントローラー
│   │       ├── WaveEffect.jsx        # 波状歪みエフェクト
│   │       ├── RGBSplitEffect.jsx    # 色収差エフェクト
│   │       ├── KaleidoscopeEffect.jsx # 万華鏡エフェクト
│   │       ├── ColorShiftEffect.jsx   # HSV色変換
│   │       ├── FractalNoiseEffect.jsx # フラクタルノイズ
│   │       ├── EdgeOutlineEffect.jsx  # エッジ検出
│   │       ├── LensFlareEffect.jsx    # レンズフレア
│   │       ├── ViewDepthVisualization.jsx    # デプスバッファ可視化
│   │       ├── SimpleCheckNormalEffect.jsx   # 法線ベクトル可視化
│   │       └── shaders/              # GLSLシェーダーファイル
│   │           ├── wave.glsl
│   │           ├── rgbSplit.glsl
│   │           ├── kaleidoscope.glsl
│   │           ├── colorShift.glsl
│   │           ├── fractalNoise.glsl
│   │           ├── edgeOutline.glsl
│   │           ├── lensFlare.glsl
│   │           ├── viewDepth.glsl
│   │           ├── worldNormal.glsl
│   │           └── index.js          # シェーダーエクスポート
│   └── pages/
│       └── PostEffectsSample.jsx     # メインデモページ
├── package.json
├── vite.config.js
└── tailwind.config.js
```

---

# セットアップ

## 1. プロジェクトのクローン

```bash
git clone https://github.com/testkun08080/react-postprocess-tester.git
cd react-postprocess-tester
```

## 2. 依存関係のインストール

```bash
npm install
```

## 3. 開発サーバー起動

```bash
npm run dev
```

ブラウザで http://localhost:5173 にアクセスできるはずです。

![サンプル画面](https://raw.githubusercontent.com/testkun08080/zenn-docs/main/images/react-postshader-195d07b1ed77da/sample-screen.png)
_Leva コントロールでリアルタイムにエフェクトを調整できます_

---

# ビルトインエフェクトの使い方

`@react-three/postprocessing`には 20 種類以上のビルトインエフェクトが用意されています。

## 基本的な使い方

```jsx
import { EffectComposer, Bloom, Vignette } from "@react-three/postprocessing";

function PostEffects() {
  return (
    <EffectComposer>
      <Bloom intensity={1.0} luminanceThreshold={0.9} />
      <Vignette offset={0.5} darkness={0.5} />
    </EffectComposer>
  );
}
```

## 主要なビルトインエフェクト

- **Bloom** - 光の滲みエフェクト
- **DepthOfField** - 被写界深度（ボケ効果）
- **ChromaticAberration** - 色収差
- **Glitch** - グリッチノイズ
- **Vignette** - ビネット（周辺減光）
- **SSAO/N8AO** - スクリーンスペース アンビエント オクルージョン
- **ToneMapping** - トーンマッピング

## Leva でパラメーター調整

`Leva`を使ってリアルタイムにパラメーターを調整できるようにします

```jsx
import { useControls } from "leva";

const bloomControls = useControls("Bloom", {
  enabled: false,
  intensity: { value: 1.0, min: 0, max: 3, step: 0.01 },
  luminanceThreshold: { value: 0.9, min: 0, max: 1, step: 0.01 },
});
```

---

# カスタムシェーダーの作り方

ここからが本題です。カスタムシェーダーを作ってワールドノーマルを可視化してみます。

## 1. Effect クラスを継承したクラスを作成

`postprocessing`ライブラリの`Effect`クラスを継承します。

```jsx
// SimpleCheckNormalEffect.jsx
import { forwardRef, useMemo, useEffect, useContext } from "react";
import { Effect } from "postprocessing";
import { Uniform, Matrix4 } from "three";
import { EffectComposerContext } from "@react-three/postprocessing";
import { checkNormalShader } from "./shaders/index.js";

class SimpleCheckNormalEffectImpl extends Effect {
  constructor({ normalBuffer, mode = 0, useWorldSpace = true }) {
    super("SimpleCheckNormalEffect", checkNormalShader, {
      uniforms: new Map([
        ["normalBuffer", new Uniform(normalBuffer)],
        ["uMode", new Uniform(mode)],
        ["uUseWorldSpace", new Uniform(useWorldSpace)],
        ["cameraMatrixWorld", new Uniform(new Matrix4())],
        ["viewMatrix", new Uniform(new Matrix4())],
        ["projectionMatrix", new Uniform(new Matrix4())],
        ["inverseProjectionMatrix", new Uniform(new Matrix4())],
      ]),
    });
  }

  // Setter for mode
  set mode(value) {
    this.uniforms.get("uMode").value = value;
  }

  set useWorldSpace(value) {
    this.uniforms.get("uUseWorldSpace").value = value;
  }

  // Camera matrix setters
  set cameraMatrixWorld(matrix) {
    this.uniforms.get("cameraMatrixWorld").value = matrix;
  }

  set viewMatrix(matrix) {
    this.uniforms.get("viewMatrix").value = matrix;
  }

  set projectionMatrix(matrix) {
    this.uniforms.get("projectionMatrix").value.copy(matrix);
  }

  set inverseProjectionMatrix(matrix) {
    this.uniforms.get("inverseProjectionMatrix").value.copy(matrix);
  }
}
```

## 2. React コンポーネントでラップ

React Three Fiber と統合するために`forwardRef`を使います。

```jsx
export const SimpleCheckNormalEffect = forwardRef((props, ref) => {
  const { normalPass, camera } = useContext(EffectComposerContext);

  const effect = useMemo(
    () =>
      new SimpleCheckNormalEffectImpl({
        normalBuffer: normalPass?.texture,
        ...props,
      }),
    [normalPass, props]
  );

  // Update camera matrices
  useEffect(() => {
    if (camera) {
      effect.cameraMatrixWorld = camera.matrixWorld;
      effect.viewMatrix = camera.matrixWorldInverse;
      effect.projectionMatrix = camera.projectionMatrix;
      effect.inverseProjectionMatrix = camera.projectionMatrixInverse;
    }
  }, [effect, camera]);

  // Update other properties
  useEffect(() => {
    if (props.mode !== undefined) effect.mode = props.mode;
    if (props.useWorldSpace !== undefined)
      effect.useWorldSpace = props.useWorldSpace;
  }, [effect, props]);

  return <primitive ref={ref} object={effect} dispose={null} />;
});
```

## 3. GLSL シェーダーを記述

```glsl
// worldNormal.glsl
uniform sampler2D normalBuffer;
uniform int uMode;
uniform bool uUseWorldSpace;
uniform mat4 cameraMatrixWorld;
uniform mat4 viewMatrix;
uniform mat4 projectionMatrix;
uniform mat4 inverseProjectionMatrix;

// Convert view space normal to world space
vec3 viewToWorldNormal(vec3 viewNormal) {
  vec4 worldNormal = vec4(viewNormal, 1.0) * viewMatrix;
  return worldNormal.xyz;
}

// Normal visualization modes
vec3 visualizeNormal(vec3 normal, int mode) {
  vec3 color;
  vec3 n = normalize(normal);

  if (mode == 0) {
    // Normal RGB mode
    color = n * 0.5 + 0.5; // Remap from [-1,1] to [0,1]
  } else if (mode == 1) {
    // Red channel only (X component)
    float x = n.x * 0.5 + 0.5;
    color = vec3(x, 0.0, 0.0);
  } else if (mode == 2) {
    // Green channel only (Y component)
    float y = n.y * 0.5 + 0.5;
    color = vec3(0.0, y, 0.0);
  } else {
    // Blue channel only (Z component)
    float z = n.z * 0.5 + 0.5;
    color = vec3(0.0, 0.0, z);
  }

  return color;
}

void mainImage(const in vec4 inputColor, const in vec2 uv, out vec4 outputColor) {
  // Read normal from normal buffer (view space)
  vec3 viewSpaceNormal = texture2D(normalBuffer, uv).xyz;
  viewSpaceNormal = viewSpaceNormal * 2.0 - 1.0; // Remap to [-1,1]

  // Choose between view space and world space
  vec3 normal;
  if (uUseWorldSpace) {
    normal = viewToWorldNormal(viewSpaceNormal);
  } else {
    normal = viewSpaceNormal;
  }

  // Generate visualization color
  vec3 normalColor = visualizeNormal(normal, uMode);

  outputColor = vec4(normalColor, inputColor.a);
}
```

## 4. EffectComposer に組み込む

```jsx
import { SimpleCheckNormalEffect } from "./SimpleCheckNormalEffect.jsx";

const worldNormalControls = useControls("World Normal", {
  enabled: false,
  mode: { value: 0, min: 0, max: 3, step: 1 },
  useWorldSpace: true,
});

<EffectComposer>
  {worldNormalControls.enabled && (
    <SimpleCheckNormalEffect
      mode={worldNormalControls.mode}
      useWorldSpace={worldNormalControls.useWorldSpace}
    />
  )}
</EffectComposer>;
```

![ワールドノーマル表示](https://raw.githubusercontent.com/testkun08080/zenn-docs/main/images/react-postshader-195d07b1ed77da/checknormal.png)
_ワールドノーマルが色として可視化されます_

---

# カスタムエフェクト実装のポイント

## Effect クラスの基本構造

```javascript
new Effect(name, fragmentShader, {
  uniforms: new Map([["uniformName", new Uniform(value)]]),
  blendFunction: BlendFunction.NORMAL,
  attributes: EffectAttribute.CONVOLUTION,
});
```

## mainImage 関数

GLSL シェーダーの`mainImage`関数が、各ピクセルに対して実行されます。

```glsl
void mainImage(
  const in vec4 inputColor,  // 入力カラー
  const in vec2 uv,          // UV座標 (0.0 ~ 1.0)
  out vec4 outputColor       // 出力カラー
)
```

## NormalPass の利用

法線バッファを使うには`NormalPass`を有効にする必要があります。

```jsx
<EffectComposer>
  <NormalPass />
  {/* Your effects here */}
</EffectComposer>
```

`EffectComposerContext`から`normalPass`を取得できます。

```jsx
const { normalPass, camera } = useContext(EffectComposerContext);
```

---

# シェーダー一覧

このプロジェクトでは以下のカスタムシェーダーを確認できます。
スライドバーで画像比較がここでできればよかったんですけど、厳しいのでテスター用のページで見ていただければと思います。

## SMAA

絶妙な違いですけど、やっぱ有無では違いますね。

![SMAA-ena](https://raw.githubusercontent.com/testkun08080/zenn-docs/main/images/react-postshader-195d07b1ed77da/ssma-ena.png)
_ON_

![SMAA-dis](https://raw.githubusercontent.com/testkun08080/zenn-docs/main/images/react-postshader-195d07b1ed77da/ssma-dis.png)
_OFF_

## Auto Focus

マニュアルでもマウスで試すことも可能です

![autofocus](https://raw.githubusercontent.com/testkun08080/zenn-docs/main/images/react-postshader-195d07b1ed77da/autofocus.png)

## SSAO

パラメーターを更新してもリアルタイムに範囲されないバグがあります。
そして、激重。

![ssao](https://raw.githubusercontent.com/testkun08080/zenn-docs/main/images/react-postshader-195d07b1ed77da/ssao.png)

## n8ao (SSAO)

SSAO 使うなら、こちらを推奨します。
使いやすいし、バグはないかなと思います。

![n8ao](https://raw.githubusercontent.com/testkun08080/zenn-docs/main/images/react-postshader-195d07b1ed77da/n8ao.png)

**[ソースレポ]https://github.com/N8python/n8ao**

## Bloom(ブルーム)

![bloom](https://raw.githubusercontent.com/testkun08080/zenn-docs/main/images/react-postshader-195d07b1ed77da/bloom.png)

## Chromatic aberration(色収差)

![chromatic](https://raw.githubusercontent.com/testkun08080/zenn-docs/main/images/react-postshader-195d07b1ed77da/chromatic.png)

## Wave Distortion（波状歪みエフェクト）

![Wave Distortion Effect](https://raw.githubusercontent.com/testkun08080/zenn-docs/main/images/react-postshader-195d07b1ed77da/wave.png)

## RGB Split（色収差）

![RGB Split Effect](https://raw.githubusercontent.com/testkun08080/zenn-docs/main/images/react-postshader-195d07b1ed77da/rgbsplit.png)

## Kaleidoscope（万華鏡エフェクト）

万華鏡っぽいやつ

![Kaleidoscope Effect](https://raw.githubusercontent.com/testkun08080/zenn-docs/main/images/react-postshader-195d07b1ed77da/kaleidoscope.png)

## Fractal Nois

定番ノイズで歪みを出せるやつです

![Fractal Noise Effect](https://raw.githubusercontent.com/testkun08080/zenn-docs/main/images/react-postshader-195d07b1ed77da/fractal.png)

## Glitch

壊れたモニターでよく使われるやつ

![glitch](https://raw.githubusercontent.com/testkun08080/zenn-docs/main/images/react-postshader-195d07b1ed77da/glitch.png)

## Pixcelation (ドット化)

ドット風モザイク

![pixcelation](https://raw.githubusercontent.com/testkun08080/zenn-docs/main/images/react-postshader-195d07b1ed77da/pixcelation.png)

## Dot screen

漫画、アメコミ風

![dot-s](https://raw.githubusercontent.com/testkun08080/zenn-docs/main/images/react-postshader-195d07b1ed77da/dot-s.png)

## Grid

![grid](https://raw.githubusercontent.com/testkun08080/zenn-docs/main/images/react-postshader-195d07b1ed77da/grid.png)

## Scanline

![scanline](https://raw.githubusercontent.com/testkun08080/zenn-docs/main/images/react-postshader-195d07b1ed77da/scanline.png)

## Outline

選択しているオブジェクトのみにアウトラインをつけたり、隠れていても見えるようにする UX/UI 用エフェクトだと思います。
※内部では固定したオブジェクトを渡しています。

![outline](https://raw.githubusercontent.com/testkun08080/zenn-docs/main/images/react-postshader-195d07b1ed77da/outline.png)

## Edge Outline（エッジ検出エフェクト）

深度バッファと法線バッファを使用してエッジを検出し、アウトラインを描画します。トゥーンシェーディングやセル画風の表現に使えます。

![Edge Outline Effect](https://raw.githubusercontent.com/testkun08080/zenn-docs/main/images/react-postshader-195d07b1ed77da/edge-outline.png)

## Sepia

![sepia](https://raw.githubusercontent.com/testkun08080/zenn-docs/main/images/react-postshader-195d07b1ed77da/sepia.png)

## Brightess contrast

![bright-contrast](https://raw.githubusercontent.com/testkun08080/zenn-docs/main/images/react-postshader-195d07b1ed77da/bright-contrast.png)

## Color Dot

![bright-contrast](https://raw.githubusercontent.com/testkun08080/zenn-docs/main/images/react-postshader-195d07b1ed77da/color-dot.png)

## Avarage Color

![avarage-color](https://raw.githubusercontent.com/testkun08080/zenn-docs/main/images/react-postshader-195d07b1ed77da/color-avarage.png)

## Avarage Color

![avarage-color](https://raw.githubusercontent.com/testkun08080/zenn-docs/main/images/react-postshader-195d07b1ed77da/color-avarage.png)

## Color Shift

![color-shift](https://raw.githubusercontent.com/testkun08080/zenn-docs/main/images/react-postshader-195d07b1ed77da/color-shift.png)

## Tilt shift

![tilt-shift](https://raw.githubusercontent.com/testkun08080/zenn-docs/main/images/react-postshader-195d07b1ed77da/tiltshift.png)

## Tilt shift 2

![tilt-shift2](https://raw.githubusercontent.com/testkun08080/zenn-docs/main/images/react-postshader-195d07b1ed77da/tiltshift2.png)

## Water

![water](https://raw.githubusercontent.com/testkun08080/zenn-docs/main/images/react-postshader-195d07b1ed77da/water.png)

## View Depth Visualization（デプスバッファ可視化）

カメラからの距離（深度）を白黒で視覚化します。デバッグやアート表現に有用で、近いほど黒く、遠いほど白く表示されます。

![vdepth](https://raw.githubusercontent.com/testkun08080/zenn-docs/main/images/react-postshader-195d07b1ed77da/vdepth.png)

## Simple Check Normal（法線ベクトル可視化）

ノーマル可視化用のデバッグ用です
ビューノーマルと、ワールド空間用のノーマルを切り替えてみれます

![checknormal](https://raw.githubusercontent.com/testkun08080/zenn-docs/main/images/react-postshader-195d07b1ed77da/checknormal.png)

## Noise

雰囲気与えるのにノイズは便利です

![checknormal](https://raw.githubusercontent.com/testkun08080/zenn-docs/main/images/react-postshader-195d07b1ed77da/noise.png)

## Vignette

![vignette](https://raw.githubusercontent.com/testkun08080/zenn-docs/main/images/react-postshader-195d07b1ed77da/Vignette.png)

## Tonemap

![tonemap](https://raw.githubusercontent.com/testkun08080/zenn-docs/main/images/react-postshader-195d07b1ed77da/tonemap.png)

## Lut

アセットは[ここ](https://github.com/mrdoob/three.js/tree/master/examples/luts)から引用させていただきました。
地味に LUT テクスチャ達のインポートにテコづりました。

![lut](https://raw.githubusercontent.com/testkun08080/zenn-docs/main/images/react-postshader-195d07b1ed77da/lut.png)

## Ascii エフェクト

文字を使ってサイバーっぽくするやつです

![ascii](https://raw.githubusercontent.com/testkun08080/zenn-docs/main/images/react-postshader-195d07b1ed77da/ascii.png)

---

## ハマったポイント

### 1. View Space → World Space の変換

カメラの`viewMatrix`を使って座標変換します。

```glsl
vec4 worldNormal = vec4(viewNormal, 1.0) * viewMatrix;
```

### 2. 一部ビルトインシェーダーが React19 では正しく動作しない

ビルトインの Godrays, Lensfrare, FXAA などは正しく動作しなかったので、今回は省いています。

## シーン設定について

一般的な、背景の非表示やライトの色などの簡易設定ができます。
![setting](https://raw.githubusercontent.com/testkun08080/zenn-docs/main/images/react-postshader-195d07b1ed77da/setting.png)

---

## まとめ

色々なすでにビルトインされているものもあって手軽に使えてありがたいです。
サンプルも shdertoy や three.js に色々落ちていたりするので、色々遊べます。

カスタムシェーダーも`Effect`クラスを継承するだけでいいんですが、パスの渡し方とかそこら辺が触ってみないと何とも言えない感じでした。
癖がわかれば、そのあとはスイスイ行けるかなぁと思います。

好評や色々な方が見られるようでしたら、工程をもっと細かく砕いて zenn などの本としてまとめてみようかと思います。

何かミスなどがあれば、コメントください〜
では！

---

### 関連リンク

- [React Three Fiber](https://docs.pmnd.rs/react-three-fiber/)
- [Postprocessing Library](https://github.com/pmndrs/postprocessing)
- [Three.js Documentation](https://threejs.org/docs/)
- [Leva Controls](https://github.com/pmndrs/leva)
- [サンプルレポ](https://github.com/testkun08080/react-postprocess-tester)
