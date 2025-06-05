# ★WSLエラー対処
## ❇️症状
　Ubuntuを起動しても起動完了せずエラーメッセージが出る<br>

## ❗エラーメッセージ

```
 　　仮想マシンまたはコンテナーからの応答が受信されなかったため、操作がタイムアウトしました。
 　　エラー コード: Wsl/Service/HCS_E_CONNECTION_TIMEOUT

 　　[プロセスはコード 4294967295 (0xffffffff) で終了しました]
 　　このターミナルを Ctrl+D で閉じるか、Enter キーを押して再起動できます。
 　　仮想マシンまたはコンテナーからの応答が受信されなかったため、操作がタイムアウトしました。
 　　エラー コード: Wsl/Service/HCS_E_CONNECTION_TIMEOUT

 　　[プロセスはコード 4294967295 (0xffffffff) で終了しました]
 　　このターミナルを Ctrl+D で閉じるか、Enter キーを押して再起動できます。
  ```

### ❓エラー内容
　これは WSL（Windows Subsystem for Linux） が立ち上がろうとしたけど、<br>
　その裏で動いている「仮想マシン（VM）」や「コンテナ」からの返事がこなくて、タイムアウト（応答なし）になったという意味<br>
<br>
## ✅対処法
### ・WSL と仮想マシンをリセットする
　**PowerShellを管理者として起動して、以下を実行**<br>
```
 　　wsl --shutdown
```

　そのあと、再びターミナル（Ubuntuなど）を開く<br>
<br>
<br>
### ・.wslconfig によるリソース制限（補足）
　PC全体のメモリの余裕が無いからとも考えられるので、WSLの使用可能メモリを制限して安定化させる

 　💠やり方<br>
  　　１．📁 .wslconfig の作成<br>
    　　　リソース制限の設定を書き込むためのテキストファイルを次の場所に作成する<br>
```
            C:\Users\<ユーザー名>\.wslconfig      #拡張子は.wslconfigにする　.txtは消す
```

  　　　　❗Ubuntu（WSL内）に .wslconfig を作っても一切効果がないので注意！<br>
  　　　　  必ず C:\Users\ユーザー名 の Windows 側に配置すること。<br>


  　　２．📁 .wslconfig の設定内容<br>
    　　　次の内容を作成したファイルに書く<br>
       
```
           [wsl2]
           memory=2GB
           processors=2
           swap=0
           localhostForwarding=true
```
　
     | 設定項目                  | 例      | 意味                             |
| --------------------- | ------ | ------------------------------ |
| `memory`              | `2GB`  | WSLの最大メモリ量（上限）                 |
| `processors`          | `2`    | 使用するCPUコア数の上限                  |
| `swap`                | `0`    | スワップファイル（仮想メモリ）のサイズ。0で無効化可能    |
| `localhostForwarding` | `true` | Windows↔WSL間のポート転送（普通はtrueでOK） |

<br>
    　　　❗メモリ制限がギリギリの状態でswapがないとOOM（Out of Memory）エラーが出やすくなる可能性あり<br>
    　　　　　memory=2GB 程度あれば swap=0 でも十分安定します。<br>
    　　　　　ただし memory=1GB以下 だと swap=2GB くらい設定しておくのが安全です。<br>
<br>
<br>

  　　３．WSLを再起動させる<br>
    　　　**PowerShellを管理者として起動して、以下を実行**<br>
```
           wsl --shutdown
```

   　　　そのあと、再びターミナル（Ubuntuなど）を開く<br> 


  　　４．設定内容が反映されているか確認<br>
   　　　WSL内(Ubuntuとか)で次のコマンドを実行する<br>
  ```
# メモリ確認
cat /proc/meminfo | grep MemTotal
# 出力例
MemTotal:        1950484 kB   → 約2GB（設定通り！）

# CPUコア数確認
cat /proc/cpuinfo | grep processor
# 出力例
processor       : 0
processor       : 1           → 0〜1で2コア（設定通り！）

```

### ・Windows Updateをする
　Windows Updateの処理が完了していないのが原因の可能性があるので<br>
　Updateを完了させる<br>
 
　参考サイト：[Windows Update が完了しているか確認する方法](https://f-security.jp/v6/support/faq/200101.html)
　
