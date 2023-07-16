Redis의 AOF(Append Only File)는 데이터 변경이 있을 때마다 모든 작업을 로그에 기록하므로, 시간이 지남에 따라 AOF 파일 크기가 커질 수 있습니다. 따라서, AOF 파일의 크기를 관리하는 데에는 AOF 재작성이라는 메커니즘이 사용됩니다.

AOF 재작성은 현재의 데이터 상태를 반영하는 새로운 AOF 파일을 생성하는 작업으로, 불필요한 기록을 제거하여 파일 크기를 줄이는 역할을 합니다. 이는 BGREWRITEAOF 명령어를 사용하여 수동으로 실행할 수 있습니다.

그러나, 수동으로 AOF 재작성을 진행하는 것은 별로 효율적이지 않기 때문에, 보통은 자동 AOF 재작성을 설정합니다. Redis 설정 파일(redis.conf)에서 다음의 두 가지 옵션을 설정할 수 있습니다:

auto-aof-rewrite-percentage: 이 옵션은 기존 AOF 파일 크기가 AOF 재작성 후 예상되는 크기보다 몇 퍼센트 더 클 때 AOF 재작성을 실행할 것인지를 설정합니다. 기본값은 100입니다. 예를 들어, 이 값을 100으로 설정하면, 현재의 AOF 파일이 재작성 후 예상되는 크기의 2배가 되면 재작성이 발생합니다.

auto-aof-rewrite-min-size: 이 옵션은 자동 AOF 재작성을 위한 최소 파일 크기를 설정합니다. 이 값보다 작은 크기의 AOF 파일에 대해서는 재작성이 발생하지 않습니다. 이는 매우 작은 AOF 파일에 대해 너무 자주 재작성이 발생하는 것을 방지하기 위한 것입니다. 기본값은 64MB입니다.

위의 두 설정을 조합하여 AOF 파일의 크기를 제어할 수 있습니다. 예를 들어, auto-aof-rewrite-percentage를 100, auto-aof-rewrite-min-size를 64MB로 설정하면, AOF 파일이 64MB 이상이고 현재 크기가 재작성 후 예상 크기의 두 배가 되는 경우에 AOF 재작성이 발생합니다.

![image](https://github.com/eastperson/TIL/assets/66561524/f03e55e1-0f47-4554-bc58-95fa58fa516c)

AOF 사용의 장점

- 모든 변경사항이 기록되므로 RDB 방식 대비 안정적으로 데이터 백업 가능
- AOF 파일은 append-only 방식이므로 백업 파일이 손상될 위험이 적음
- 실제 수행된 명령어가 저장되어 있으므로 사람이 보고 이해할 수 있고 수정도 가능

![image](https://github.com/eastperson/TIL/assets/66561524/416041a8-67de-4f15-b327-75c9bcc99303)

AOF 사용의 단점

- RDB 방식보다 파일 사이즈가 커짐
- RDB 방식 대비 백업&복구 속도가 느림(백업 성능은 fsync 정책에 따라 조절 가능)

```sql
// AOF 사용(기본값은 no)
appendonly yes

// AOF 파일 이름
appendfilename appendonly.aof

// fsync 정책설정(always, everysec, no)
appendfsync everysec
```

fsync 정책(appendfsyync 설정 값)

- fsync() 호출은 OS에게 데이터를 디스크에 쓰도록 함
- 가능한 옵션과 설명
    - always: 새로운 커맨드가 추가될 때마다 수행. 가장 안전하지만 가장 느림
    - everysec: 1초마다 수행. 성능은 RDB 수준에 근접
    - no: OS에 맡김. 가장 빠르지만 덜 안전한 방법(커널마다 수행 시간이 다를 수 있음)

AOF 관련 개념

- Log rewiring: 최종 상태를 만들기 위한 최소한의 로그만 남기기 위해 일부를 새로 씀 (ex. 1개의 key 값을 100번 수정해도 최종 상태는 1개이므로 SET 1개로 대체 가능)
- Multi Part AOF: Redis 7.0부터 AOF가 단일 파일에 저장되지 않고 여러 개가 사용됨
    - base file: 마지막 rewrite 시의 스냅샷을 저장
    - incremental file: 마지막으로 base file이 생성된 이후의 변경사항이 쌓이
    - manifest file: 파일들을 관리하기 위한 메타 데이터를 저장
