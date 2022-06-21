# AIFFELthon_개인_프로젝트

AIFFELthon은 SOCAR 데이터를 제공받아 사용하기 때문에 깃허브에 올리기 전 검수를 받아야 합니다  
때문에 현재 프로젝트 파일을 확인할 수 없다는 점 양해 부탁드립니다  

<br/>

대신 제가 프로젝트를 어떻게 했는지 정리하고자 합니다

## 문제 정의

사용자가 파손된 차량 부위를 사진으로 찍으면,  
모델이 파손된 부위를 segmentation해 어떤 파손이 감지되었는지 사용자에게 알려주는 서비스를 제작  

파손 class로는 A,B,C가 있다


## 1. semantic segmentation 혹은 instance segmentation

segmentation은 크게 두 방식으로 나뉩니다  

https://blog.kakaocdn.net/dn/bAmtqx/btq4f0vR6mp/lzThOlW2XQtMoHq5urHri1/img.jpg

Semantic segmentation이란 Object segmentation을 하되 같은 class인 object들은 같은 영역 혹은 색으로 분할해주고,  
반대로 Instance segmentation은 같은 class이여도 서로 다른 instance로 구분해줍니다  

따라서 object가 겹쳤을때 각각의 object를 구분해주지 못하면 Semantic segmentation  
구분할 수 있다면 Instance segmentation이라 할 수 있습니다

주어진 데이터를 확인해본 결과, mask value가 모두 동일해 semantic segmentation이 적절하다 판단했습니다


## 2. model

최대한 깊고 섬세한 모델이 필요하다 생각이 들어 resnet50을 이용한 deeplabv3를 사용했습니다
왜냐하면 파손 class 중 C는 마스크 영역이 굉장히 작았고, 사람이 봐도 패턴을 알 수 없을 정도로 규칙성이 없었습니다  
심지어 데이터가 그리 많지 않아 딥러닝으로 좋은 결과를 얻기 어려울거라 예상되어  
resnet과 atrous pooling으로 최대한 깊고 섬세한 feature를 뽑아내고자 했습니다

또 deeplabv3는 구현하기 쉽다는 장점 또한 있습니다

## 3. data preprocessing

모델이 한 번에 A, B, C를 예측할 수 있게 하기 위해 mask를 overlap했습니다  
이때 같은 이미지에서 mask를 추출한 A와 B는 mask를 이미지 연산으로 overlap할 수 있었지만,  
C같은 경우 전혀 다른 이미지에서 mask를 추출했기 때문에 이미지 연산으로 overlap할 수 없었습니다

때문에 A, B overlap mask를 학습시켜 모델이 A, B를 예측할 수 있게 만든 뒤,
C 이미지에서 A, B를 예측해 mask를 그리도록 했습니다

이렇게 A,B,C 마스크를 전부 overlap 했습니다


## 4. 결과

A, B overlap mask만 학습했을 때는 만족할 만한 결과가 나오지만,  
A, B, C overlap mask를 학습시키면 성능이 크게 하락합니다  
기존에 감지할 수 있던 class 마저도 감지를 못 하게 됩니다  

## 분석 및 2차 실험

분석 결과, 제가 만든 모델은 target mask 영역 안에서 예측해,  
마스크가 점점 작아지는 특징이 있었습니다  
저는 마스크가 작아지는게 C class mask를 소실시켜버리는 원인이라 생각해  
C class mask 외곽선을 중심으로 마스크의 크기를 늘렸습니다  
또 보다 섬세한 에측을 위해 Unet 구조를 모방해 decoder에 resnet50의 초반 activation layer를 연결해줬습니다

그 결과 만족할만한 결가가 나와 프로젝트를 끝내게 되었습니다
