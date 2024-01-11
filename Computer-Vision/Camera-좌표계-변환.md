>어떤 물체의 실제 좌표, 카메라 좌표, 사진 좌표, 픽셀 좌표간 변환을 할 때 카메라의 정보들이 사용된다
>실제 좌표 → 픽셀 좌표를 Forward projection이라 부르고
>픽셀 좌표 → 실제 좌표를 Backward projection이라 부른다
---
### Camera Coords → Film Coords
카메라의 초점거리를 f라고 할 때, 대상 물체의 좌표 X,Y, 대상 물체와의 거리 Z를 활용한 카메라 내의 좌표 x,y는

$$x=f\frac{X}{Z},y=f\frac{Y}{Z}$$
- 2D 와 3D 좌표간 변환
$$\begin{bmatrix}x'\\y'\\z'\end{bmatrix}=\begin{bmatrix}f&0&0&0\\0&f&0&0\\0&0&1&0\end{bmatrix}\begin{bmatrix}x\\y\\z\\1\end{bmatrix}$$

### World Coords → Camera Coords
다른 좌표계에서 바라본 같은 점을 각각 $P_c, P_w$라고 하고, 
좌표계의 원점을 다른 좌표계의 원점으로 이동하는 값을 C, 
좌표계를 다른 좌표계와 동일하게 회전하는 값을 R이라고 할 때 식은
$$P_c=R(P_w-C)$$
$$\begin{bmatrix}X\\Y\\Z\\1\end{bmatrix}=\begin{bmatrix}r_{11}&r_{12}&r_{13}&0\\r_{21}&r_{22}&r_{23}&0\\r_{31}&r_{32}&r_{33}&0\\0&0&0&1\end{bmatrix}\begin{bmatrix}1&0&0&-c_x\\0&1&0&-c_y\\0&0&1&c_z\\0&0&0&1\end{bmatrix}\begin{bmatrix}x\\y\\z\\1\end{bmatrix}$$
로 표현할 수 있다
- Stereo Camera
	Stereo Camera의 방향 값은 동일하며, 왼쪽 카메라와 오른쪽 카메라의 거리 차이가 X좌표에서 난다고 할 때 식은
	
$$\begin{bmatrix}X\\Y\\Z\\1\end{bmatrix}=\begin{bmatrix}r_{11}&r_{12}&r_{13}&0\\r_{21}&r_{22}&r_{23}&0\\r_{31}&r_{32}&r_{33}&0\\0&0&0&1\end{bmatrix}\begin{bmatrix}1&0&0&T_x\\0&1&0&0\\0&0&1&0\\0&0&0&1\end{bmatrix}\begin{bmatrix}x\\y\\z\\1\end{bmatrix}$$
	이를 각각의 Camera 기준 Film Coords로 나타내면
	왼쪽 카메라
$$\begin{bmatrix}x_l\\y_l\\1\end{bmatrix}=\begin{bmatrix}f&0&0&0\\0&f&0&0\\0&0&1&0\end{bmatrix}\begin{bmatrix}1&0&0&0\\0&1&0&0\\0&0&1&0\\0&0&0&1\end{bmatrix}\begin{bmatrix}X\\Y\\Z\\1\end{bmatrix}$$
	오른쪽 카메라
$$\begin{bmatrix}x_r\\y_r\\1\end{bmatrix}=\begin{bmatrix}f&0&0&0\\0&f&0&0\\0&0&1&0\end{bmatrix}\begin{bmatrix}1&0&0&-T_x\\0&1&0&0\\0&0&1&0\\0&0&0&1\end{bmatrix}\begin{bmatrix}X\\Y\\Z\\1\end{bmatrix}$$
$$x_r=f\frac{X-T_x}{Z}, y_r=f\frac{Y}{Z}$$
### Film Coords → Pixel Coords
픽셀 방향과 film 방향이 일치할 때, (x+, y-)
$$u=f\frac{X}{Z}+O_x,v=f\frac{X}{Z}+O_y,$$
여기서 보는 방향에 따른 원근감을 준다면
$$\begin{bmatrix}x'\\y'\\z'\end{bmatrix}=\begin{bmatrix}f/S_x&0&o_x&0\\0&f/S_y&o_y&0\\0&0&1&0\end{bmatrix}\begin{bmatrix}x\\y\\z\\1\end{bmatrix}$$
$$u=\frac{1}{s_x}f\frac{X}{Z}+o_x,v=\frac{1}{s_y}f\frac{Y}{Z}+o_y$$
---
