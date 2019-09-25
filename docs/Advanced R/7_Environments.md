7.4 Special Environments
------------------------

대부분의 env들은 너가 만드는게 아니라 R이 만든다. <br /> 이 섹션에서는 package env부터 시작해서 중요한 env들에 대해 배운다. <br /> 그리고 함수function가 만들어질 때 함수에 bound되는 function env에 대해 배우고, <br /> 함수가 호출될 때마다 생겼다가 없어지는, ephemeral execution env에 대해 배운다. <br /> 1. package env, 2. function env, 3. execution env

마지막으로, package와 function env가 namespaces를 지원support하기 위해 어떻게 interact하는지, <br />     namespaces는 user가 어떤 다른 패키지들을 load했던 간에 패키지가 항상 같은 방식으로 행동하게끔 한다.
