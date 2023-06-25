# OS勉強録

x86_64アーキテクチャの自作OSを作成するにあたっての勉強録😓 \
勉強会で発表した資料や作業録をまとめていく

## 内容

### ブートローダー

### ページング
- [物理アドレスと物理アドレス](./paging/address.md)
- [ページング](./paging/paging.md)
- [作業録](./paging/worklog.md)

### Local APIC
- [社内勉強会発表資料 - Local APIC Timerを用いて時間計測機能を実装](./documents/local_apic_timer_study.pdf)
- [勉強会用リポジトリ](https://github.com/junyaU/time-measurement-using-Local-APIC)


### システムコール
- [システムコールとは](./syscall/syscall.md)
- [ユーザーモードとカーネルモード](./syscall/mode.md)

## 参考
Intel® 64 and IA-32 Architectures SDM \
https://software.intel.com/content/www/us/en/develop/download/intel-64-and-ia-32-architectures-sdm-combined-volumes-1-2a-2b-2c-2d-3a-3b-3c-3d-and-4.html

コンピューターシステムの理論と実装 - モダンなコンピュータの作り方 \
https://www.oreilly.co.jp/books/9784873117126/

自作エミュレータで学ぶx86アーキテクチャ-コンピュータが動く仕組みを徹底理解! \
https://www.amazon.co.jp/%E8%87%AA%E4%BD%9C%E3%82%A8%E3%83%9F%E3%83%A5%E3%83%AC%E3%83%BC%E3%82%BF%E3%81%A7%E5%AD%A6%E3%81%B6x86%E3%82%A2%E3%83%BC%E3%82%AD%E3%83%86%E3%82%AF%E3%83%81%E3%83%A3-%E3%82%B3%E3%83%B3%E3%83%94%E3%83%A5%E3%83%BC%E3%82%BF%E3%81%8C%E5%8B%95%E3%81%8F%E4%BB%95%E7%B5%84%E3%81%BF%E3%82%92%E5%BE%B9%E5%BA%95%E7%90%86%E8%A7%A3-%E5%86%85%E7%94%B0%E5%85%AC%E5%A4%AA/dp/4839954747

ゼロからのOS自作入門 \
https://www.amazon.co.jp/%E3%82%BC%E3%83%AD%E3%81%8B%E3%82%89%E3%81%AEOS%E8%87%AA%E4%BD%9C%E5%85%A5%E9%96%80-%E5%86%85%E7%94%B0-%E5%85%AC%E5%A4%AA/dp/4839975868

Introduction to Paging \
https://os.phil-opp.com/paging-introduction/

Hello uchan world, Local APICタイマー入門（PDF） \
https://booth.pm/ja/items/1319521

Microsoft ACPI system description tables \
https://learn.microsoft.com/en-us/windows-hardware/drivers/bringup/acpi-system-description-tables

EDKⅡ UEFI Driver Writer’s Guide \
https://edk2-docs.gitbook.io/edk-ii-uefi-driver-writer-s-guide/

https://wiki.osdev.org/ACPI_Timer

https://wiki.osdev.org/APIC





