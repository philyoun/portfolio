#19 Quasiquotation

나만의 요약: 
**quotation**이란? unevaluated expression을 캡쳐링하는 act
**unquotation**은? quoted evaluation이었을 부분을, 선택적으로 evaluate하는 기능

이걸 합쳐서 **quasiquotation**이라고 함. quasi라는 뜻은 준(準), 유사 이런 뜻임.

quasiquotation은 함수를 create하는 것을 쉽게 만들어준다.
어떤 함수? [function's author가 만든 코드]와 [function's user가 만든 코드]를 combine하는 함수