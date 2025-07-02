질문 요지가 다음과 같다고 이해했습니다:

> “**윈도우에서 STM32 MTP 장치(SD 카드)에 폴더를 통째로 복사**하면, STM32가 그 폴더 내의 `.hwp`, `.doc`, `.txt` 파일만 받아서 **microSD에 저장**하고, 나머지 파일(예: `.jpg`, `.exe`, `.mp3`)은 **무시하거나 저장하지 않도록** 구현할 수 있는가?”

👉 **답변: 예, 가능합니다.**
다만 이를 구현하기 위해서는 **MTP의 `SendObject`, `SendObjectInfo` 요청을 해석하여, 전송 대상 파일의 확장자를 판단한 후 저장 여부를 결정**해야 합니다.

---

## ✅ 동작 방식 요약 (필터링 구현 포인트)

### 윈도우 → STM32로 폴더 복사 시, MTP 내부에서는 다음 흐름이 발생합니다:

1. `SendObjectInfo`: 전송할 파일의 **이름, 크기, 포맷코드** 등의 정보 전송
2. `SendObject`: 해당 파일의 **실제 데이터** 전송 시작
3. 장치는 두 요청을 받아 **파일을 저장할지 말지 판단**

---

## ✅ 필터링 구현 포인트

### ▶ `ux_device_class_pima_send_object_info()`

* 여기서 `object_format_code`, `object_filename`을 보고 확장자를 확인
* `.txt`, `.hwp`, `.doc`가 아닌 경우 **`UX_DEVICE_CLASS_PIMA_RC_Invalid_Object_Format_Code`** 등의 **오류 응답을 리턴**

```c
if (!has_valid_extension(object_name)) {
    return UX_DEVICE_CLASS_PIMA_RC_Invalid_Object_Format_Code;  // 전송 거부
}
```

```c
bool has_valid_extension(const char* filename) {
    const char* ext = strrchr(filename, '.');
    if (!ext) return false;

    return strcasecmp(ext, ".hwp") == 0 ||
           strcasecmp(ext, ".doc") == 0 ||
           strcasecmp(ext, ".txt") == 0;
}
```

### ▶ `ux_device_class_pima_send_object()`

* 전송이 수락된 경우에만 FileX 또는 FATFS로 실제 저장

---

## ✅ 폴더 구조 복사 시 처리

* 윈도우에서 폴더 전체를 복사하면 내부적으로는:

  * **폴더 생성 요청 (`SendObjectInfo` + `ObjectFormat = association`)**
  * **파일 단위로 반복 전송**
* 폴더는 그냥 생성되지만, 그 안의 파일은 확장자가 허용된 것만 저장

📁 예:
윈도우에서 `Documents/` 폴더를 MTP 장치에 복사할 때:

| 파일           | 처리 결과   |
| ------------ | ------- |
| `report.hwp` | ✅ 저장됨   |
| `image.jpg`  | ❌ 저장 거부 |
| `readme.txt` | ✅ 저장됨   |
| `video.mp4`  | ❌ 저장 거부 |

폴더 구조는 유지되지만, 내부 파일은 필터링됩니다.

---

## ✅ 정리

| 질문                             | 답변                                           |
| ------------------------------ | -------------------------------------------- |
| 윈도우에서 폴더 복사 → 내부 문서 파일만 저장 가능? | ✅ **가능**                                     |
| 어디서 구현해야 하나요?                  | `send_object_info()`에서 파일명 기반 판단             |
| 폴더는 어떻게 처리되나요?                 | 자동 생성됨 (파일만 필터링 대상)                          |
| 거부된 파일은 윈도우에 어떻게 보이나요?         | 복사 실패 메시지 없이 사라지거나 경고창 발생 가능 (MTP 동작에 따라 다름) |

---

## 💡 추가로 구현하면 좋은 기능 (선택사항)

* 파일 확장자 외에도 **파일 크기 제한**, **숨김 파일 제외** 등의 조건 추가 가능
* 거부된 파일은 **임시 로그에 기록하거나 무시**
* 루트 경로 외에도 **서브 폴더 경로 유지 저장**도 가능

---

필요하시면:

* 위 함수들에 대한 실 코드 템플릿 (`send_object_info`, `send_object`)
* CubeIDE 프로젝트 구조 예시
* 필터링된 파일 저장 경로 관리 방식
