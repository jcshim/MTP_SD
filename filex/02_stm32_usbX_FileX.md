가장 유사한 구현 예제를 찾았습니다! 아래 GitHub 저장소가 당신의 요구사항과 가장 잘 맞습니다:

---

## ✅ 추천 저장소: **STMicroelectronics/stm32-usbx-examples**

* 이 저장소는 **STM32 H7 시리즈를 포함한 USBX + ThreadX + FileX 기반 예제 모음**을 제공합니다 ([github.com][1]).
* 특히 **`USBX Ux_Device_PIMA_MTP`** 예제가 **Media Transfer Protocol (MTP)** 장치로 작동하는 완전한 구현 예시를 포함합니다 ([github.com][1]).
* STM32와 USBX로 MTP 장치를 만들고, FileX 기반으로 파일 입출력을 처리하는 구조를 그대로 볼 수 있어 매우 유용합니다.

---

### 🔍 예제 특징

| 항목                               | 설명                                                               |
| -------------------------------- | ---------------------------------------------------------------- |
| #1 **MTP 기능**                    | `Ux_Device_PIMA_MTP` 예제는 STM32 보드를 Windows MTP 장치로 인식하도록 설정되어 있음 |
| #2 **USBX + ThreadX + FileX 통합** | RTOS와 파일 시스템, 그리고 USBX가 어떻게 연동되는지 명확하게 구조화됨                      |
| #3 **실습 가능한 코드 베이스**             | 폴더 및 파일 탐색, 전송, 저장 전체 흐름을 예제로 제공                                 |
| #4 예제 설명 포함                      | README에 기본 사용법, 빌드 및 디버깅 방법이 정리됨                                 |

---

### 📌 다음 단계 제안

1. **GitHub 저장소 복제**
   `stm32-usbx-examples` 를 자신의 개발환경에 클론하세요.

2. **`Ux_Device_PIMA_MTP` 프로젝트 열기**
   STM32CubeIDE나 CubeMX를 사용하여 예제 프로젝트를 불러옵니다.

3. **필요한 기능 추가/수정**

   * **FATFS + SDMMC** 코드 통합
   * **파일 필터링 로직** (예: `.txt`, `.doc`, `.hwp`) 삽입
   * 다양한 파일 전송 시나리오 테스트

4. **PC와 테스트**
   윈도우에서 MTP 장치를 확인하고, 파일/폴더 전송 및 저장 여부를 확인하세요.

---

이 예제는 **최소한의 FileX + USBX + MTP 송/수신 구조**를 이미 제공하고 있어서,
당신이 구현하려는 **sdmmc + fatfs + MTP + 필터링**의 기반을 빠르게 갖출 수 있습니다.

이 리포지토리의 **어떤 파일이나 구조를 먼저 살펴보고 싶으신가요?**
필요하시면 직접 분석 도와드릴게요. 😊

[1]: https://github.com/STMicroelectronics/stm32-usbx-examples?utm_source=chatgpt.com "STMicroelectronics/stm32-usbx-examples: Provide a set of USB ..."
