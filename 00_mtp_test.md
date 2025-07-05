좋습니다. STM32H7B0VBT6 (Aliman 보드)에서 STM32CubeIDE 1.18.1을 사용하여 **직접 USBD 기반의 MTP 장치를 구현하고**, **microSD(FATFS)** 연동까지 포함할 때, **PC에서 장치가 인식되지 않는 문제**를 해결하기 위해 **최소 코드 기반 단계별 디버깅 및 확인** 절차를 아래와 같이 안내드리겠습니다.

---

## ✅ 1단계: **CubeMX 설정 (STM32CubeIDE 1.18.1 내장)**

### 🧩 필수 설정

| 설정 항목                       | 값                               |
| --------------------------- | ------------------------------- |
| **USB\_OTG\_HS**            | Device Only (FS PHY 사용)         |
| **Middleware > USB Device** | 활성화                             |
| **Class for USB Device**    | Custom HID (임시, 이후 직접 구현 대체 예정) |
| **SDMMC1**                  | SD 4-bit Wide bus mode          |
| **FATFS**                   | User-defined, SDMMC1 연동         |
| **RCC > HSE**               | Crystal/Ceramic Resonator       |
| **SYS > Timebase**          | SysTick                         |

> ⚠️ Class는 MTP가 없으므로 일단 “Custom HID”를 선택 후 코드를 생성하고, 이후 `usbd_customhid_if.c/h`, `usbd_customhid.c/h`를 MTP로 바꿔치기합니다.

---

## ✅ 2단계: **코드 생성 후 최소 변경 확인**

### `usb_device.c`에 다음이 있는지 반드시 확인

```c
USBD_HandleTypeDef hUsbDeviceFS;

void MX_USB_DEVICE_Init(void)
{
  // 우리가 직접 만든 MTP 클래스 연결
  USBD_Init(&hUsbDeviceFS, &FS_Desc, DEVICE_FS);
  USBD_RegisterClass(&hUsbDeviceFS, &USBD_MTP);
  USBD_MTP_RegisterInterface(&hUsbDeviceFS, &USBD_Interface_fops_FS);
  USBD_Start(&hUsbDeviceFS);
}
```

---

## ✅ 3단계: **Custom MTP Class 파일 구성**

* 다음 4개의 파일을 직접 생성하거나 기존 Custom HID 파일을 복사해서 이름 변경:

  * `usbd_mtp.c`
  * `usbd_mtp.h`
  * `usbd_mtp_if.c`
  * `usbd_mtp_if.h`

### `usbd_mtp.h` 기본 구조

```c
#ifndef __USB_MTP_H__
#define __USB_MTP_H__

#include "usbd_ioreq.h"

extern USBD_ClassTypeDef USBD_MTP;
#define USBD_MTP_CLASS &USBD_MTP

#endif
```

### `usbd_mtp.c` 최소 골격 예제

```c
#include "usbd_mtp.h"
#include "usbd_ctlreq.h"

static uint8_t USBD_MTP_Init(USBD_HandleTypeDef *pdev, uint8_t cfgidx) {
  return USBD_OK;
}
static uint8_t USBD_MTP_DeInit(USBD_HandleTypeDef *pdev, uint8_t cfgidx) {
  return USBD_OK;
}
static uint8_t USBD_MTP_Setup(USBD_HandleTypeDef *pdev, USBD_SetupReqTypedef *req) {
  return USBD_OK;
}
static uint8_t USBD_MTP_DataIn(USBD_HandleTypeDef *pdev, uint8_t epnum) {
  return USBD_OK;
}
static uint8_t USBD_MTP_DataOut(USBD_HandleTypeDef *pdev, uint8_t epnum) {
  return USBD_OK;
}

USBD_ClassTypeDef USBD_MTP = {
  USBD_MTP_Init,
  USBD_MTP_DeInit,
  USBD_MTP_Setup,
  NULL, NULL,
  USBD_MTP_DataIn,
  USBD_MTP_DataOut,
  NULL, NULL,
  NULL
};
```

---

## ✅ 4단계: **Device Descriptor 수정**

`usbd_desc.c`에서 MTP 장치로 보이도록 **Product string**과 **VID/PID**를 수정합니다.

```c
#define USBD_VID     0x0483  // STMicroelectronics
#define USBD_PID     0x5740  // MTP 호환 PID 권장
#define USBD_PRODUCT_STRING     "STM32 MTP Device"
```

---

## ✅ 5단계: **컴파일 후 연결 테스트**

1. 컴파일 및 다운로드
2. 보드 연결
3. PC에서 장치 관리자로 이동

> ❗장치 관리자에서 `"STM32 MTP Device"` 또는 `"Unknown MTP Device"`라도 표시되면 **USB Enumeration은 성공**입니다.
> 이 단계에서 아무것도 뜨지 않으면:

* USB 핀 연결 문제 (PA11, PA12)
* 클럭 설정 문제
* USB 전원 미공급
* `USBD_Start()` 호출 누락
* Descriptor 구조 오류

---

## ✅ 6단계: **문제 발생 시 확인 포인트**

| 항목                 | 점검 내용                                             |
| ------------------ | ------------------------------------------------- |
| USB 클럭             | 48MHz 동기화 되어야 함                                   |
| PA11/PA12          | USB\_DM/DP로 설정되어야 함                               |
| VBUS Sense         | Disable 권장 (`MX_USB_DEVICE_Init()` 전후 확인)         |
| USB\_IRQHandler 등록 | `OTG_HS_IRQHandler()` → `HAL_PCD_IRQHandler()` 확인 |
| USB Pull-Up        | 내부 pull-up 확인 (`HAL_PCDEx_SetConnectionState`)    |

---

## ✅ 7단계: **다음 단계로 MTP 프로토콜 구현**

* Container 구조, Event, Object Handle, File transfer 기능 추가
* SD 카드 탐색 → 파일 열기 → 전송 → MTP 이벤트 응답 구조 구성

---

## ➕ 요청 시 제공 가능한 추가 자료

* `usbd_mtp.c/.h` 최소 작동 버전
* Windows 측 드라이버 확인법
* MTP 구조 전체 흐름도
* FATFS 연동 예제

---

필요한 단계에서 멈춘 부분이나, `usb_device.c`, `usbd_mtp.c` 샘플 코드 요청하시면 이어서 도와드릴게요.
