### input
- 100개의 이미지
- 100개의 transpose값
	pose는 4x4 [[Extrinsic-Parameter]], 각각의 이미지를 3d상의 위치로 변환시켜주는 행렬

### Train
한 iteration 마다 하나의 이미지를 random sampling 하여 모델에 집어 넣음
##### Model input
- ray의 방향 d
- ray내부에 포함되는 point들의 3d좌표값 (x,y,z)

#### Prediction
입력받은 x,y,z 좌표와 d로부터 mlp 모델은 각 포인트들의 color, density를 예측
이렇게 한 ray에 대해서 sampling된 point들의 예측된 값들을 [[#Volume Rendering]] 작업을 통해 하나의 pixel 값으로 만들어 줌
해당 pixel의 rgb값과 비교해 lose를 구하는 방식으로 학습이 진행

pose값을 input으로 넣고, output으로 예측된 이미지가 나오면 실제 이미지와 비교하여 loss를 구하는 방식으로 학습이 진행됨

### Model
8개의 Linear layer로 이루어진 MLP 모델
3D 좌표를 포함하는 [images x smaple points : 3]이 input으로 들어가고

[[#Positional Encoding]] 과정을 거쳐 [images x smaple points : 63]이 input으로 들어감
8개의 layer를 거쳐 density output을 뽑고, ray의 d값을 input으로 넣어 최종 rgb output을 얻어냄

물체를 바라보는 방향에 따라 물체를 바라보는 rgb값이 달라진다는 개념

### Volume Rendering
model의 output으로 나온 한 ray의 color와 density 값들은 한 pixel로 합쳐지는 volume rendering 과정을 거침
합쳐진 rgb값은 실제 이미지와 MSE loss??? 를 거쳐 back propagation을 통해 학습이 진행됨

ray 내에서 point들을 sampling 할 때, 물체가 있을법한 범위를 정해두고 가장 가까운 point를 near, 먼 point를 far로 정해둠

$$C(r) = \int_{t_n}^{t_f}T(t) \sigma(r(t))c(r(t),d)\mathrm{d}x, \ where \ \ T(t)=exp\bigg(-\int_{t_n}^{t_f}\sigma(r(s))ds\bigg)$$
$$r(t)=o+td$$

여기서 투과도 $T(t)$ 는 near와 far사이의 t에서의 점의 weight 값은 near로 부터 t까지의 density 값을 더한 값이 클 수록 작아진다는 개념
앞쪽에 가려진 부분의 density가 클수록, weight 값이 작아진다

$$t_i \sim u\bigg[t_n+\frac{i-1}{N}(t_f-t_n), t_n+\frac{i}{N}(t_f-t_n)\bigg].$$
*랜덤 샘플링*
$$\hat{C}(r) = \sum_{i=1}^{N}T_i\big(1-exp(-\sigma_i\delta_i)\big)c_i,\ where T_i=exp\bigg(-\sum_{j=1}^{i-1}\sigma_j\delta_j\bigg)$$


### Positional Encoding
input 3D 좌표의 정보를 늘려주는 과정
$$\gamma(p) = \big(sin(2^0\pi p),cos(2^0\pi p),\cdots,sin(2^{L-1}\pi p),cos(2^{L-1}\pi p)\big)$$



### Hierarchical Volume Sampling
$$\hat{C}_c(r)=\sum_{i=1}^{N_c}w_ic_i, \ w_i=T_i\big(1-exp(-\sigma_i\delta_i)\big)$$
Coarse Model에서 구한 weight 값 (density X Transmittance)을 nomalize하여 piecewise-constant PDF(Probability Density Function) 을 구하고, 이 ray 내의 distribution으로부터 point들을 다시 sampling
Density가 높은 값들을 위주로 다시 sampling 하여 volume rendering을 시킴
$$L=\sum_{r\in R}\bigg[\parallel \hat{C_c}(r)-C(r) \parallel^2_2 \bigg] + \bigg[\parallel \hat{C_f}(r)-C(r) \parallel^2_2 \bigg]$$
Coarse model과 실제 값과의 loss, Fine model과 실제 값과의 loss를 각각 구하고 더해서 전체 loss를 정함


---
