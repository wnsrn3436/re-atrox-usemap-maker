# 아트록스 유즈맵메이커

아트록스에는 맵 에디터가 딸려 있는데 정작 직접 만든 맵을 게임에서 불러올 방법이 없다. 그래서 게임의 리소스 아카이브인 pak 파일을 바이너리로 뜯어 캠페인 미션 맵이 저장된 위치를 전부 찾아내고, 그 자리를 사용자 맵으로 덮어써서 미션 선택 화면에서 유즈맵을 플레이할 수 있게 만든 도구다.

**호환 버전**: 아트록스 v1.10 build 1


## 사용 방법

Releases에서 받아 압축을 풀고 `UseMap` 폴더에 맵 파일을 넣은 뒤 실행하면 된다. 창은 뜨지 않고 작업이 끝나면 저절로 종료된다.

파일 이름이 곧 어느 미션 자리에 넣을지를 정한다.

| 이름 | 진영 |
|---|---|
| `h_1` ~ `h_9` | 호미니언 |
| `c_1` ~ `c_9` | 크리티스 |
| `i_1` ~ `i_8` | 인텔리언 |

`atrox.pak` 은 기본 설치 경로에서 찾는다. 거기 없으면 파일 선택창이 뜨니 아트록스가 설치된 폴더의 `dev\atrox.pak` 을 고른다.

그다음 아트록스를 실행해 미션 선택 화면에서 해당 미션을 고르면 넣은 맵이 나온다.

넣은 맵이 끝난 뒤 다음 미션으로 이어지게 하려면 맵 에디터의 트리거(실행 > 다음 스테이지 사용 맵 설정)에 원래 미션 파일의 경로를 적는다.

| 진영 | 경로 |
|---|---|
| 호미니언 | `Scenario\hominian01.spm`, `hominian02a.spm`, `hominian02b.spm`, `hominian03.spm` 부터 `hominian08.spm` |
| 크리티스 | `Scenario\createse01.spm` 부터 `createse09.spm` |
| 인텔리언 | `Scenario\intelion01.spm` 부터 `intelion08.spm` |

pak 파일을 직접 고쳐 쓰므로 실행 전에 `atrox.pak` 을 백업해두는 게 좋다.


## 구현 원리

**미션 맵이 pak 안에 놓인 위치를 찾아 표로 만들었다.** pak을 바이너리로 분석해 세 진영의 캠페인 미션 26개가 각각 어느 바이트 위치에 저장되는지 알아냈고, 그 값을 진영별 스크립트에 그대로 박아뒀다. 미션 번호를 넣으면 오프셋을 돌려주는 표다.

```gml
// sk_h.gml - 호미니언
switch(argument0)
{
case 1: return 85598720
case 2: return 86157824
case 3: return 86743552
...
}
```

**덮어쓰기는 그 위치로 건너뛰어 바이트를 밀어 넣는 것이다.** pak을 쓰기 모드로 열고 해당 오프셋으로 이동한 다음, 사용자 맵 파일을 한 바이트씩 읽어 그대로 써 넣는다. 파일 구조를 다시 쓰거나 인덱스를 갱신하지 않고 자리만 바꿔치기한다.

```gml
for(i=1; file_exists("usemap\h_"+string(i)+".spm"); i+=1)
{
  open1 = file_bin_open(folder, 2)
  open2 = file_bin_open("usemap\h_"+string(i)+".spm", 0)

  file_bin_seek(open1, sk_h(i))
  for(j=0; j!=file_bin_size(open2); j+=1)
  {
    file_bin_write_byte(open1, file_bin_read_byte(open2))
  }

  file_bin_close(open1)
  file_bin_close(open2)
}
```

**pak 위치는 기본 설치 경로를 먼저 본다.** 거기에 없으면 파일 선택창을 띄워 직접 고르게 한다. 화면도 버튼도 없이 실행하면 끝나는 구조라 오브젝트 하나의 Create 이벤트에 전부 들어가 있다.


## 파일

| 경로 | 내용 |
|---|---|
| `source/atrox-usemap-maker.gmk` | 원본 프로젝트 파일 |
| `source/split/` | GmkSplitter로 분해한 텍스트 트리 |
| Releases | 실행 파일과 사용 설명, 예제 맵 1개 |


## 라이선스

zlib 라이선스다. 자세한 내용은 [LICENSE](LICENSE) 에 있다.
