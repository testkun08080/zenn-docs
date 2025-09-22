---
title: "FastAPIをVercelへまた置いてみた。官報ビューワーアップデート"
emoji: "🎉"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [python, fastapi, vercel, pdf]
published: false
---

# はじめに
官報ビューワーという簡易RSSビューワーを作成してからおよそ数ヶ月。
ほぼほぼ誰にも使われていない気がするけど、、
ちょいちょい自分で眺めては、ほぉこんなのがあるのかと自分がヘビーユーザー化してしまいました。
PDFもどうせならダウンロードしたいよなぁー。と思ってバックエンドを用意して実装することにしました。
この記事では、簡単にVercel上へのホスティングまでを記します。

## 対象読者
- FastAPIやPythonなどを使ったものをvercel上でホスティングしてみたい方

### 過去の関連記事
https://zenn.dev/testkun08080/articles/githubaction-python-06736ff9d25672
https://zenn.dev/testkun08080/articles/vercel-fastapi-591926e41c4a69

### 何をホスティングしたか
https://github.com/testkun08080/kanpo-viewer/blob/main/api/app/api/pdf.py

抜粋して書くと。
基本的なPDF用の処理クラスを定義
（Claudeに簡単な設計を渡してチョちょっと直したぐらいのコードですが、おそらく大丈夫なはず）

```python
import urllib.request
import tempfile
import os
from typing import Tuple, Optional
from urllib.parse import urlparse
import logging

logger = logging.getLogger(__name__)


class SimplePdfDownloadService:
    """標準ライブラリのみを使用したシンプルなPDFダウンロードサービス"""

    def __init__(self):
        self.max_file_size = 50 * 1024 * 1024  # 50MB制限
        self.timeout = 30  # 30秒タイムアウト

    def download_pdf(self, url: str, filename: Optional[str] = None) -> Tuple[str, int]:
        """
        PDFをダウンロードして一時ファイルとして保存

        Args:
            url: ダウンロードするPDFのURL
            filename: ファイル名（指定がない場合はURLから推測）

        Returns:
            Tuple[str, int]: (一時ファイルパス, ファイルサイズ)

        Raises:
            ValueError: 無効なURLまたはファイルサイズ制限超過
            urllib.error.URLError: HTTP通信エラー
        """
        # ファイル名の決定
        if not filename:
            parsed_url = urlparse(url)
            filename = os.path.basename(parsed_url.path)
            if not filename or not filename.endswith('.pdf'):
                filename = 'download.pdf'

        # .pdf拡張子がない場合は追加
        if not filename.endswith('.pdf'):
            filename += '.pdf'

        try:
            # HTTPリクエストの準備
            req = urllib.request.Request(url)
            req.add_header('User-Agent', 'Mozilla/5.0 (compatible; PDF-Downloader/1.0)')

            # URLを開く
            with urllib.request.urlopen(req, timeout=self.timeout) as response:
                # レスポンスステータスチェック
                if response.status != 200:
                    raise ValueError(f"HTTP error: {response.status}")

                # Content-Typeチェック（可能な場合）
                content_type = response.headers.get('content-type', '')
                if content_type and 'pdf' not in content_type.lower():
                    logger.warning(f"Content-Type is not PDF: {content_type}")

                # ファイルサイズチェック
                content_length = response.headers.get('content-length')
                if content_length and int(content_length) > self.max_file_size:
                    raise ValueError(f"File size too large: {content_length} bytes")

                # 一時ファイルに保存
                temp_file = tempfile.NamedTemporaryFile(
                    delete=False,
                    suffix='.pdf',
                    prefix='pdf_download_'
                )

                total_size = 0
                try:
                    with open(temp_file.name, 'wb') as f:
                        while True:
                            chunk = response.read(8192)
                            if not chunk:
                                break
                            total_size += len(chunk)
                            if total_size > self.max_file_size:
                                raise ValueError(f"File size exceeded limit: {total_size} bytes")
                            f.write(chunk)

                    logger.info(f"PDF downloaded successfully: {filename}, size: {total_size} bytes")
                    return temp_file.name, total_size

                except Exception as e:
                    # エラーが発生した場合は一時ファイルを削除
                    if os.path.exists(temp_file.name):
                        os.unlink(temp_file.name)
                    raise e

        except urllib.error.URLError as e:
            logger.error(f"URL error downloading PDF from {url}: {e}")
            raise
        except Exception as e:
            logger.error(f"Unexpected error downloading PDF from {url}: {e}")
            raise

    def cleanup_temp_file(self, temp_file_path: str) -> None:
        """一時ファイルをクリーンアップ"""
        try:
            if os.path.exists(temp_file_path):
                os.unlink(temp_file_path)
                logger.debug(f"Cleaned up temporary file: {temp_file_path}")
        except Exception as e:
            logger.error(f"Error cleaning up temporary file {temp_file_path}: {e}")
```

ダウンロード用のapiはこんな感じです
```python
from fastapi import APIRouter, HTTPException, BackgroundTasks, Depends
from fastapi.responses import FileResponse
import logging
import os

from ..schemas.pdf import PdfDownloadRequest
from ..services.simple_pdf_service import SimplePdfDownloadService
from ..core.security import verify_api_key

logger = logging.getLogger(__name__)
router = APIRouter()
pdf_service = SimplePdfDownloadService()


async def cleanup_file(file_path: str):
    """バックグラウンドでファイルをクリーンアップ"""
    pdf_service.cleanup_temp_file(file_path)


@router.post("/download", response_class=FileResponse)
async def download_pdf(
    request: PdfDownloadRequest,
    background_tasks: BackgroundTasks,
    authenticated: bool = Depends(verify_api_key)
):
    """
    PDFをダウンロードしてクライアントに返すAPI

    Args:
        request: PDFダウンロードリクエスト（URL、ファイル名）
        background_tasks: バックグラウンドタスク（ファイルクリーンアップ用）

    Returns:
        FileResponse: PDFファイル
    """
    try:
        # PDFをダウンロード
        temp_file_path, file_size = pdf_service.download_pdf(
            str(request.url),
            request.filename
        )

        # ファイル名の決定（リクエストで指定されているか、サービスで決定されたもの）
        filename = request.filename or os.path.basename(temp_file_path)
        if not filename.endswith('.pdf'):
            filename += '.pdf'

        # バックグラウンドでファイルをクリーンアップするタスクを追加
        background_tasks.add_task(cleanup_file, temp_file_path)

        logger.info(f"Serving PDF file: {filename}, size: {file_size} bytes")

        # PDFファイルをクライアントに返す
        return FileResponse(
            path=temp_file_path,
            filename=filename,
            media_type='application/pdf',
            headers={
                "Content-Disposition": f"attachment; filename={filename}",
                "Content-Length": str(file_size)
            }
        )

    except ValueError as e:
        logger.error(f"Validation error: {e}")
        raise HTTPException(status_code=400, detail=str(e))

    except Exception as e:
        logger.error(f"Error downloading PDF: {e}")
        raise HTTPException(
            status_code=500,
            detail="PDF download failed"
        )


@router.get("/health")
async def health_check():
    """ヘルスチェックエンドポイント"""
    return {"status": "healthy", "service": "pdf_download"}

```


#### レポはこちら
https://github.com/testkun08080/kanpo-viewer



### 設定で引っかかったところ

数ヶ月前へこれで問題なかったんですが、
これだとエラーが起きました。

```json
{
    "routes": [
        {
            "src": "/(.*)",
            "dest": "api/app/main.py"
        }
    ],
    "functions": {
        "api/app/main.py": {
            "maxDuration": 60
        }
    }
}
```


以下の設定で一旦回避することができました。
（公式のドキュメントがなんかみずらいので、ベストなものかが微妙です）

```json
{
    "builds": [
        {
            "src": "api/app/main.py",
            "use": "@vercel/python"
        }
    ],
    "routes": [
        {
            "src": "/(.*)",
            "dest": "api/app/main.py"
        }
    ]
}
```

## Vercel CLI を使ってテストからデプロイ

1. **ローカルでテスト**
    ```bash
    vercel dev
   ```

2. **vercelへデプロイ（プレビューとして）**
    ```bash
    vercel
   ```

3. **、vercelへデプロイ（プロダクトとして）**
    ```bash
    vercel --prod
   ```

## まとめ
ご覧いただきありがとうございました。
それでは、良き`vercel fastapi 生活`を。
