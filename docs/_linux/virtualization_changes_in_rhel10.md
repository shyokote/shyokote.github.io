# Virtualization changes in RockyLinux10 (RHEL10)

## 問題
ProxMoxでRockyLinux10をそのままインストールしようとしたら、
インストーラーでKernel Panicが発生して進まない

## 原因
CPUの種別が x86-64-v2-AESがデフォルトだったことが原因

RHEL10系から

??? Note
	参考：
		https://docs.redhat.com/ja/documentation/red_hat_enterprise_linux/10/html/considerations_in_adopting_rhel_10/virtualization

			レガシー CPU モデルが削除される
			RHEL 9 で非推奨となった多数の CPU モデルは、RHEL 10 ではサポートされなくなり、仮想マシン (VM) で使用できなくなりました。削除されたモデルは次のとおりです。
			* Intel の場合: Intel Xeon 55xx および 75xx プロセッサーファミリー (Nehalem とも呼ばれます) より前のモデル
			* AMD の場合: AMD Opteron G4 より前のモデル
			* IBM Z の場合: IBM z14 より前のモデル


??? Note
	参考：
	https://access.redhat.com/solutions/7066628

	Red Hat will upgrade the instruction set architecture (ISA) baseline to the x86-64-v3 microarchitecture level in RHEL 10 and x86-64-v1 and x86-64-v2 x86-64 microarchitecture level of CPUs will be marked deprecated in RHEL 8 and RHEL 9 and unsupported in RHEL 10.

Noteにあるようなことになっているため、仮想マシンを作成するときに CPUの種別を x86-64-v3 で作成する必要がある。
これは、RHEL10互換OSであれば全てで発生します。