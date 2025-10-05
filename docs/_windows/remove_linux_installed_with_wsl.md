# Unable to remove Linux installed with WSL

## 目的
WSLでインストールしたLinuxが削除できないときがある

## 削除方法
```bash
wsl --unregister <DistributionName>
```
例：
```bash
wsl --unregister kali-linux
```
## 参考
https://learn.microsoft.com/ja-jp/windows/wsl/
