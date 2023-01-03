# WSL install に関するトピック

## デフォルトのインストール

- `PowerShell` または `Windowsコマンドプロンプト` を管理者モードで開き `wsl --install` コマンドを入力

```
wsl --install
```

デフォルトでは、カレントの `Ubuntu` がインストールされる。
Windows11 における、2023/1/3 時点でのデフォルトは　`Ubuntu 22.04` となった。

## WSLインスタンスの格納場所の変更

デフォルトでは、UserProfile 以下に格納されるので、それ以外の場所にインストールしたい場合には、以下のようにすればよい。
データそのものの移動と設定の変更の２ステップが必要。

### データの移動

```
mkdir D:\WSL\images
mkdir D:\WSL\instances\<newDistroName>
cd D:\WSL

wsl -l -v
wsl --export <distroname> .\images\<distroname>.tar
wsl --unregister <distroname>
wsl --import <newDistroName> .\instances\<newDistroName> .\images\<distroname>.tar
wsl --set-default <newDistroName>
```
- `<distroname>` には格納場所を変えたいディストリビューションを、 `<newDistroName>` には新しい格納場所につけるディストリビューション名を指定する。
- `D:\WSL\images` は一時的にイメージを保存する場所。不要になれば削除してOK。
- `D:\WSL\instances\<newDistroName>` には新たな格納場所を指定すること。
- `wsl -l -v` で移動したいディストリビューション名を確認する。ここで確認した名前を、`<distroname>` にセットすること。
- `wsl --unregister <distroname>` は `<distroname>` と `<newDistroName>` を同じにする場合、上記の位置での実行が必要。
  - `<distroname>` と `<newDistroName>` を異なるものにする場合は、作業完了後の実行でもOK。必要に応じて実行すればよい。

ここで終わったら 、一旦 `Windows Terminal`を再起動しておく。

### 設定の変更

移動しただけでは、ユーザは `root`、起動時のディレクトリが、`/mnt/c/home/<username>`になっている。
それらを適切な値に修正する。

`Windows Terminal` の「設定」（`settings.json`）を開き、`startingDirectory` のフィールドを追加する。

```
            {
                "guid": "{割愛}",
                "hidden": false,
                "name": "<newDistroName>",
                "source": "Windows.Terminal.Wsl",
                "startingDirectory": "\\\\wsl$\\<newDistroName>\\home\\<username>"
            }
```

念のため、`Windows Terminal`を再起動した後、`Power Shell`で下記のコマンドを実行する。

```
Function WSL-SetDefaultUser ($distro, $user) { Get-ItemProperty Registry::HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Lxss\*\ DistributionName | Where-Object -Property DistributionName -eq $distro | Set-ItemProperty -Name DefaultUid -Value ((wsl -d $distro -u $user -e id -u) | Out-String); };
WSL-SetDefaultUser <newDistroName> <username>
```

これで完了。

## おまけ

`Ubuntu 22.04`以外のディストリビューションをインストールする方法。

- `wsl --list --online` を実行して使用可能なディストリビューションの一覧を表示
- `wsl --install -d <DistroName>` を実行してディストリビューションをインストール

ほかのディストリビューションも上記の手順を踏めば、格納先を変更することができる。

## 参考にした Webページ

- https://learn.microsoft.com/ja-jp/windows/wsl/install
- https://superuser.com/questions/1714345/change-of-wsl-installation-location
- https://www.aise.ics.saitama-u.ac.jp/~gotoh/HowToReplaceWSL.html
