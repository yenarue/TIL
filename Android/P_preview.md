Android P Preview
====
* [Migration CheckList](https://developer.android.com/distribute/best-practices/develop/target-sdk)

# Messaging Style
알림에 다양한 정보 출력 가능. 인물/사진 등...

# ImageDocoder
`Bitmap` & `Drawable`용 `ImageDecoder`. `BitmapFactory`를 대체.
* 보다 정확한 스케일링
* 하드웨어 메모리에 대한 단일 단계 티코딩 지원 (Single step decoding)
* 후처리 및 디코딩 지원
* 애니메이션 이미지 디코딩

# JobScheduler Network Updates

# Display Cutout Bounds
`WindowsManagert.LayoutParams.LAYOUT_IN_DISPLAY_CUTOUT_MODE_ALWAYS`

# Dual Camera Support for Apps

# HDR VP9 Video

# OIS / Disply-baed Flash

# Wifi RTT (Round Trip Time)
실내위치를 활용할 수 있는 하드웨어를 지원하는 안드로이드 기기에서, 해당 프로토콜을 지원하는 가까운 AP에 연결. 실제로 연결되는건 X
* 3개의 AP 거리를 계산해서 위치 추정 (삼각측량법)
* 정확도 1~2m (헐)