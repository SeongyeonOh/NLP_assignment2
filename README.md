# NLP_assignment2

<Encoder>
 
Encoder는 전체 소스 문장 X를 압축하지 않고 sequence Z로 만든다. 이때 Z 는 기존의 RNN 모델과 달리 현재 시점 t 이전의 토큰들의 정보를 가지는 것이 아닌 문장 X 전체의 정보를 담고 있다는 차이를 가진다.

소스 문장 X는 embedding layer를 거친 다음, recurrent하지 않은 각 토큰이 문장에서 어떤 순서를 가지는지 알려주기 위한 positional embedding layer로 들어간다.
(원래 논문에서는 fixed static embedding을 활용하지만 BERT와 같은 최신 모델에서는 positional embedding을 사용하므로 이 코드에서도 이 방식을 채택했다.)

위의 첫번째 layer를 거쳐서 얻은 토큰과 두번째 layer를 거쳐서 얻은 position 정보는 elementwise sum을 통해 하나의 combined 벡터로 만들어진다.

이후 이렇게 만들어진 combined 벡터는 본격적으로 encoder layer에 들어간다.

encoder layer로 들어온 combined 벡터는 
1. Multi-head attention
2. dropout, residual connection, Layer Normalization layer
3. position-wise feedforward layer
4. dropout, residual connection, Layer Normalization layer
를 차례대로 지난다.

이때, multi-head attention 과정에서 self-attention을 적용한다. 이 과정은 이후 자세히 서술하겠다.

그리고 위와 같은 encoder layer를 N번 더 거쳐 최종 Z를 계산한다.

<Decoder>

Decoder는 Encoder를 거쳐서 나온 
