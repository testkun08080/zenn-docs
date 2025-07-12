---
title: "USDをビルドして、Viewerをカスタムしてみる"
emoji: "🗂"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [python, cpp ,USD, viewer, 3d]
published: false
---

# はじめに
USD(Universal Scene Description)Viewerを触ることがあったので、色々まとめたいと思います。
会社でUSD Viewerを導入するにあたって、まずは手始めにスタンドアローンでプレビューするのに使ったり。または、モジュールとして組み込むで使える様になどを行っていました。
最初は訳わかんなった（ビルド）ですけど、僕みたいなエンジニアではない人や、USDこれから触ろうとしている人に向けて記録として残しておきます。
ゴールとしては、ビルドしたものをカスタムしてViewerをコントロールするところまでとします。

とりあえず、サンプルのデータ見てみたいという方は、以下レポをご覧ください。
https://github.com/testkun08080/custom-embed-usdviewer

#### 作成される最終イメージ
![イメージ](https://raw.githubusercontent.com/testkun08080/custom-embed-usdviewer/refs/heads/main/docs/sample.gif)

## この記事の流れ
1. Universal Scene Description (USD) とは？
2. ローカルでビルドする(python)
3. USD viewer単独で起動する
4. USD Viewerをカスタムしてみる

## Universal Scene Description (USD) とは？
ざっくり言えば、、、*複雑になった3Dアセットのやり取りを、USDが中間に入って円滑にしよう!*
という解釈でいいと思います。以下「USDとは」を公式から翻訳したもの。

> 映画やゲームなどのコンピューターグラフィックスを制作するパイプラインでは、大量の3Dデータ（シーン記述）を生成、保存、送信する必要があります。これらのデータは、モデリング、シェーディング、アニメーション、ライティング、エフェクト、レンダリングなど、各アプリケーションの特定のニーズに応じてカスタマイズされており、他のアプリケーションでは読み込みや編集ができないことが一般的です。
> 
> Universal Scene Description (USD) は、この課題を解決するために初めて公開されたソフトウェアであり、複数の要素で構成された3Dシーンを効率的かつスケーラブルに共有し、拡張するための強力なツールです。
> 
> USDはモデルやアニメーションなどの要素資産を交換するだけでなく、それらを仮想セット、シーン、ショット、ワールドにまとめ、各アプリケーション間で送信し、破壊的でない方法で編集することも可能です。さらに、単一のAPIとシーングラフを使用して、3Dジオメトリ、シェーディング、ライティング、物理シミュレーションなど、さまざまなグラフィック関連ドメインを迅速にプレビューしたり編集したりするための豊富なツールセットを提供します。
> 
> 特に、USDのコアであるシーングラフとコンポジションエンジンは、特定のドメインに依存しない設計となっており、他のデータドメインを持続可能な形で拡張することも可能です。
> 
> USDはオープンソースプロジェクトであり、TOSTライセンスのもとで提供されています。[^1]

[^1]:USDとは(https://openusd.org/release/intro.html#what-is-usd)

## 開発環境
- macOS Sequoia 15.5
- VsCode
- zsh 5.9 (arm64-apple-darwin24.0)

## ローカルでビルドしてみる

- まずはローカルへクローンまたはサブモジュールを行います
  サブモジュールを使う場合
    ```zsh
    git submodule add https://github.com/PixarAnimationStudios/OpenUSD OpenUSD         
    ```
    or

  クローン
    ```zsh
    git clone https://github.com/PixarAnimationStudios/OpenUSD.git        
    ```

- UVを使用してVENVの作成し、ビルド開始
  よく公式の[ドキュメント](https://github.com/PixarAnimationStudios/OpenUSD.git)をご覧になって、ビルドしてみてください。
  ```zsh
  uv init -p 3.11
  uv add PyOpenGL PySide6 numpy
  uv run OpenUSD/build_scripts/build_usd.py BuildUSD
  ```
  ビルドが正しく終われば以下のようなログが見れるはずです。
  ```zdh
  Success! To use USD, please ensure that you have:

    The following in your PYTHONPATH environment variable:
    /Users/Username/path/custom-embed-usdviewer/BuildUSD/lib/python

    The following in your PATH environment variable:
    /Users/Username/path/custom-embed-usdviewer/BuildUSD/bin
  ```

- 環境パスの設定
  
  PYTHONPATHの設定
    ```zsh
    cat > .env <<EOL
    PYTHONPATH=./BuildUSD/lib/python
    EOL
    ```
  
  PATHの設定
    ```zsh
    export PATH=./BuildUSD/bin:$PATH
    ```


## USD viewer単独で起動する
以下コマンドで、USDViewerが立ち上がるはずです。(初回は起動に時間がかかるはずです)
- uvでusdviewを直接起動
  ```zsh
  uv run --env-file=.env usdview OpenUSD/extras/usd/tutorials/convertingLayerFormats/Sphere.usda
  ```

Python経由で起動する場合
- pythonファイル経由で起動
  - ファイル作成
    ```zsh
    cat > open_usd_viewer.py <<EOL
    """This script is a simple USD viewer using the pxr.Usdviewq module."""
    import sys
    import pxr.Usdviewq as Usdviewq
    
    if __name__ == "__main__":
        # Let Ctrl-C kill the app.
        import signal

        signal.signal(signal.SIGINT, signal.SIG_DFL)

        try:
            usd_path = "assets/Sphere.usda"
            sys.argv.append(usd_path)
            Usdviewq.Launcher().Run()
        except Usdviewq.InvalidUsdviewOption as e:
            print("ERROR: " + str(e), file=sys.stderr)
            sys.exit(1)
    EOL
    ```

  - 作成後、起動
    ```zsh
    uv run open_usd_viewer.py
    ```

## USD Viewerをカスタムしてみる

### usdviewerを埋め込んだwidgetを作成
まずは、usdviewerを埋め込んだwidgetを作成します。
コードを見ていただければわかると思いますが、`from pxr.Usdviewq.stageView import StageView`が重要なところかと思います。
あとラッパー関数を作っておいて、後に記述するUI側で操作する感じです。
**以下コマンドをコピーしてファイルを作成できます**

:::details embed_usd_widget.py
```python
cat > embed_usd_widget.py <<EOL
"""A simple application to embed a USD viewer using PySide6."""

import os
import sys
from datetime import datetime
from PySide6 import QtCore, QtWidgets
from pxr import Usd, UsdUtils
from pxr.Usdviewq.stageView import StageView, RenderModes


class EmmbedUSDWidget(QtWidgets.QWidget):
    """A widget to embed USD stage viewer."""

    def __init__(self, stage=None):
        super(EmmbedUSDWidget, self).__init__()
        self.model = StageView.DefaultDataModel()
        self.stage = stage
        self.view = StageView(dataModel=self.model)

        layout = QtWidgets.QVBoxLayout(self)
        layout.addWidget(self.view)
        layout.setContentsMargins(0, 0, 0, 0)

        if self.stage:
            self.set_stage(self.stage)

        # Reset View
        self.view.updateView(resetCam=True, forceComputeBBox=True)

    def image_save(self):
        """Save the current view as an image file."""
        print("GetRendererAovs", self.view.GetRendererAovs())
        image = self.view.grabFramebuffer()
        timestamp = datetime.now().strftime("%Y-%m-%d_%H%M%S")
        os.makedirs("screenshots", exist_ok=True)
        filename = f"screenshots/screenshot_{timestamp}.png"
        image.save(filename)

    def set_stage(self, stage):
        """Set the USD stage for the viewer

        Args:
            stage (QGLWidget): USD stage to set in the viewer.

        """
        self.model.stage = stage

    def set_render_mode(self, mode):
        """Set the render mode for the viewer.

        Args:
            mode (str): The render mode to set.
        """
        if mode in RenderModes:
            self.model.viewSettings.renderMode = mode
        else:
            print(f"Invalid render mode: {mode}")

    def reset_camera(self):
        """Resets the camera to fit the geometry in the view."""
        self.view.updateView(resetCam=True, forceComputeBBox=True)

    def show_hud(self, enable=True):
        """Show the HUD (Heads-Up Display) in the viewer.

        Args:
            enable (bool, optional): A value will handle HUD . Defaults to True.
        """

        self.model.viewSettings.showHUD = enable

    def show_bboxes(self, enable=True):
        """Show bounding boxes in the viewer.

        Args:
            enable (bool, optional): A value will handle bounding boxes. Defaults to True.
        """

        self.model.viewSettings.showBBoxes = enable

    def closeEvent(self, event):
        """Close envet handler to ensure proper cleanup.

        Args:
            event (QtGui.QCloseEvent): The close event triggered by the widget.
        """

        # Ensure to close the renderer to avoid GlfPostPendingGLErrors
        self.view.closeRenderer()

if __name__ == "__main__":
    app = QtWidgets.QApplication([])

    import signal

    signal.signal(signal.SIGINT, signal.SIG_DFL)

    path = "assets/Sphere.usda"

    with Usd.StageCacheContext(UsdUtils.StageCache.Get()):
        stage = Usd.Stage.Open(path)

    window = EmmbedUSDWidget(stage)
    window.setWindowTitle("USD Viewer")
    window.resize(QtCore.QSize(750, 750))
    window.show()

    sys.exit(app.exec_())
EOL
```
:::

### 作ったwidgetを使ってカスタムviewerを作る
カスタムといっても、ドッキング可能なwidgetから、viewerのUIをラップー関数を使って操作する簡単なwindowを作ります。
構造としては、大まかなUIファイルで作成しておいて、埋め込みwidgetを使っているだけです。

#### UIファイル
以下イメージのようにQTCreatorなどでファイルを作成します。
![イメージ](/images/python-usdview-a51e8bdf7222cf/ss-a.png)
uiファイルの詳細はgit内のファイルをご確認ください。
https://github.com/testkun08080/custom-embed-usdviewer/blob/main/UI/usdViewerController.ui

**以下コマンドをコピーしてUIファイルをダウンロードしできます**
```zsh
mkdir -p UI && curl -L -o UI/usdViewerController.ui https://raw.githubusercontent.com/testkun08080/custom-embed-usdviewer/main/UI/usdViewerController.ui
```

#### UIファイルを使って埋め込み用usdviewer widgetを操作する

**以下コマンドをコピーしてファイルを作成できます**

:::details app.py
```python
cat > app.py <<EOL
"""Module providing a USD viewer application."""

import sys
from PySide6 import QtWidgets
from PySide6.QtUiTools import QUiLoader
from PySide6.QtCore import QFile, QIODevice
from pxr import Usd, UsdUtils
from pxr.Usdviewq.stageView import RenderModes

from embed_usd_widget import EmmbedUSDWidget


def get_ui_widget(ui_file_name):
    """Loads and returns a UI widget from a UI file.

    Args:
        ui_file_name (str): The path to the UI file to load.

    Returns:
        QWidget: The loaded UI widget.
    """
    ui_file = QFile(ui_file_name)
    if not ui_file.open(QIODevice.ReadOnly):
        print(f"Cannot open {ui_file_name}: {ui_file.errorString()}")
        sys.exit(-1)
    loader = QUiLoader()
    ui_widget = loader.load(ui_file)
    ui_file.close()

    return ui_widget


class EmbedUsdViewerController:
    """Controller class for the Embed USD viewer.

    Manages the loading of USD stages and the viewer widget.
    """

    def __init__(self, usd_file=None):
        """Initializes the USD viewer controller.

        Args:
            usd_file (str): USD file
        """

        self.ui = get_ui_widget("UI/usdViewerController.ui")

        # Load USD Stage
        with Usd.StageCacheContext(UsdUtils.StageCache.Get()):
            self.stage = Usd.Stage.Open(usd_file)

        # Embed USD Viewer widget
        self.usd_widget = EmmbedUSDWidget(self.stage)
        self.ui.main_layout.addWidget(self.usd_widget)

        # Connect Save Image button event
        self.ui.save_image_button.clicked.connect(self.usd_widget.image_save)

        # Connect Reset Camera button event
        self.ui.reset_camera_button.clicked.connect(self.usd_widget.reset_camera)

        # Connect Render Mode comboBox event
        self.ui.shading_mode_combobox.addItems([mode for mode in RenderModes])
        self.ui.shading_mode_combobox.setCurrentText(self.usd_widget.model.viewSettings.renderMode)
        self.ui.shading_mode_combobox.currentTextChanged.connect(self.usd_widget.set_render_mode)

        # Connect HUB checkBox event
        self.ui.show_hud_checkbox.setChecked(self.usd_widget.model.viewSettings.showHUD)
        self.ui.show_hud_checkbox.clicked.connect(self.usd_widget.show_hud)

        # Connect BBoxes checkBox event
        self.ui.show_bbxs_checkbox.setChecked(self.usd_widget.model.viewSettings.showBBoxes)
        self.ui.show_bbxs_checkbox.clicked.connect(self.usd_widget.show_bboxes)

    def setup_aov_ui(self):
        """Sets up the UI for AOVs selection.

        Retrieves the AOVs list and adds them to a combo box.
        """

        # Retrieve and add AOVs list to combo box
        aovs = self.usd_widget.view.GetRendererAovs()
        if not aovs:
            # Add default AOVs if the list is empty
            default_aovs = ["color", "primId", "depth", "Neye"]
            for aov in default_aovs:
                self.ui.aovs_combobox.addItem(aov)
        else:
            for aov in aovs:
                self.ui.aovs_combobox.addItem(aov)

        # Connect AOV selection event
        self.ui.aovs_combobox.currentTextChanged.connect(self.on_aov_changed)

    def on_aov_changed(self, aov_name):
        """Updates the view based on the selected AOV.

        Args:
            aov_name (str): The name of the selected AOV.
        """

        # Update view to the selected AOV
        if aov_name:
            self.usd_widget.view.SetRendererAov(aov_name)
            self.usd_widget.view.update()


if __name__ == "__main__":
    app = QtWidgets.QApplication(sys.argv)

    viewer_controller = EmbedUsdViewerController("assets/Sphere.usda")
    viewer_controller.ui.show()

    # You might need to call this after shop up the window
    viewer_controller.setup_aov_ui()
    sys.exit(app.exec_())
EOL
```
:::

app.pyとuiファイルの作成が終わったら、以下コマンドで起動してみてください。
```zsh
uv run app.py
```

うまくいけば、以下のようなUIが起動するはずです。
![最終形態](https://raw.githubusercontent.com/testkun08080/custom-embed-usdviewer/refs/heads/main/docs/sample_image.png)


## まとめ、感想
最後までご覧いただきありがとうございました。
各会社、独自でゲームエンジンなどを持つところは、これを使ってプレビューツールとして外部へ渡したりなども可能かなと考えています。
(大変そうだけども)
いいね頂けると一日ハッピーです。

## 参考文献
- 日本語での情報はかなり少なくて、こちらのページはUSDを理解するのに非常に参考になりました。
  https://fereria.github.io/reincarnation_tech/usd/what_is_usd

- スタンドアローンのusdviewerをwidgetに埋め込んで使用するのに非常に助かりました
  https://gist.github.com/BigRoy/5ac50208969fdc69a722d66874faf8a2#file-usdviewport_qt-py

- 公式
  https://openusd.org/release/index.html