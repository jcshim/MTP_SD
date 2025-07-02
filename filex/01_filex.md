# MTP_SD: MTP 필터링 SD 저장장치
목표는 STM32H7B0VBT6 보드에서 다음 기능을 구현

---

## 🎯 **목표 요약**

> **"STM32H7B0VBT6 보드에서 SDMMC로 SD카드를 연결하고, USB로 PC에 MTP 장치처럼 보이게 하되, 특정 확장자(.hwp, .txt, .doc)만 보여지도록 필터링"**

---

## ✅ 구현 구성요소

| 구성 요소                | 설명                            |
| -------------------- | ----------------------------- |
| **SDMMC + FATFS**    | SD카드 파일 시스템 연결                |
| **USBX (Device)**    | USB 통신을 위한 ST USBX 미들웨어       |
| **FileX**            | USBX가 사용하는 파일 시스템 (FAT과 유사)   |
| **MTP Class (PIMA)** | PC에서 사진/문서 파일처럼 탐색 가능         |
| **파일 필터링**           | `.hwp`, `.txt`, `.doc` 파일만 노출 |

---

## ✅ 단계별 학습 로드맵 (쉽고 간단하게)

### ① **CubeMX 기초**

* ✅ STM32CubeMX에서 `SDMMC1` + `USBX Device` + `FileX + MTP` 설정하는 법
* 💡 CubeMX는 각 기능의 드라이버와 코드 구조를 자동 생성해 줌

### ② **SDMMC + FileX 실습**

* ✅ SD 카드에 파일 읽기/쓰기 실습 (FileX API 익히기)
* 💡 `fx_file_open`, `fx_file_read`, `fx_file_close` 중심

### ③ **USBX + MTP 구조 이해**

* ✅ `ux_device_class_pima_xxx.c` 구조 익히기
* 💡 `GetObjectHandles`, `GetObjectInfo`, `GetObject` 함수가 핵심

### ④ **파일 필터링 구현**

* ✅ `GetObjectHandles` 내에서 `.hwp`, `.doc`, `.txt` 확장자만 Object에 포함
* 💡 FileX로 파일 목록 순회하며 조건 필터링 추가

### ⑤ **PC와 테스트**

* ✅ Windows 탐색기에서 연결 확인
* ✅ 특정 파일만 보이는지 확인

---

## ✅ 반드시 공부할 개념 (쉬운 설명)

| 개념           | 왜 필요한가요?                | 핵심만 쉽게                                 |
| ------------ | ----------------------- | -------------------------------------- |
| **SDMMC**    | 빠른 속도로 SD카드 읽기/쓰기       | SD카드를 병렬로 연결하는 인터페이스                   |
| **FileX**    | USBX에서 사용하는 FAT 비슷한 시스템 | `fx_`로 시작하는 함수로 파일 처리                  |
| **USBX**     | USB 프로토콜 처리             | PC가 STM32를 USB 장치로 인식하게 함              |
| **MTP**      | PC가 스마트폰처럼 STM32를 인식함   | 사진/문서처럼 파일 탐색 가능                       |
| **PIMA 명령어** | MTP에서 어떤 파일인지 알려주는 방식   | `GetObjectHandles()`가 중요               |
| **파일 필터링**   | 원하는 파일만 보여주기            | 확장자 `.hwp`, `.txt`, `.doc`만 Object로 등록 |

---

## ✅ 활용할 예제 및 소스

| 자료                   | 설명                                | 위치                                                       |
| -------------------- | --------------------------------- | -------------------------------------------------------- |
| **USBX MTP 예제**      | 완성된 구조 제공                         | STM32Cube\_FW\_H7/Examples/USBX/Ux\_Device\_PIMA\_MTP    |
| **FileX 예제**         | 파일 읽기/쓰기 기본 구조                    | STM32Cube\_FW\_H7/Applications/FileX/FX\_uSD\_File\_Edit |
| **CubeIDE 자동 생성 코드** | 설정한 USBX + FileX + SDMMC 구조 자동 생성 | CubeMX `.ioc` 설정 기반                                      |

---

## ✅ 개발 꿀팁 요약

* CubeMX로 **USBX + FileX + SDMMC1 + MTP 클래스** 설정만 잘하면 기본 구조는 자동 생성됨
* **`ux_device_class_pima_get_object_handles()`** 안에서 `fx_directory_first_entry_find()`를 이용해 `.hwp`, `.txt`, `.doc`만 Object로 등록하면 됨
* FileX는 `fx_media_open()`, `fx_file_open()`, `fx_file_read()` 위주로 사용
* MTP는 `PC ↔ 장치 간 파일 교환 규약`이므로 명령 흐름 (Command → Data → Response)을 이해하면 됨

---

## ✅ 예시 흐름: 필터링 핵심 코드 (GetObjectHandles 내부)

```c
FX_DIRECTORY_ENTRY entry;
CHAR* ext;

if (fx_directory_first_entry_find(&media, &entry) == FX_SUCCESS) {
  do {
    ext = strrchr((char*)entry.fx_file_name, '.');
    if (ext && (strcasecmp(ext, ".hwp") == 0 || strcasecmp(ext, ".txt") == 0 || strcasecmp(ext, ".doc") == 0)) {
      // 해당 파일을 Object Handle로 등록
    }
  } while (fx_directory_next_entry_find(&media, &entry) == FX_SUCCESS);
}
```

---

## ✅ 마무리 요약

| 단계 | 해야 할 일                                |
| -- | ------------------------------------- |
| 1  | CubeMX로 `SDMMC + FileX + USBX MTP` 설정 |
| 2  | FileX로 SD 카드 접근 테스트                   |
| 3  | USBX MTP 예제 분석 및 MTP 동작 흐름 익히기        |
| 4  | `GetObjectHandles()` 수정하여 확장자 필터링 구현  |
| 5  | Windows PC에서 MTP 연결 테스트               |

---

필요하시면:

* CubeMX `.ioc` 파일 예제
* 수정된 `ux_device_class_pima_get_object_handles.c` 예시
* FileX + USBX + SDMMC 통합 샘플 코드
