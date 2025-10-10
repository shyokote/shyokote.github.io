# YAMAHA RTX1210 Set up automatic route switching in the event of a failure (network backup function)

## 目的
主系経路が障害になった場合に副系経路に自動で切り替わるようにする

## 設定方法

### 経路設定
全ての通信（default）において、正常経路を「tunnel 1」・バックアップを経路「pp 11」としたとき
```bash
ip route default gateway tunnel 1 keepalive 1 gateway pp 11 weight 0
```

### キープアライブの設定 (トリガーの設定)
「10」秒ごとに計「6」カウントを「192.168.1.1」に向けて送信し、応答がなければ、バックアップ経路に切り替える
```bash
ip keepalive 1 icmp-echo 10 6 192.168.1.1
```