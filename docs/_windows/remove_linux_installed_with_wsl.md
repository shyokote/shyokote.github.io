# Unable to remove Linux installed with WSL

## 目的
WSLでインストールしたLinuxが削除できないときがある

## 削除方法
```linenums="0"
wsl --unregister <DistributionName>
```
例：
```linenums="0"
wsl --unregister kali-linux
```
## 参考
https://learn.microsoft.com/ja-jp/windows/wsl/
