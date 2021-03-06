# Webスクレイピング-開始処理

***

## Webページから情報を取得する方法

* IEを起動して対象のWebページから情報を取得する

  ```PowerShell
  # 対象ページのURLを設定する
  $url = "https://d4c-lt.com"

  # IEを起動する
  $ie = New-Object -ComObject InternetExplorer.Application

  # IEを表示する
  $ie.Visible = $true

  # 対象のWebページへ移動する
  $ie.Navigate($url)

  # ページが切り替わるまで待つ
  while($ie.Busy) { Start-Sleep -milliseconds 100 }

  # ドキュメントオブジェクトを取得する
  $doc = $ie.document
  ```

* すでに起動しているIEのWebページから情報を取得する

  ```PowerShell
  # 対象ページのURLを設定する
  $url = "https://d4c-lt.com"

  # シェルを取得する
  $shell = New-Object -ComObject Shell.Application

  # IEで開いているページ一覧を取得する
  $ieList = @($shell.Windows() | Where-Object { $_.Name -match "Internet Explorer" })

  # URL指定でIEページのオブジェクトを取得する
  # ([-1]で同一ページの最新タブのオブジェクトを取得する)
  $ie = @($ieList | Where-Object { $_.LocationURL -match $url })[-1]

  # ドキュメントオブジェクトを取得する
  $doc = $ie.document
  ```
