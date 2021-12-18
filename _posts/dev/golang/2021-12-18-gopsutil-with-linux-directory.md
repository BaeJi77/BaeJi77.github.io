---
title: "System information (metrics)에 대해서 (+ gopsutil에 대해서)"
description: ""

categories: 
  - dev
  - golang
tags:
  - dev
  - golang
  - goutils
  - linux directory
---

- [예제 소스코드](https://github.com/BaeJi77/blog-code/tree/main/2021-12/gopsutil)

# system information

모든 컴퓨터들은 cpu, memory, disk와 같은 정보을 가지고 있습니다. 그뿐만 아니라 cpu 모델명부터해서 현재 동작하고 있는 Process 숫자까지 모두 OS system이 동작하는데 제공해줄 수 있는 정보입니다. 

## 해당 정보들은 왜 필요할까?

서버 한대에 하나의 어플리케이션이 동작하고 있다고 생각해보자. 해당 어플리케이션의 사용량에 따라 cpu 사용량이 크게 자지우지 될 것입니다. 그 뿐만 아니라 하나의 서버에 여러 어플리케이션을 동작한다고 해도 cpu 샤용량, 메모리 사용량은 여러 어플리케이션 모두에게 영향을 줄 수 있는 수치입니다. 

그뿐만 아니라 이런 정보들은 어플리케이션에 문제가 발생했을 경우 어떤 문제인지에 대해서 알려줄 수도 있습니다. 어플리케이션의 로그가 너무 많아서 disk 사용을 하지 못하고 어플리케이션이 멈출수도 있으며 메모리 사용으로 인해서 OOM이 발생할 수도 있습니다.

그러기 때문에 특정 서버에 대해서 시스템 정보를 주기적으로 확인하는 것은 모니터링하는데 있어서 중요합니다.

# 이런 정보들은 어떻게 얻을까?

여러 방법들이 있겠지만 오늘 제가 소개하려고 하는 것은 간단하게 시스템 커맨드라인으로 정보를 얻는 방법과 golang 라이브러리를 통해서 얻는 방법을 소개해드리겠습니다.

## 시스템 명령어로 정보 획득하기

### cpu 

cpu 정보와 같은 것들에 대해서 `top` 이라는 명령어가 유명한 것으로 알고 있습니다. 

검색을 해보니 여러가지 새로운 것들도 있어서 같이 소개합니다.
- vtop (https://github.com/MrRio/vtop)
- gtop (https://github.com/aksakalli/gtop)
- gotop (https://github.com/cjbassi/gotop)
- htop (https://htop.dev/)

![gotop](/assets/images/2021-12-18-gopsutil/demo.gif)
출처: `gotop` github


위에 있는 사진은 `gotop`이 실제로 동작하고 있는 모습입니다. commandline에서 실행하게 되면 실시간으로 정보를 얻을 수 있습니다.

해당 commandline은 cpu말고 다양한 정보도 같이 제공합니다!

### 메모리

`free` 라는 명령어가 있습니다.

``` bash
$ free
              total        used        free      shared  buff/cache   available
Mem:        1014656      238236      316100         272      460320      756196
Swap:       2097148      161536     1935612
```

위에 명령어와 별개로 `top` 명령어를 통해서 가져올 수도 있습니다. [더 보기](https://zetawiki.com/wiki/%EB%A6%AC%EB%88%85%EC%8A%A4_%EB%A9%94%EB%AA%A8%EB%A6%AC_%EC%82%AC%EC%9A%A9%EB%A5%A0_%ED%99%95%EC%9D%B8)에 더욱 다양한 방법에 대해서 소개합니다!!

### disk

`df` 라는 명령어가 있습니다.

``` bash
$ df
Filesystem     1K-blocks    Used Available Use% Mounted on
devtmpfs          496892       0    496892   0% /dev
/dev/vda2       10472448 7345368   3127080  71% /
...
```

현재 마운트되어 있는 정보들에 대해서 정보를 보여줍니다. 해당 명령어에 대해서 궁금하다면 [더 보기](https://www.ibm.com/docs/ko/aix/7.2?topic=system-displaying-available-space-file-df-command)를 읽어보세요!

## gopsutil

명령어로 시스템 정보들을 얻을 수 있습니다. 하지만 직접 호스트에 접근해서 항상 정보를 얻기는 쉽지 않습니다. 그러다보니 해당 데이터를 수집를 해서 보고 싶은 needs가 생길 수 있습니다. 

쿠버네티스에서는 프로메테우스와 node-exporter와 같은 컴포넌트들이 해당 역할을 수행해줍니다. 하지만 그런 환경이 아니라서 직접 시스템 정보를 수집해야됩니다. 

이런 요구사항을 해결하기 위해서 여러 언어에서 해당 기능을 제공하는 라이브러리들이 있습니다. golang에서는 `gopsutil`이라는 시스템 정보들을 잘 제공해줄 수 있도록 만들었습니다. 

특징으로는 golang의 장점 중 하나로 cross compile 이기 때문에 여러 os에서 지원해주기 때문에 여러 os에서 동작하도록 가능합니다. (os마다 지원하는 기능이 다릅니다.)

``` go
package main

import (
	"fmt"

	"github.com/shirou/gopsutil/cpu"
	"github.com/shirou/gopsutil/mem"
)

func main() {
	memory, _ := mem.VirtualMemory()
	fmt.Printf("Total: %d, Free:%d, UsedPercent:%f%% \n", memory.Total, memory.Free, memory.UsedPercent)

	// convert to JSON. String() is also implemented
	fmt.Println(memory)

	fmt.Println()

	info, _ := cpu.Info()
	for _, stat := range info {
		fmt.Println(stat)
	}
}

---

Total: 34359738368, Free:202891264, UsedPercent:55.978644% 
{"total":34359738368,"available":15125622784,"used":19234115584,"usedPercent":55.97864389419556,"free":202891264,"active":14810374144,"inactive":14922731520,"wired":3149877248,"laundry":0,"buffers":0,"cached":0,"writeback":0,"dirty":0,"writebacktmp":0,"shared":0,"slab":0,"sreclaimable":0,"sunreclaim":0,"pagetables":0,"swapcached":0,"commitlimit":0,"committedas":0,"hightotal":0,"highfree":0,"lowtotal":0,"lowfree":0,"swaptotal":0,"swapfree":0,"mapped":0,"vmalloctotal":0,"vmallocused":0,"vmallocchunk":0,"hugepagestotal":0,"hugepagesfree":0,"hugepagesize":0}

{"cpu":0,"vendorId":"GenuineIntel","family":"6","model":"158","stepping":10,"physicalId":"","coreId":"","cores":6,"modelName":"Intel(R) Core(TM) i5-8500 CPU @ 3.00GHz","mhz":3000,"cacheSize":256,"flags":["fpu","vme","de","pse","tsc","msr","pae","mce","cx8","apic","sep","mtrr","pge","mca","cmov","pat","pse36","clfsh","ds","acpi","mmx","fxsr","sse","sse2","ss","htt","tm","pbe","sse3","pclmulqdq","dtes64","mon","dscpl","vmx","smx","est","tm2","ssse3","fma","cx16","tpr","pdcm","sse4.1","sse4.2","x2apic","movbe","popcnt","aes","pcid","xsave","osxsave","seglim64","tsctmr","avx1.0","rdrand","f16c","rdwrfsgs","tsc_thread_offset","sgx","bmi1","hle","avx2","smep","bmi2","erms","invpcid","rtm","fpu_csds","mpx","rdseed","adx","smap","clfsopt","ipt","sgxlc","mdclear","tsxfa","ibrs","stibp","l1df","ssbd","syscall","xd","1gbpage","em64t","lahf","lzcnt","prefetchw","rdtscp","tsci"],"microcode":""}

```

위에 결과처럼 시스템 정보들을 얻을 수 있습니다. 이것뿐만 아니라 `disk`, `process` 외에 여러 정보들을 얻을 수 있습니다. 

그리고 위에 결과에서 알 수 있듯이 `String()`을 통해서 json 형태로 출력되는 것을 볼 수 있습니다. 이것을 이용하면 어떤 변환없이 바로 외부로 요청을 보낼 수도 있을 것 같습니다.

# 과연 정보를 어떻게 가져올까?

저도 구체적으로 알지는 못하지만 공부를 하고 실제 내부 구현체를 보다보니 linux 디렉토리 구조에 알게되었습니다.

`/proc`이라는 디렉토리가 있는데 해당 디렉토리는 process와 관련된 정보를 제공합니다. 하지만 단순히 프로세스 정보뿐만 disk정보와 메모리 정보를 가지고 있습니다.

그 뿐만 아니라 여러 디렉토리가 특성있게 시스템 정보들을 저장하고 있습니다.

[gopsutil 코드](https://github.com/shirou/gopsutil/blob/c6bccaff3b394c7108aa522999111214fe541bd4/internal/common/common.go#L331-L353)를 보게 되면 해당 디렉토리마다 저장한 데이터가 있고 그 데이터들을 파일로 되어 있으면 거기에 있는 정보들을 읽어와서 그것을 보여주는 것입니다. 

linux는 이런 구조로 되어 있으면 또 다른 os에서는 이런 역할을 하는 디렉토리 혹은 파일들이 있을 것이라고 생각합니다.

# ref

- [gopsutil](https://github.com/shirou/gopsutil)
- [제타위키 메모리 사용률](https://zetawiki.com/wiki/%EB%A6%AC%EB%88%85%EC%8A%A4_%EB%A9%94%EB%AA%A8%EB%A6%AC_%EC%82%AC%EC%9A%A9%EB%A5%A0_%ED%99%95%EC%9D%B8)
- [IBM df](https://www.ibm.com/docs/ko/aix/7.2?topic=system-displaying-available-space-file-df-command)
- [linux directory 설명](https://wlgnschlkkk.tistory.com/62)
