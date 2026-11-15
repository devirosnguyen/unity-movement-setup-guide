---
title: Lỗi & cách sửa
nav_order: 6
---

# Lỗi đã gặp và cách sửa
{: .no_toc }

Đây là các lỗi gặp thực tế khi dựng project test Movement SDK 209.0.0 trên Unity 6000.3.9f1.

<details open markdown="block">
  <summary>Mục lục</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## 1. Lỗi compile: `NUnit could not be found`
{: #issue-1 }

**Triệu chứng**

```
Packages\com.meta.xr.sdk.movement\Tests\Editor\MSDKUtilityEditorMetadataPathTests.cs(5,7):
error CS0246: The type or namespace name 'NUnit' could not be found
```

**Nguyên nhân:** assembly `Meta.XR.Movement.Editor.Tests` tham chiếu `nunit.framework.dll` và `UnityEngine.TestRunner`. Hai thư viện này đi kèm package **Unity Test Framework**, mà project mới tạo không có sẵn package đó.

**Cách sửa:** thêm vào `Packages/manifest.json`:

```json
"com.unity.test-framework": "1.6.0"
```

---

## 2. Không tải được bộ cài Meta XR Simulator
{: #issue-2 }

**Triệu chứng**

```
[Meta XR Simulator Installer] Failed to download installer:
Failed to create file C:\Users\<user>\Downloads\meta_xr_simulator.msi
Curl error 56: Recv failure: Connection was reset
```

**Nguyên nhân:** Unity không ghi được file vào thư mục Downloads, đồng thời kết nối tải bị ngắt.

**Cách sửa:** tải bộ cài Meta XR Simulator bằng tay từ Meta Horizon developer center rồi cài. Lỗi này không ảnh hưởng nếu bạn chỉ test trên Quest thật.

---

## 3. Không kết nối được Meta XR Simulator khi Play
{: #issue-3 }

**Triệu chứng:** Simulator đã cài và đã Activate, nhưng bấm Play thì cửa sổ Simulator không hiện và không có log OpenXR nào.

**Nguyên nhân:** `Assets/XR/XRGeneralSettingsPerBuildTarget.asset` chỉ có cấu hình **Android**. Play mode trong Editor dùng cấu hình **Standalone (Windows)**, mà ở đó không có loader nào được bật.

**Cách sửa:**
1. **Project Settings → XR Plug-in Management → tab Windows**: tick **OpenXR**, bật **Initialize XR on Startup**.
2. **OpenXR → tab Windows**: tick **Meta XR Feature Group** và thêm **Oculus Touch Controller Profile**.
3. Chạy **Project Setup Tool → Fix All** ở tab Windows.

---

## 4. `OpenXR package has been updated, Unity must be restarted`
{: #issue-4 }

**Triệu chứng:** Console báo lỗi này. Play vẫn không kết nối được Simulator.

**Nguyên nhân:** Project Setup Tool đã nâng `com.unity.xr.openxr` (1.15.1 lên 1.16.1). Chưa restart thì OpenXR loader không khởi tạo được.

**Cách sửa:** đóng hẳn Unity rồi mở lại project.

---

## 5. Simulator bị Deactivate
{: #issue-5 }

**Triệu chứng:** Log ghi `[Meta XR Simulator is deactivated] XR_SELECTED_RUNTIME_JSON is set to` với giá trị rỗng.

**Cách sửa:** vào **Meta → Meta XR Simulator → Activate**, rồi kiểm tra bằng **Status** (`Active: Yes`).

---

## 6. `[OVRBody] Failed to start body tracking with joint set FullBody`
{: #issue-6 }

**Nguyên nhân:** XR chưa khởi tạo (do các lỗi #3, #4, #5), hoặc runtime không hỗ trợ joint set Full Body.

**Cách sửa:** sửa các lỗi #3 đến #5 trước. Nếu vẫn còn, đổi **Body Tracking Joint Set = Upper Body** trong OVRManager, hoặc test trên Quest thật.

---

## 7. `Assertion failure. Value was Null` ở `RayInteractor.Start()`
{: #issue-7 }

**Nguyên nhân:** Interaction SDK không tìm thấy controller hoặc tay đang được track. Thường xảy ra khi XR chưa kết nối, hoặc khi input của Simulator đang để **None**.

**Cách sửa:** để Simulator kết nối được và đặt input = **Controller** hoặc **Hand**.

---

## 8. `The referenced script on this Behaviour is missing!`
{: #issue-8 }

**Nguyên nhân:** scene sample tham chiếu một script (GUID `a79441f348de89743a2939f4d699eac1`) không có trong project. Có thể là script camera của URP, vì sample được làm trên URP còn project đang dùng Built-in RP.

**Ảnh hưởng:** chỉ sinh cảnh báo, không làm hỏng tracking hay locomotion. Chuyển project sang URP thì cảnh báo sẽ hết.

---

## 9. Locomotion character không di chuyển
{: #issue-9 }

**Triệu chứng:** trong `MovementISDKLocomotion`, đi bằng WASD trong Simulator nhưng nhân vật đứng yên.

**Nguyên nhân:** nhân vật chỉ chạy animation khi Interaction SDK phát **locomotion event** (thumbstick slide, teleport, turn). WASD chỉ di chuyển headset, không phát event nào.

**Cách sửa:** đặt Simulator input = **Controller** rồi dùng **thumbstick** (phím gán trong **Input bindings**, hoặc tay cầm Xbox). Xem [Demo §3](demo#step-3).

---

## 10. `SceneSelectMenu` không chuyển scene
{: #issue-10 }

**Cách sửa:** thêm các scene sample vào **Build Profiles → Scene List**.

---

## 11. Thiếu layer `HiddenMesh`
{: #issue-11 }

**Triệu chứng:** `RecalculateNormals` hoạt động sai mà không báo lỗi, hoặc có assert `LayerMask.NameToLayer("HiddenMesh") > -1`.

**Cách sửa:** tạo layer 10, 11 và `HiddenMesh` (xem [Setup §5](setup#step-5)). Có thể dùng **Project Setup Tool → Fix** để tự gán.
