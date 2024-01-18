### Abstract
We introduce a method to render Neural Radiance Fields (NeRFs) in real time using PlenOctrees, an octree-based 3D representation which supports view-dependent effects.
Our method can render 800×800 images at more than 150 FPS, which is over 3000 times faster than conventional NeRFs.
We do so without sacrificing quality while preserving the ability of NeRFs to perform free-viewpoint rendering of scenes with arbitrary geometry and view-dependent effects. Real-time performance is achieved by pre-tabulating the NeRF into a PlenOctree. 
In order to preserve view-dependent effects such as specularities, we factorize the appearance via closed-form spherical basis functions. Specifically, we show that it is possible to train NeRFs to predict a spherical harmonic representation of radiance, removing the viewing direction as an input to the neural net- work. 
Furthermore, we show that PlenOctrees can be directly optimized to further minimize the reconstruction loss, which leads to equal or better quality compared to com- peting methods.
Moreover, this octree optimization step can be used to reduce the training time, as we no longer need to wait for the NeRF training to converge fully. Our real-time neural rendering approach may potentially en- able new applications such as 6-DOF industrial and prod- uct visualizations, as well as next generation AR/VR sys- tems. 
PlenOctrees are amenable to in-browser rendering as well; please visit the project page for the interactive online demo, as well as video and code:

PlenOctrees를 사용하여 NeRF를 실시간으로 렌더링하는 방법을 소개합니다. 
이 방식은 800×800 이미지를 150FPS 이상으로 렌더링할 수 있으며, 이는 기존 NeRF보다 3000배 이상 빠른 속도입니다. 
또한 임의의 지오메트리와 뷰 종속 효과가 있는 장면의 자유 시점 렌더링을 수행하는 NeRF의 기능을 유지하면서 품질 저하 없이 렌더링할 수 있습니다. 
실시간 성능은 NeRF를 PlenOctree에 사전 테이블링하여 달성합니다. 
Specularlity(거울같은) 와 같은 뷰 종속적 효과를 보존하기 위해 폐쇄형 구형 기저 함수를 통해 광도를 인수분해합니다. 
특히, 우리는 신경망의 입력으로 보는 방향을 다시 이동하여 광채의 구형 고조파 표현을 미리 지시하도록 NeRF를 훈련할 수 있음을 보여줍니다. 
또한 재구성 손실을 최소화하기 위해 PlenOctree를 직접적으로 최적화할 수 있으며, 이를 통해 컴펫팅 방식과 비교했을 때 동등하거나 더 나은 품질을 얻을 수 있음을 보여줍니다. 
또한 이 옥트리 최적화 단계는 NeRF 훈련이 완전히 수렴될 때까지 기다릴 필요가 없기 때문에 훈련 시간을 단축하는 데 사용할 수 있습니다. 
우리의 실시간 뉴럴 렌더링 접근 방식은 6-DOF 산업 및 제품 시각화, 차세대 AR/VR 시스템과 같은 새로운 애플리케이션을 구현할 수 있습니다. 

### Introduction
...중략...

Although one could convert an existing NeRF into such a representations via projection onto the SH basis functions, we show that we can in fact modify a NeRF network to predict appearances explicitly in terms of spherical harmonics.
Specifically, we train a network that produces coefficients for the SH functions instead of raw RGB values, so that the predicted values can later be directly stored within the leaves of the PlenOctree.

SH 기저 함수에 투영하여 기존 NeRF를 이러한 표현으로 변환할 수도 있지만, 우리는 실제로 구형 고조파 측면에서 명시적으로 모양을 미리 구술하도록 NeRF 네트워크를 수정할 수 있음을 보여줍니다. 
특히 원시 RGB 값 대신 SH 함수에 대한 계수를 생성하는 네트워크를 훈련하여 예측된 값을 나중에 PlenOctree의 잎사귀에 직접 저장할 수 있도록 합니다. 

We also introduce a sparsity prior during NeRF training to improve the memory efficiency of our octrees, consequently allowing us to render higher quality images.
Furthermore, once the structure is created, the values stored in PlenOctree can be optimized because the rendering procedure remains differentiable.
This enables the PlenOctree to obtain similar or better image quality compared to NeRF. Our pipeline is illustrated in Fig. 2.

또한 NeRF 훈련 중에 sparsity prior를 도입하여 옥트리의 메모리 효율을 개선함으로써 결과적으로 더 높은 품질의 이미지를 렌더링할 수 있습니다. 
또한 구조가 생성되면 렌더링 프로시저가 차별성을 유지하기 때문에 PlenOctree에 저장된 값을 최적화할 수 있습니다. 
이를 통해 PlenOctree는 NeRF와 비교하여 비슷하거나 더 나은 이미지 품질을 얻을 수 있습니다. 우리의 파이프라인은 그림 2에 나와 있습니다.
### Method
We propose a pipeline that enables real-time rendering of NeRFs. 
Given a trained NeRF, we can convert it into a PlenOctree, an efficient data structure that is able to represent non-Lambertian effects in a scene. 
Specifically, it is an octree which stores spherical harmonics (SH) coefficients at the leaves, encoding view-dependent radiance.

우리는 NeRF를 실시간으로 렌더링할 수 있는 파이프라인을 제안합니다. 
훈련된 NeRF가 주어지면 장면에서 비램버시안 효과를 표현할 수 있는 효율적인 데이터 구조인 PlenOctree로 변환할 수 있습니다. 
구체적으로는 나뭇잎에 구형 고조파(SH) 계수를 저장하는 옥트리로, 뷰에 따라 달라지는 광도를 인코딩합니다.

To make the conversion to PlenOctree more straightforward, we also propose NeRF-SH, a variant of the NeRF network which directly outputs the SH coefficients, thus eliminating the need for a view-direction input to the network. 
With this change, the conversion can then be performed by evaluating on a uniform grid followed by thresholding. 
We fine-tune the octree on the training images to further improve image quality, Please see Fig. 2 for a graphical illus- tration of our pipeline.

PlenOctree로의 변환을 보다 간단하게 하기 위해 저희는 SH 계수를 직접 출력하여 네트워크에 뷰 방향 입력이 필요 없는 NeRF 네트워크의 변형인 NeRF-SH를 제안합니다. 
이렇게 변경하면 균일한 그리드에서 평가한 다음 임계값을 설정하여 변환을 수행할 수 있습니다. 
이미지 품질을 더욱 향상시키기 위해 훈련 이미지의 옥트리를 미세 조정합니다. 
파이프라인의 그래픽 일루젼은 그림 2를 참조하세요.

The conversion process leverages the continuous nature of NeRF to dynamically obtain the spatial structure of the octree. We show that even with a partially trained NeRF, our PlenOctree is capable of producing results competitive with the fully trained NeRF.

변환 프로세스는 NeRF의 연속적인 특성을 활용하여 옥트리의 공간 구조를 동적으로 얻습니다.
부분적으로 훈련된 NeRF를 사용하더라도 PlenOctree가 완전히 훈련된 NeRF와 경쟁할 수 있는 결과를 생성할 수 있음을 보여줍니다.