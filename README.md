# NLP_assignment2

## Encoder
 
Encoder는 전체 소스 문장 X를 압축하지 않고 sequence Z로 만든다. 이때 Z 는 기존의 RNN 모델과 달리 현재 시점 t 이전의 토큰들의 정보를 가지는 것이 아닌 문장 X 전체의 정보를 담고 있다는 차이를 가진다.

소스 문장 X는 embedding layer를 거친 다음, recurrent하지 않은 각 토큰이 문장에서 어떤 순서를 가지는지 알려주기 위한 positional embedding layer로 들어간다.
(원래 논문에서는 fixed static embedding을 활용하지만 BERT와 같은 최신 모델에서는 positional embedding을 사용하므로 이 코드에서도 이 방식을 채택했다.)

위의 첫번째 layer를 거쳐서 얻은 토큰과 두번째 layer를 거쳐서 얻은 position 정보는 elementwise sum을 통해 하나의 combined 벡터로 만들어진다.

이후 이렇게 만들어진 combined 벡터는 본격적으로 encoder layer에 들어간다.

### Encoder Layer

encoder layer로 들어온 combined 벡터는 
1. Multi-head attention
2. dropout, residual connection, Layer Normalization layer
3. position-wise feedforward layer
4. dropout, residual connection, Layer Normalization layer
를 차례대로 지난다.

이때, multi-head attention 과정에서 self-attention을 적용한다. 이 과정은 이후 자세히 서술하겠다.

그리고 위와 같은 encoder layer를 N번 더 거쳐 최종 Z를 계산한다.

### Multi Head Attention Layer

- Attention
 1. 어텐션 함수는 Query에 대해 모든 Key 값의 유사도(attention score)를 구하고
 2. 유사도를 가중치로 (softmax) Key 와 맵핑된 각각의 Value에 반영한다.
 3. 이후 이 Value를 모두 가중합 해서 Attention Value를 반환한다.

- Self Attention
 위의 Attention과정을 자기 자신에게 수행한다. 즉, Q, K, V(셋다 동일)가 입력 문장의 모든 단어 토큰들을 의미한다.

- Transformer
 
 Transformer Encoder에 Self Attention을 적용하는 과정은 아래와 같다.
 1. Q, K, V에 가중치 W를 곱해 head와 같은 차원의 Q, K, V를 얻는다.
 2. 1에서 가중치를 곱해서 얻은 Q, K, V로 scaled dot product attention을 한다. scaled dot product attention이란 dot product한 값이 너무 커져 gradient가 작아지는 것을 막기 위해 사용하는 방법이다. 아래의 이미지는 scaled dot product attention을 수식으로 표현한 것이다.
![image](https://user-images.githubusercontent.com/48917098/201504506-9b876745-3319-4220-8a3d-d6d12d523d2d.png)
 3. 2의 과정은 head의 갯수만큼 진행되며 그 결과로 head의 수만큼의 attention matrix를 얻는다. 
 4. 모든 attention matrix를 concat한 뒤, 가중치 W를 곱해 최종 Multi head attention score를 얻는다.

### Position-wise Feedforward Layer
 - hid_dim 을 pf_dim 크기로 바꿨다가(fc_1) 다시 hid_dim 크기로 만드는(fc_2) 2개의 linear layer이다.
 - 이때, pf_dim은 일반적으로 hid_dim보다 크다. 논문에서는 pf_dim=2048, hid_dim=512로 설정했다.
 - fc_1을 지나며 activation layer(relu)와 dropout이 적용된다.


## Decoder

Decoder는 Encoder를 거쳐서 나온 Z를 decoding 하여 확률분포를 계산해 높은 확률로 나타나는 토큰들을 output 한다.

Decoder의 첫 부분에는 output embedding과 positional encoding을 거치고 이후에 decoder layer로 들어간다.
- output embedding은 token embedding으로 vocab에 있는 토큰으로 벡터를 만들어준다.
- position embedding은 최대 길이의 문장만큼 존재하며 위의 output embedding의 output과 element-wise sum을 거쳐 하나의 vector로 출력된다.

Decoder layer에는 2개의 Multi-head attention layer와 1개의 Linear layer가 존재한다.
- 첫번째 Multi-head attention layer는 input인 trg 시퀀스에 대해 masked self attention을 진행한다.
- 두번째 Multi-head attention layer는 첫번째 decoder의 output을 Q로 사용하고, encoder의 src_output을 K, V로 사용해 self-attention을 진행한다. 
- 마지막으로 Linear layer를 거쳐 최종 output의 크기를 hidden_dim에서 output_dim으로 맞춘다.

### Delcoder layter

위에서 언급한대로 2개의 Multi-head attention layer와 1개의 Linear layer가 존재한다. 각 layer를 편의상 아래와 같이 표시하겠다.
 1. Self-attention layer
 2. Encoder-attention layer
 3. Position-wise feedforward layer

- Self-attention layer
  Decoder representation 사이의 self-attention을 의미한다. 이 과정은 Encoder의 self-atteintion과 비슷하나 target sequence masking이 사용된다는 점에서 차이가 있다. 이는 Decoder가 다음 단어를 예측하는 시점에 미래의 단어(정답)를 보지 못하도록 만들기 위해서 masking을 사용한다. masking을 해야 하는 부분에 음의 무한대에 가까운 음수를 더해 softmax과정을 거치면 0이 되게 만들어 attention 연산이 일어나지 않도록 만든다.
  
- Encoder-attention layer
  이 layer에서는 이전 decoder의 output을 Q로 사용하고, encoder의 src_output을 K, V로 사용해 self-attention을 진행한다. 여기서도 이전 layer와 마찬가지로 src_mask가 사용된다.
- Position-wise feedforward layer
  Encoder의 fc_layer와 동일하게 진행된다.
  
  
## Seq2Seq
Encoder와 Decoder가 함께 구현(연결)되어 있고, mask를 생성하는 부분이 포함되어 있는 형태이다.

source mask란 <pad> 토큰이 아닐 때는 1이고, <pad> 토큰일 때는 0이다. unsqueeze가 되는 이유는 energy와 계산 되는 과정에서 올바르게 broadcast 되게 하기 위해서 이다.

target mask란 우선 source mask처럼 <pad>토큰을 생성한다. 그리고 torch.tril을 이용해 "subsequent"마스크를 생성하는데 이는 아래 이미지처럼 대각선의 우측 상단이 모두 0인 행렬이 만들어진다. 이런 형태의 행렬은 input과 관계없이 우측 상단이 모두 0인 행렬을 만들어 마스킹을 할 수 있다.

![image](https://user-images.githubusercontent.com/48917098/201505777-98f28aac-1fd0-4e79-85c7-1c7c6f17b808.png)

이후에 패딩 마스크가 함께 결합되면 최종적으로 아래와 같은 형태의 마스킹 행렬이 만들어진다.

![image](https://user-images.githubusercontent.com/48917098/201505791-47e225f0-95ab-4508-aa4e-62295d4e8811.png)
 
이렇게 생성된 mask는 Encoder Decoder에세 예측 대상 문장의 output을 얻기 위해 src sentence와 trg sentence에 적용되어 사용된다.
