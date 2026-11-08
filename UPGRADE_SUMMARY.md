# DYNAMIC ISLAND UPGRADE & MODIFICATION REPORT

This document summarizes all technical upgrades, bug fixes, performance optimizations, and code modifications integrated into the Dynamic Island for Windows project.

---

## 1. OVERVIEW OF UPGRADES

### 1.1. Interactive Companion Eyes System
- Integrated animated luminous robot eyes into the Dynamic Island across both Collapsed Idle and Media Bar modes.
- Real-time full-screen cursor gaze tracking with natural physics smoothing.
- Typing Peek: Automatically tilts gaze downwards toward the keyboard when continuous high-speed typing is detected.
- Dizzy Roll: Detects rapid back-and-forth zigzag mouse wiggling (5 directional reversals within 0.4 seconds) to trigger a 2-second spinning pupil state. Linear high-speed cursor movements on high-DPI displays no longer trigger false positives.
- Head-bobbing / Groove: Rhythmic alternating bouncing (~110 BPM) synchronized to music playback.
- Natural Blinking and Happy Charging crescent animation.

### 1.2. Angry Eyebrow Geometry Fix
- Issue: The left eyebrow originally calculated its starting point from the island contour, resulting in a distorted horizontal bar stretching across to the notch wing.
- Solution: Fixed to independent, compact vectors directly above each eye (`eyeCenterX - 9px` to `+7.5px`), with a thickness of `2.6px`, rounded endcaps via `FillEllipse`, and sharp inward downward slant (`\ /`).

### 1.3. Deep Sleep Mode & Floating "Zzz" Effect
- Trigger condition: Automatically enters deep sleep when idle (no mouse or keyboard input) for 10 seconds.
- Drool removal: Drool line, droplet bulb, and specular reflection have been completely removed per design change.
- Replacement effect: Floating vector "Zzz" animation. Three staggered `Z` glyphs rise from the right corner of the sleeping face, scaling smoothly from small `z` to larger `Z` along a gentle sine wave path before fading out.

### 1.4. Music Playback Sleep Exemption
- While media is active and playing (`state.media.available && state.media.playing`), deep sleep and night sleepiness are strictly prohibited.
- If music playback starts while the island is asleep, it immediately wakes up smoothly without startled or angry reactions, entering the groove state directly.

### 1.5. High Refresh Rate Optimization (60Hz to 300Hz+)
- Removed the artificial 60 FPS cap (`kContinuousFrameMs = 1000.0 / 60.0`).
- Hardware VSync synchronization via `DwmFlush()`, supporting native monitor refresh rates (120Hz, 144Hz, 165Hz, 240Hz, 300Hz, 360Hz+).
- Frame-rate independent exponential smoothing for gaze tracking: `smoothFactor = 1.0 - exp(-18.0 * dt)`.
- Sub-millisecond pacing with `timeBeginPeriod(1)` and yield-spinning under 10ms frame times, delivering stutter-free rendering with minimal CPU overhead.

### 1.6. Typography & Responsive DPI Scale Synchronization
- **Icon Button Scaling**: Fixed Play/Pause, Previous/Next, and generic icons (`iconFormat_`, `mediaPlayIconFormat_`, `mediaNavIconFormat_`) to properly scale with `textScale` (`16.0f * ts`, `18.0f * ts`, `14.0f * ts`).
- **Weather Description Scaling**: Fixed weather status text to scale with both island geometry `sizeScale` and typography `textScale`.
- **Immersive Lyrics Scaling**: Synchronized lyrics text size and cache invalidation guard with both `sizeScale` and `textScale` for high-DPI (4K) multi-monitor setups.

### 1.7. Group 1: Dynamic Ambient Visual & Particle System (UI / Visual Nâng Cao)
- **Dynamic Ambient Color & Aura Glow**:
  - **Media Playback**: Ambient aura automatically adopts the sampled album art accent color (`currentAccent_`). Audio-reactive rhythm modulates aura intensity and stroke based on real-time frequency bands (`state.bands`) and ~110 BPM groove beat.
  - **Music Beat Pulse Ripple**: Subtle expanding halo waves propagate outwards from the island contour during energetic bass beats.
  - **Charging State**: Glowing Emerald/Cyan energy aura (`#00F5D4` / `#00E676`) with vibrant breathing cycle.
  - **Charging Energy Sparkles**: 3 luminous nano particles glide continuously along the island contour with smooth sine-envelope opacity.
  - **Low Battery**: Neon warning pulse in vivid amber-red (`#FF3B30`).
  - **Weather Context**: Warm golden amber aura for clear sunny days (`#FFB300`), deep twilight blue for rain/storms, and glacial ice cyan for cold/snow.
- **Smooth Module Cross-Fade Transition (Activity Morphing)**:
  - Added seamless 220ms quadratic ease-out transition between activity modes (Idle ↔ Media ↔ Volume ↔ Weather). Eliminates abrupt visual jumps during mode switches.

### 1.8. Hover Expand Spring Physics & Volume Metric Scaling
- **Fluid Critically Damped Expansion**:
  - Re-tuned expand spring stiffness from `380.0f` to `270.0f` and damping from `26.0f` to `33.0f` ($\zeta \approx 1.0$).
  - Eliminates the jarring numerical "pop" when mouse hovers the Dynamic Island. The island and its hero metrics (clock, temperature, calendar) expand organically with natural, liquid-smooth pacing (~0.24s).
- **Enhanced Volume Flyout Typography**:
  - Promoted volume level string (`XX%`) from faint 11px body text to semi-bold `13.0f * ts` with clean right-alignment, matching the visual prominence and hierarchy of the battery percentage metric.

### 1.9. Bold Text Overlap & Tab Transition Ghosting Elimination
- **Expanded Tab Transition Decoupling (`DrawTabTransition`)**:
  - Increased `kTravelPx` from `34.0f` to `50.0f` and sharpened fade envelope `kFadePower` from `1.4f` to `2.4f`.
  - Completely eliminates the visual collision where two large bold numbers (56px clock vs 63px calendar vs 42px weather) were painted over the same coordinate space simultaneously during tab switching.
- **Hero Typography Refinement (Bold to Semi-Bold)**:
  - Re-weighted `timeDashboardFormat_` (56px) and `calDayLargeFormat_` (63px) from `BOLD (700)` to `SEMI_BOLD (600)`.
  - Large numerals render with Apple-grade precision and elegance, preventing ink bleed, blurry bold halos, and overlapping stroke artifacts.
- **Zero-Overlap Collapsed vs Expanded Threshold**:
  - Separated the dissolve alpha thresholds in `DrawIdleDashboard`: collapsed view now cleanly reaches 0 alpha at 54px before expanded dashboard metrics begin at 56px.
- **Eliminated Nested Layering in `DrawPill`**:
  - Removed secondary layer wrappers on module switches, allowing each surface to draw text directly without D2D subpixel alpha doubling.

### 1.10. Resolution of Font Thickness Bloom (Grayscale Typography)
- **Direct2D Text Antialiasing Standardization**:
  - Configured `target_->SetTextAntialiasMode(D2D1_TEXT_ANTIALIAS_MODE_GRAYSCALE)` both on DC Render Target creation and per-frame after `BeginDraw()`.
  - **Eliminates Subpixel ClearType Popping**: Prevents Direct2D from dynamically toggling between ClearType and Grayscale rasterization modes when text renders across fading layers (`alpha < 0.995f` vs opaque `drawTab`).
  - **Apple-Grade Clarity**: Produces smooth, uniform text strokes across all transparency levels, completely eradicating colored subpixel fringes and mid-animation font stroke dilation ("blooming").

### 1.11. Elimination of Geometric Font Zooming & Pure Dissolve Transition
- **Removed Affine Matrix Scaling**:
  - Completely removed `D2D1::Matrix3x2F::Scale(scale, scale, center)` in `DrawTabTransition`.
  - Content and typography are locked at true 1:1 pixel scaling. Eliminates all text stretching, zooming, and font dilation when switching between tabs.
- **Pure Clean Dissolve Transition**:
  - Transitioned from overlapping vertical translations to smooth Hermite ease-in-out alpha blending (`std::pow(1.0f - eased, 2.8f)`).
  - Tabs transition with calm, Apple Watch / iOS-style cross-dissolve, preserving exact element coordinates without digits flying across or colliding into neighboring layouts.

---

## 2. MODIFIED CODE SECTIONS (PHAN MA NGUON DA CHINH SUA)

### 2.1. Frame-Rate Independent Cursor Gaze Tracking
Tieng Viet: Thuat toan noi suy ham mu doc lap toc do khung hinh, giup doi mat lieng theo chuot muot ma o tan so quet tu 60Hz den 300Hz+ ma khong bi tre hay giat cuc.

```cpp
// File: dynamic-island-for-windows.wh.cpp (DrawCompanionEyes)
// Frame-rate independent exponential smoothing for buttery-smooth tracking at 60Hz - 360Hz+:
float dt = static_cast<float>(mouseDt);
if (dt < 0.0005f || dt > 0.1f) dt = 0.016f;
const float smoothFactor = 1.0f - expf(-18.0f * dt);

g_companionEyes.lookX += (targetLookX - g_companionEyes.lookX) * smoothFactor;
g_companionEyes.lookY += (targetLookY - g_companionEyes.lookY) * smoothFactor;
```

### 2.2. Compact Symmetrical Angry Eyebrows
Tieng Viet: Chan may cau co tuc gian duoc co dinh toa do ngay tren tung ben mat, khong bi keo le thanh duong dai sang mep dao.

```cpp
// File: dynamic-island-for-windows.wh.cpp (DrawCompanionEyes)
if (isAngry) {
    D2D1_COLOR_F browCol = D2D1::ColorF(1.0f, 0.28f, 0.10f, 0.96f * settingsOpacity_);
    if (ID2D1SolidColorBrush* browBrush = AcquireBrush(browCol)) {
        const float browStroke = 2.6f * scale;
        D2D1_POINT_2F pStart, pEnd;
        if (isLeft) {
            // Left eyebrow slants down toward nose \
            pStart = D2D1::Point2F(eyeCenterX - 9.0f * scale, eyeCenterY - currentH - 4.5f * scale);
            pEnd   = D2D1::Point2F(eyeCenterX + 7.5f * scale, eyeCenterY - currentH - 1.2f * scale);
        } else {
            // Right eyebrow slants down toward nose /
            pStart = D2D1::Point2F(eyeCenterX - 7.5f * scale, eyeCenterY - currentH - 1.2f * scale);
            pEnd   = D2D1::Point2F(eyeCenterX + 9.0f * scale, eyeCenterY - currentH - 4.5f * scale);
        }

        target_->DrawLine(pStart, pEnd, browBrush, browStroke);
        target_->FillEllipse(D2D1::Ellipse(pStart, browStroke * 0.5f, browStroke * 0.5f), browBrush);
        target_->FillEllipse(D2D1::Ellipse(pEnd, browStroke * 0.5f, browStroke * 0.5f), browBrush);
    }
}
```

### 2.3. Floating Vector "Zzz" Dream Effect
Tieng Viet: Xoa bo hoan toan giot nuoc mieng, thay the bang 3 chu Z vector bay luan phien tu khoe mat phai, phong to dan va mo dan theo nhip tho ngu say.

```cpp
// File: dynamic-island-for-windows.wh.cpp (DrawCompanionEyes)
if (isDeepSleeping && !isLeft) {
    EnsureRoundJoinStyle();
    const double sleepDuration = (now - g_companionEyes.sleepStartTime);

    for (int zIdx = 0; zIdx < 3; ++zIdx) {
        const float cycleDuration = 2.4f;
        const float timeOffset = zIdx * 0.8f;
        const float progress = fmodf(static_cast<float>(sleepDuration + timeOffset), cycleDuration) / cycleDuration;

        // Wave trajectory: rises upward and drifts gently to the right
        const float driftX = (10.0f + 16.0f * progress + 3.0f * sinf(progress * 6.283f)) * scale;
        const float riseY = (2.0f + 20.0f * progress) * scale;
        const float zOriginX = eyeCenterX + driftX;
        const float zOriginY = eyeCenterY - riseY;

        // Dynamic scaling: small z near face growing to larger Z higher up
        const float zW = (3.8f + 3.6f * progress) * scale;
        const float zH = (4.5f + 4.2f * progress) * scale;
        const float zStroke = (1.3f + 0.5f * progress) * scale;

        // Smooth opacity envelope: fade-in and fade-out
        float zAlpha = 1.0f;
        if (progress < 0.20f) {
            zAlpha = progress / 0.20f;
        } else if (progress > 0.65f) {
            zAlpha = (1.0f - progress) / 0.35f;
        }
        zAlpha = Clamp(zAlpha, 0.0f, 1.0f) * 0.90f * settingsOpacity_;

        if (zAlpha > 0.02f) {
            D2D1_COLOR_F zCol = D2D1::ColorF(0.40f, 0.82f, 1.0f, zAlpha);
            if (ID2D1SolidColorBrush* zBrush = AcquireBrush(zCol)) {
                ComPtr<ID2D1PathGeometry> zPath;
                if (SUCCEEDED(d2dFactory_->CreatePathGeometry(&zPath))) {
                    ComPtr<ID2D1GeometrySink> sink;
                    if (SUCCEEDED(zPath->Open(&sink))) {
                        sink->BeginFigure(D2D1::Point2F(zOriginX, zOriginY), D2D1_FIGURE_BEGIN_HOLLOW);
                        sink->AddLine(D2D1::Point2F(zOriginX + zW, zOriginY));
                        sink->AddLine(D2D1::Point2F(zOriginX, zOriginY + zH));
                        sink->AddLine(D2D1::Point2F(zOriginX + zW, zOriginY + zH));
                        sink->EndFigure(D2D1_FIGURE_END_OPEN);
                        sink->Close();
                        target_->DrawGeometry(zPath.Get(), zBrush, zStroke, roundJoinStyle_.Get());
                    }
                }
            }
        }
    }
}
```

### 2.4. Music Playback State Gating
Tieng Viet: Khi he thong dang phat nhac, chan hoan toan trang thai ngu. Neu dang ngu ma bat nhac se tu dong tinh giac de hoa theo nhip dieu.

```cpp
// File: dynamic-island-for-windows.wh.cpp (DrawCompanionEyes)
const bool isPlayingMusic = (state.media.available && state.media.playing);

// Deep sleep only allowed when idle >= 10s AND no music is playing
if (idleMs >= 10000 && !isPlayingMusic) {
    if (!g_companionEyes.isDeepSleeping) {
        g_companionEyes.isDeepSleeping = true;
        g_companionEyes.sleepStartTime = now;
    }
} else if (g_companionEyes.isDeepSleeping) {
    g_companionEyes.isDeepSleeping = false;
    if (!isPlayingMusic) {
        g_companionEyes.startledEndTime = now + 0.6;
        g_companionEyes.angryEndTime = now + 0.6 + 2.5;
        TriggerNudge();
    }
}

const bool isDeepSleeping = g_companionEyes.isDeepSleeping && !isPlayingMusic;
const bool isStartled = (now < g_companionEyes.startledEndTime) && !isPlayingMusic;
const bool isAngry = (!isStartled && now < g_companionEyes.angryEndTime) && !isPlayingMusic;
const bool isSleepy = !isPlayingMusic && !isDeepSleeping && (isNight || isLowBattery) && ...;
```

### 2.5. High-Refresh-Rate Render Loop (300Hz+)
Tieng Viet: Xoa bo gioi han 60 FPS cua continuousAnimation, tich hop DwmFlush khoa thang vao tan so quet cua man hinh va bo dem thoi gian chinh xac duoi 1ms.

```cpp
// File: dynamic-island-for-windows.wh.cpp (RenderThreadProc)
if (!needsRender) {
    WaitForSingleObject(g_stopEvent, (stripActive && !springsAnimating) ? 50 : 16);
    nextFrameTarget = std::chrono::steady_clock::now();
} else if ((springsAnimating || continuousAnimation) && g_settings.targetFps <= 0 && SUCCEEDED(DwmFlush())) {
    // Hardware VSync lock via Desktop Window Manager (144Hz - 360Hz+)
    nextFrameTarget = std::chrono::steady_clock::now();
} else {
    nextFrameTarget += std::chrono::duration_cast<std::chrono::steady_clock::duration>(
        std::chrono::duration<double, std::milli>(targetFrameMs));

    auto nowTime = std::chrono::steady_clock::now();
    if (nowTime < nextFrameTarget) {
        double remainingMs = std::chrono::duration<double, std::milli>(nextFrameTarget - nowTime).count();
        if (remainingMs >= 2.0) {
            WaitForSingleObject(g_stopEvent, static_cast<DWORD>(remainingMs - 1.0));
        }
        if ((springsAnimating || continuousAnimation) && targetFrameMs < 10.0) {
            while (std::chrono::steady_clock::now() < nextFrameTarget &&
                   WaitForSingleObject(g_stopEvent, 0) == WAIT_TIMEOUT) {
                std::this_thread::yield();
            }
        }
    } else {
        nextFrameTarget = nowTime;
    }
}
```

### 1.12. Tối Ưu Tương Tác Chóng Mặt (Dizzy Shake Detection) Chuẩn Mọi Tần Số Quét (60Hz – 360Hz+)
- **Vấn đề:** Điều kiện nhận diện đảo chiều lắc chuột trước đây bị ràng buộc bởi độ dời cố định theo từng frame (`mouseDeltaX > 32.0f`). Ở các màn hình tần số quét cao (120Hz, 144Hz, 240Hz+), thời gian 1 frame quá ngắn dẫn đến độ dời mỗi frame chỉ khoảng 8px - 14px, khiến người dùng hầu như không thể kích hoạt hiệu ứng chóng mặt dù lắc rất nhanh.
- **Giải pháp:**
  1. **Chuẩn hóa vector vận tốc độc lập với tần số quét:** Tính vận tốc tức thời `curVx = mouseDeltaX / dtF` và `curVy = mouseDeltaY / dtF` theo đơn vị $px/s$.
  2. **Ngưỡng vận tốc đảo chiều chuẩn:** Sử dụng ngưỡng vận tốc `kReversalVelThreshold = 650.0f px/s` và tổng tốc độ lắc `kMinShakeSpeed = 1400.0f px/s`, giúp nhận diện chính xác cử chỉ lắc tay zigzag trên cả màn hình 60Hz lẫn 144Hz - 360Hz.
  3. **Lưu hướng vận tốc qua điểm dừng:** Chỉ cập nhật hướng vận tốc khi vượt ngưỡng, không bị mất dấu ở điểm chuyển tiếp khi chuột giảm tốc trước khi đổi chiều.
  4. **Ưu tiên tương tác chóng mặt:** Khi phát hiện cử chỉ lắc chuột deliberate (4 lần đảo chiều), lập tức xóa trạng thái giật mình/cáu kỉnh sau giấc ngủ để con ngươi xoay tròn (swirl) ngay tức khắc.
  5. **Mở to mắt & khóa nháy mắt khi xoay:** Trong 2.2 giây chóng mặt, mắt mở rộng 1.15x và tạm ngừng nháy mắt để hiệu ứng xoay tròn con ngươi hiển thị rõ nét và sống động nhất.

### 1.13. Clipboard Manager (Bộ Nhớ Tạm Lịch Sử Sao Chép)
- **Tính năng có sẵn mặc định:** Không cần vào cài đặt để bật.
- **Kích hoạt nhanh bằng phím tắt:** Giữ phím `Ctrl + C` lâu (> 0.35s) từ bất kỳ ứng dụng nào $\rightarrow$ Dynamic Island tự động bung mở hiển thị ngay thẻ **Clipboard History** (Bộ nhớ tạm).
- **Hiển thị trực quan & Thumbnail ảnh:**
  - Tự động lưu 10–15 mục đã copy gần nhất (cả văn bản, link và hình ảnh).
  - Đối với hình ảnh: Tự động render thumbnail preview sắc nét 22px dạng bo tròn squircle.
  - Đối với văn bản: Hiển thị preview 2 dòng, đếm ký tự và icon ứng dụng gốc (Chrome, Word, VS Code...).
- **Cuộn qua các mục sao chép:** Hỗ trợ lăn chuột (mouse wheel) cuộn qua danh sách mượt mà.
- **Click để Paste trực tiếp:** Bấm chuột vào mục bất kỳ $\rightarrow$ tự động nạp vào bộ nhớ clipboard của Windows, khôi phục focus về ứng dụng đang soạn thảo và giả lập phím `Ctrl + V` để dán ngay lập tức vào văn bản $\rightarrow$ đóng Dynamic Island.
- **Nút dọn sạch (Bin icon):** Bấm một chạm để xóa sạch toàn bộ lịch sử sao chép.

### 1.14. Hộp Drop Bộ Nhớ Tạm Kéo Thả File (File Tray Proximity & Sticky Auto-Pinned)
- **Tính năng có sẵn mặc định:** `FileTrayModule: true` được kích hoạt sẵn.
- **Cảm biến lại gần (Drop Proximity Detection):** Mở rộng vùng đón drag trong `WM_NCHITTEST` (160px theo chiều ngang, 140px theo chiều dọc bao trùm kích thước bung mở của khay). Khi người dùng kéo file từ Windows Explorer hoặc trình duyệt lại gần Dynamic Island, hệ thống nhận diện drag và lập tức mở bung rộng hộp drop **File Tray** trước khi người dùng kịp thả file xuống.
- **Xử lý triệt để xung đột WS_EX_TRANSPARENT & UIPI Filter:**
  1. Khi ở chế độ Idle/Thu nhỏ, cửa sổ overlay của Dynamic Island thường mang thuộc tính `WS_EX_TRANSPARENT` (click-through) khiến Windows OLE Drag & Drop bỏ qua hoàn toàn. Đã bổ sung bộ phát hiện tiệm cận `nearTrayDrag`: ngay khi con trỏ chuột giữ file (`VK_LBUTTON`) đi vào vùng bán kính tiếp cận của khay, `SetClickThrough(hwnd, false)` được gọi ngay lập tức để gỡ bỏ `WS_EX_TRANSPARENT`.
  2. Bổ sung `ChangeWindowMessageFilterEx` cho `WM_DROPFILES`, `WM_COPYDATA`, và message `0x0049` (WM_COPYGLOBALDATA), cho phép nhận file kéo thả mượt mà từ Explorer kể cả khi tiến trình có quyền quản trị (UIPI).
  3. Bổ sung `DragAcceptFiles(hwnd, TRUE)` trong `WM_CREATE` hỗ trợ đồng thời cả OLE IDropTarget lẫn Win32 Shell Drop.
- **Luôn mở cửa sổ khi có file trong bộ nhớ tạm (Sticky Auto-Pinned):**
  - Khi thả file vào: Hệ thống tự động chuyển sang chế độ ghim mở (`g_trayAutoPinned = true`). Dynamic Island sẽ **luôn luôn mở liên tục**, không tự động co lại kể cả khi người dùng rê chuột ra ngoài làm việc khác.
  - Cho phép người dùng dễ dàng xem các file tạm đã đỗ ở đây, kéo ra ứng dụng khác bất kỳ lúc nào.
- **Tự động biến mất khi lấy hết file ra:**
  - Khi người dùng kéo file từ File Tray thả sang ứng dụng khác (Discord, Email, Explorer...) hoặc bấm nút xóa (X) / nút dọn sạch (Bin) để lấy file ra khỏi bộ nhớ tạm:
  - Ngay khi file cuối cùng được lấy ra (`fileTrayItems.empty()`), Dynamic Island sẽ tự động gỡ ghim và thu nhỏ biến mất hoàn toàn.

### 1.15. Khắc Phục Biểu Tượng Cấm Thả File (Fix DROPEFFECT_NONE / 🚫 Icon)
- **Vấn đề phát hiện:** Khi kéo file từ Desktop hoặc Windows Explorer vào Dynamic Island, con trỏ chuột xuất hiện biểu tượng cấm màu đỏ (🚫 `DROPEFFECT_NONE`), không cho phép thả file.
- **Nguyên nhân cốt lõi:**
  1. `TrayModuleEnabled()` đọc từ cấu hình Windhawk đã lưu trước đó của người dùng. Nếu mod từng được chạy khi `FileTrayModule` còn ở giá trị `false`, giá trị cache trong Windhawk khiến module bị coi là tắt $\rightarrow$ `DragEnter` trả về `DROPEFFECT_NONE`.
  2. `PickTrayDropEffect` trước đây chỉ chấp nhận `DROPEFFECT_COPY` và `DROPEFFECT_LINK`. Khi người dùng kéo file trong cùng một ổ đĩa từ Windows Explorer hoặc Desktop, Windows mặc định gửi hiệu ứng `DROPEFFECT_MOVE`. Do thiếu cờ này, hệ thống từ chối nhận và trả về biểu tượng cấm 🚫.
  3. `TrayDataAcceptable` chỉ gọi `QueryGetData` đơn thuần cho `CF_HDROP`, có thể bị từ chối trên Windows 11 Explorer thế hệ mới nếu định dạng ưu tiên là `CFSTR_SHELLIDLIST`.
- **Giải pháp triệt để:**
  1. **Bật mặc định vĩnh viễn:** Đặt `TrayModuleEnabled() { return true; }` và `next.fileTrayModule = true;` để đảm bảo khay file luôn hoạt động ngay tức thì mà không phụ thuộc vào trạng thái cache cài đặt.
  2. **Chấp nhận mọi chế độ kéo thả:** Nâng cấp `PickTrayDropEffect` và `PickTrayDropEffectAny`: Chuyển đổi an toàn `DROPEFFECT_MOVE` thành nhận diện tham chiếu `DROPEFFECT_COPY`, tuyệt đối không bao giờ trả về `DROPEFFECT_NONE` (🚫) khi đang kéo file hợp lệ.
  3. **Hỗ trợ định dạng kép (CF_HDROP & Shell IDList):** `TrayDataAcceptable` và `ExtractDropPaths` hiện hỗ trợ cả chuẩn `CF_HDROP` cổ điển lẫn `CFSTR_SHELLIDLIST` hiện đại thông qua `IEnumFORMATETC` và `SHGetPathFromIDListW`, tương thích 100% với mọi phiên bản Windows 10/11, Desktop và Windows Explorer.

### 1.16. Khóa Cứng Tỷ Lệ 1:1 Pixel & Chuyển Cảnh Cross-Dissolve Thuần Khiết Chuẩn Apple
- **Hiện tượng:** Khi cuộn chuột qua lại giữa các tab (Đồng hồ $\leftrightarrow$ Lịch $\leftrightarrow$ Thời tiết $\leftrightarrow$ Hardware Monitor $\leftrightarrow$ File Tray $\leftrightarrow$ Clipboard History), font chữ số lớn bị phồng to, co giãn béo nét rồi thu lại do lệnh biến đổi ma trận hình học (`Matrix3x2F::Scale`) và độ dời trục dọc (`Translation(0, dy)`).
- **Khắc phục triệt để:**
  1. **Khóa cứng tỷ lệ 1:1 Pixel nguyên bản:**
     - Xóa bỏ hoàn toàn `D2D1::Matrix3x2F::Scale` và `D2D1::Matrix3x2F::Translation` trong `DrawTabTransition`.
     - Thay thế bằng `target_->SetTransform(oldTransform)` trước khi vẽ nội dung tab.
     - Chấm dứt 100% hiện tượng chữ số bị co giãn, phóng to hay béo dính nét khi vuốt/kéo chuyển tab.
  2. **Chuyển cảnh Cross-Dissolve thuần khiết chuẩn Apple:**
     - Áp dụng công thức làm mờ dốc: `const float alpha = std::pow(1.0f - eased, 2.8f);`
     - Khi chuyển tab, tab cũ mờ nhanh chóng và biến mất dứt khoát trước khi tab mới đi vào tâm điểm. Hai tab không bị trượt dọc đè bóng lên nhau, chuyển cảnh êm dịu, tĩnh tại như widget iOS / Apple Watch.
  3. **Đồng bộ font số lớn sang Semi-Bold (600):**
     - Đặt `timeDashboardFormat_` (56px) và `calDayLargeFormat_` (63px) sang `DWRITE_FONT_WEIGHT_SEMI_BOLD` (600), nét số thanh thoát, tinh tế và sắc nét chuẩn thiết kế Apple.

### 1.17. Tinh Chỉnh Triệt Để Tab Lịch (Calendar Hero Card & Grayscale Antialiasing)
- **Vấn đề phát hiện:** Tab Lịch khi hiển thị hoặc chuyển cảnh vẫn còn cảm giác bị cấn, dày nét hoặc đè nhẹ viền.
- **Nguyên nhân:**
  1. Con số ngày (`calDayLargeFormat_`) ban đầu có kích thước quá lớn (63px) bên trong khung Hero Card rộng chỉ 93px, gây chật chội và sát mép viền năm (`wYear`) ở trên và thứ (`weekday`) ở dưới.
  2. Lệnh vẽ số ngày sử dụng cờ `D2D1_DRAW_TEXT_OPTIONS_NONE` thay vì `D2D1_DRAW_TEXT_OPTIONS_CLIP`, khiến phần đuôi chữ số có thể bị lem viền.
  3. Direct2D trên cửa sổ trong suốt mặc định sử dụng ClearType (Subpixel AA). Khi đổi qua lại giữa các lớp mờ (layer alpha), ClearType bị đổi qua Grayscale AA rồi đổi ngược lại gây hiện tượng "nở nét chữ" (Font Thickness Bloom).
  4. Ngưỡng chuyển tiếp chiều cao giữa chế độ thu nhỏ và mở rộng (`collapsedAlpha` và `expandedAlpha`) từng bị chồng chéo ở khoảng 56px–70px khiến cả hai giao diện cùng render đè mờ lên nhau.
- **Giải pháp hoàn thiện:**
  1. **Khóa Grayscale AA toàn cục cho Direct2D:** Gọi `target_->SetTextAntialiasMode(D2D1_TEXT_ANTIALIAS_MODE_GRAYSCALE)` ngay sau `BeginDraw()`, giúp mọi font chữ trên cửa sổ trong suốt luôn hiển thị sắc nét, đồng nhất 100%, chấm dứt hoàn toàn hiện tượng nở nét chữ.
  2. **Tinh chỉnh tỷ lệ số ngày Hero Card:** Giảm kích thước số ngày từ 63px xuống **50px Semi-Bold** chuẩn tỉ lệ vàng, kết hợp khoảng cách tọa độ an toàn (`top + 36px` cách xa năm `top + 34px`), kèm cờ `D2D1_DRAW_TEXT_OPTIONS_CLIP` để số ngày luôn nằm gọn gàng, tinh tế bên trong khối thẻ Hero.
  3. **Tách biệt ngưỡng thu nhỏ / mở rộng:** Đặt `collapsedAlpha` tắt hẳn ở 52px trước khi `expandedAlpha` bắt đầu ở 56px, triệt tiêu 100% tình trạng giao thoa đè hình.

---

## 3. DANH SACH TEP TIN DA CAP NHAT VA BIEN DICH

1. `dynamic-island-for-windows.wh.cpp`
   - Primary Windhawk mod source code containing all features and fixes.
2. `sync_to_windhawk.bat`
   - Tap lenh tu dong dong bo ma nguon sang thu muc he thong cua Windhawk (`C:\ProgramData\Windhawk\ModsSource\` va `C:\ProgramData\Windhawk\EditorWorkspace\mod.wh.cpp`) voi quyen Administrator.

3. `DynamicIsland_Full.exe`
   - Ban thuc thi doc lap day du tinh nang (kich thuoc: 1,862,656 bytes).
   - Da duoc bien dich thanh cong voi c standard C++20 va toi uu hoa O2.

4. `cyber_island.cpp` va `cyber_island.exe`
   - Ban demo bieu cam mat doc lap gon nhe (kich thuoc: 598,016 bytes).
   - Da tich hop DwmFlush ho tro 300Hz+ va hieu ung Zzz.

