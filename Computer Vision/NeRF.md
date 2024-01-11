>Neural Radiance Fields
>
>[[Novel View Synthesis]] 분야에서, point cloud나 mesh, voxel과 같은 `3D Object` 자체를 렌더링 하는 거이 아닌, 3D Object를 바라본 `이미지` 들을 예측할 수 있는 모델을 만드는 것이 목표
---
### 개념
[[<NeRF> 개념]] : [link](Computer_Vision/<NeRF>_개념.md)

### 단점
- Training, Rendering 속도가 느림
- Static한 Scene에 대해서만 성능이 좋음
- 같은 환경 (밝기, 색상 등)에서 촬영한 이미지에 대해서만 성능이 좋음
- 한 Model로 하나의 물체만 만들어낼 수 있음
- 다양한 시점의 training set이 필요
- input으로 camera parameter값이 필요