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


## Decoder

Decoder는 Encoder를 거쳐서 나온 Z를 decoding 하여 확률분포를 계산해 높은 확률로 나타나는 토큰들을 output 한다.

Decoder의 첫 부분에는 output embedding과 positional encoding을 거치고 이후에 decoder layer로 들어간다.

delcoder layter는
