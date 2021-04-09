---
title: KALDI, ZEROTH 개발환경 삽질기
tags: Kaldi Zeroth TTS
---

## KALDI, ZEORTH 개발환경 설정

하기 사이트 참고하여 진행하였습니다.

1. https://cording-cossk3.tistory.com/6
2. https://bourbonkk.tistory.com/28
3. http://nblog.syszone.co.kr/archives/9788
4. https://groups.google.com/g/zeroth-help

생각보다 KALDI프로젝트의 INSTALL파일의 설명을 따라가다보면 쉘 스크립트가 다음 단계를 어떻게 해야하는지
가이드를 잘 뱉어 줍니다.

막힌 부분들에서는 상기 사이트들에서 힌트를 얻으면서 진행했습니다.

작업하는 PC의 스펙이 낮아서 설정을 찾느라 고생을 좀 했습니다.

작업 PC 하드웨어 환경  
CPU : 16 core  
RAM : 32 GB  
GPU : 2 GB  

GPU는 여러개인 편이 좋습니다. 그이유는 아래 나옵니다.

### 삽질 1. GPU wait 옵션 주기

- train.py 실행단계에서 gpu에 다른 점유 프로세스가 있을 시, 학습 중 에러를 뱉는 이슈. GPU가 1개
일시 뱉을 확률이 매우매우매우 높은것으로 보인다.
- 제로스 구글 그룹 에러문의에서 같은 증상에 대한
  [답변](https://groups.google.com/g/zeroth-help/c/8Vp22CYxHmk)을 참고했다.


- [run_tdnn_1n.sh : L 222](https://github.com/goodatlas/zeroth/blob/master/s5/local/chain/run_tdnn_1n.sh#L222)
  : --use-gpu 옵션을 true -> wait으로 변경한다.

~~~python
steps/nnet3/chain/train.py --stage $train_stage \
  --cmd "$decode_cmd" \

...

  --use-gpu true // -> wait으로 변경
~~~
